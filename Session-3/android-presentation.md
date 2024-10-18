// APP FLOW


manifest verify you have added network permission

```kotlin
<uses-permission android:name="android.permission.INTERNET"/>
```

Copy the API KEY obtained from newsapi.org to shared/com.trenser.newsapp/data/remote/ApiService
```kotlin
private val API_KEY = "e7e844fe75c94063bc9a71df9280621f"
```

appmodule

```kotlin
val appModule = module {
    // Provide ViewModel
    viewModel {
        ContentViewModel()
    }
    viewModel {
        SplashViewModel()
    }
    viewModel {
        ArticleListViewModel(get())
    }
    viewModel {
        BookmarkListViewModel(get())
    }

    // Define ViewModel with parameter injection
    viewModel { (article: Article) ->
        ArticleDetailViewModel(
            article = article,
            addToBookmarksUsecase = get(),
            deleteFromBookmarksUsecase = get(),
            isBookmarkedUsecase = get()
        )
    }

}
```

1.Create Application class

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApplication)
            modules(listOf(
                sharedModule,
                platformSpecificSharedModule,
                appModule
            ))
        }
    }
}
```

1.Create Main Activity

MainActivity
```kotlin
//[package_name]\MainActivity.kt
package com.trenser.newsapp.android

import android.app.Application
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.trenser.newsapp.android.di.appModule
import com.trenser.newsapp.platformSpecificSharedModule
import com.trenser.newsapp.sharedModule
import org.koin.android.ext.koin.androidContext
import org.koin.core.context.startKoin

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApplication)
            modules(listOf(
                sharedModule,
                platformSpecificSharedModule,
                appModule
            ))
        }
    }
}

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApplicationTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    ContentView()
                }
            }
        }
    }
}

@Composable
fun GreetingView(text: String) {
    Text(text = text)
}

@Preview
@Composable
fun DefaultPreview() {
    MyApplicationTheme {
        GreetingView("Hello, Android!")
    }
}

```

Create ContentView
```kotlin
package com.trenser.newsapp.android

import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import com.trenser.newsapp.android.home.ButtonViewModel
import com.trenser.newsapp.android.home.HomeView
import com.trenser.newsapp.android.home.articles.ArticleListView
import com.trenser.newsapp.android.splash.SplashScreen
import org.koin.androidx.compose.koinViewModel

@Composable
fun ContentView( viewModel: ContentViewModel = koinViewModel()){
    val state by viewModel.state.collectAsState()

    Box(modifier = Modifier.fillMaxSize()){
        when(state.startType){
            StartType.SPLASH ->{
                SplashScreen {
                    viewModel.doAction(ContentViewModel.Action.setStartType(StartType.HOME))
                }
            }
            StartType.GUIDANCE ->{
                Text(text = "Guidance")
            }
            StartType.HOME->{
                HomeView()
            }
        }
    }
}
```

Content ViewModel
```kotlin
package com.trenser.newsapp.android

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch

enum class StartType(){
    SPLASH,
    GUIDANCE,
    HOME
}

class ContentViewModel(): ViewModel(){

    data class State(
        var startType: StartType = StartType.SPLASH
    )

    sealed class Action {
        data class setStartType(val startType: StartType) : Action()
    }

    fun doAction(action: Action){
        when(action){
            is Action.setStartType -> {
                setStartType(action.startType)
            }
        }
    }

    private val _uiStates = MutableStateFlow(State())
    val state: StateFlow<State> = _uiStates.asStateFlow()

    private fun setStartType(type: StartType){
        viewModelScope.launch {
            _uiStates.update { it.copy(startType = type) }
        }
    }
}
```

Create SplashViewModel

```kotlin
package com.trenser.newsapp.android.splash

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch

class SplashViewModel(): ViewModel(){

    data class State(
        var isDataLoadingCompleted: Boolean = false,
    )

    sealed class Action {
        object LoadData : Action()
    }

    private val _uiStates = MutableStateFlow(State())
    val state: StateFlow<State> = _uiStates.asStateFlow()

    fun doAction(action: Action){
        when(action){
            is Action.LoadData -> {
                loadData()
            }
        }
    }

    private fun loadData(){
        viewModelScope.launch {
            delay(2000L)
            _uiStates.update { it.copy(isDataLoadingCompleted = true) }
        }
    }
}
```

SplashScreen

```kotlin
package com.trenser.newsapp.android.splash

import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.trenser.newsapp.android.R
import org.koin.androidx.compose.koinViewModel

@Composable
fun SplashScreen(
    viewModel: SplashViewModel = koinViewModel(),
    showNextScreen: () -> Unit
) {
    val state by viewModel.state.collectAsState()

    LaunchedEffect(Unit) {
        viewModel.doAction(SplashViewModel.Action.LoadData)
    }

    if (state.isDataLoadingCompleted) {
        showNextScreen()
    } else {
        SplashPreview()
    }
}

@Preview
@Composable
fun SplashPreview(){
    Column(
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = Modifier.fillMaxSize()
    ) {
        Image(
            painter = painterResource(id = R.drawable.splash),
            contentDescription = null, Modifier.size(240.dp)
        )
        Text(text = "News API",
            style = TextStyle(color = Color.White,
                fontSize = 24.sp,
                fontFamily = FontFamily.Serif),
            modifier = Modifier.padding(20.dp))
    }
}
```

Create HomeView

```kotlin
package com.trenser.newsapp.android.home

import com.trenser.newsapp.domain.entity.Article
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.filled.Home
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.NavigationBar
import androidx.compose.material3.NavigationBarItem
import androidx.compose.material3.NavigationBarItemColors
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import com.trenser.newsapp.android.home.articles.ArticleListView
import com.trenser.newsapp.android.home.bookmarks.BookmarkListView
import com.trenser.newsapp.android.home.details.ArticleDetailViewModel
import com.trenser.newsapp.android.home.details.ArticleDetailsView
import kotlinx.serialization.encodeToString
import kotlinx.serialization.json.Json
import org.koin.androidx.compose.koinViewModel
import org.koin.core.parameter.parametersOf

@Composable
fun HomeView() {
    val navController = rememberNavController()
    NavHost(navController = navController, startDestination = "home"){
        composable("home"){
            HomeTabView {article->
                val jsonString = Json.encodeToString(article)
                navController.currentBackStackEntry?.savedStateHandle?.set("article", jsonString)
                navController.navigate("details")
            }

        }
        composable("details"){
            navController.previousBackStackEntry?.savedStateHandle?.get<String>("article")?.let {
                val article = Json.decodeFromString<Article>(it)
                val viewModel: ArticleDetailViewModel = koinViewModel { parametersOf(article) }
                ArticleDetailsView(viewModel = viewModel){
                    navController.popBackStack()
                }
            }
        }
    }
}

@Composable
fun HomeTabView(showArticle: (article:Article)->Unit){
    val bottomNavController = rememberNavController()
    val tabs = listOf("News", "Bookmarks")
    var selectedTabIndex by rememberSaveable { mutableStateOf(0) }

    Scaffold(modifier = Modifier.fillMaxSize(), bottomBar = {
        BottomTabsComponent(tabs = tabs, selectedTabIndex = selectedTabIndex) {index->
            selectedTabIndex = index
            bottomNavController.navigate(tabs[index])
        }
    }){
        val bottomPadding = it.calculateBottomPadding()
        NavHost(
            navController = bottomNavController,
            startDestination = tabs.first(),
            modifier = Modifier.padding(bottom = bottomPadding)
        ){
            composable(route = tabs[0]){
                ArticleListView {
                    showArticle(it)
                }
            }
            composable(route = tabs[1]){
                BookmarkListView {
                    showArticle(it)
                }
            }
        }
    }

}

@Composable
fun BottomTabsComponent(tabs:List<String>,
                        selectedTabIndex: Int,
                        onTabSelected: (Int) -> Unit){
    NavigationBar(
        modifier = Modifier.fillMaxWidth(),
        containerColor = MaterialTheme.colorScheme.background,
        tonalElevation = 10.dp
    ){
        tabs.forEachIndexed { index, title ->
            NavigationBarItem(
                icon = {
                    val icon = if (index==0){
                        Icons.Filled.Home
                    }else{
                        Icons.Filled.Favorite
                    }
                    Icon(icon, contentDescription = title) },
                label = { Text(title) },
                selected = index == selectedTabIndex,
                onClick = {
                    onTabSelected(index)
                },
                colors = NavigationBarItemColors(
                    selectedIndicatorColor = Color(0xFF6200EE),
                    selectedIconColor = Color.White,
                    unselectedIconColor = Color.Black,
                    selectedTextColor = Color(0xFF6200EE),
                    unselectedTextColor = Color.Black,
                    disabledIconColor = Color.White,
                    disabledTextColor = Color.White
                )

            )
        }
    }
}
```

Create a MockData class in shared module package/data folder

```kotlin
package com.trenser.newsapp.data

import com.trenser.newsapp.domain.entity.Article
import com.trenser.newsapp.domain.entity.NewsPage
import com.trenser.newsapp.domain.entity.Source

object MockData {

    val article = Article(
        Source("bv", "ABC News"),
        "",
        "Asian stocks down after Big Tech pulls S&P 500 and Nasdaq lower",
        "",
        "",
        "https://i.abcnewsfe.com/a/006ea474-07ff-459d-8572-ce779550d02d/wirestory_8bf31188126a0b5a963174dd1eee0465_16x9.jpg?w=100",
        "",
        "Asian stocks down after Big Tech pulls S&P 500 and Nasdaq lower"
    )
    val newsPage = NewsPage(1,3,List(3) { article })

}
```


Create ArticleListViewModel

```kotlin
package com.trenser.newsapp.android.home.articles

import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.trenser.newsapp.domain.entity.Article
import com.trenser.newsapp.domain.usecase.GetNews
import kotlinx.coroutines.launch

class ArticleListViewModel(private val getNewsUsecase: GetNews): ViewModel(){

    init {
        doAction(Action.FetchNews)
    }

    data class State(
        val newsList: List<Article> = emptyList()
    )

    sealed class Action {
        object FetchNews: Action()
    }

    fun doAction(action: Action){
        when(action){
            is Action.FetchNews -> {
                fetchNews()
            }
        }
    }

    // Mutable state for the current state
    private var _state by mutableStateOf(State())
    val state: State
        get() = _state

    // Function to fetch news and update the LiveData
    fun fetchNews() {
        viewModelScope.launch {
            try {
                val newsResponse = getNewsUsecase("bbc-news",1) // Fetching news from "bbc-news" source
                val articles = newsResponse.articles.distinctBy { it.title } // Removing duplicates
                // Update the state directly
                _state = _state.copy(newsList = articles)
            } catch (e: Exception) {
                e.printStackTrace()
                // Update the state directly
                _state = _state.copy(newsList = emptyList())
            }
        }
    }
}
```
Create ArticleListView

```kotlin
package com.trenser.newsapp.android.home.articles

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.PaddingValues
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.KeyboardArrowRight
import androidx.compose.material3.HorizontalDivider
import androidx.compose.material3.Icon
import androidx.compose.material3.ListItem
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import coil.compose.AsyncImage
import coil.request.ImageRequest
import com.trenser.newsapp.data.MockData
import com.trenser.newsapp.domain.entity.Article
import org.koin.androidx.compose.koinViewModel

@Composable
fun ArticleListView(viewModel: ArticleListViewModel = koinViewModel(),
                    showArticle: (article: Article)->Unit
) {
    val articles = viewModel.state.newsList
    ArticleListViewContent(articles = articles, showArticle =showArticle)
}

@Composable
fun ArticleListViewContent(
    articles: List<Article>?,
    showArticle: (article:Article)->Unit) {
    articles?.let {
        if (articles.size>0){
            LazyColumn(
                modifier = Modifier.fillMaxSize(),
                verticalArrangement = Arrangement.spacedBy(10.dp),
                contentPadding = PaddingValues(all = 10.dp)
            ) {
                items(
                    count = articles.size,
                ) {
                    articles[it]?.let { article ->
                        ArticleListItem(article = article,showArticle)
                    }
                }
            }
        }else{
            Text(text = "ArticleListView")
        }
    }
}

@Composable
fun ArticleListItem(article: Article,showArticle: (article:Article)->Unit){
    Column{
        ListItem(
            headlineContent = { Text(article.title) },
            supportingContent = { Text(article.source.name) },
            trailingContent = { Icon(Icons.AutoMirrored.Filled.KeyboardArrowRight,"") },
            leadingContent = {
                AsyncImage(
                    modifier = Modifier
                        .size(100.dp)
                        .clip(CircleShape),
                    model = ImageRequest.Builder(LocalContext.current).data(article.urlToImage).build(),
                    contentDescription = null,
                    contentScale = ContentScale.Crop
                )
            },
            modifier = Modifier.clickable {
                showArticle(article)
            }
        )
        HorizontalDivider()
    }
}

@Preview
@Composable
fun ArticleListItemPreview() {
    ArticleListItem(MockData.article){}
}
```

Create ArticleDetailsViewModel
```kotlin
package com.trenser.newsapp.android.home.details

import android.content.Context
import android.content.Intent
import android.net.Uri
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.trenser.newsapp.domain.entity.Article
import com.trenser.newsapp.domain.usecase.AddToBookmarks
import com.trenser.newsapp.domain.usecase.DeleteFromBookmarks
import com.trenser.newsapp.domain.usecase.IsBookmarked
import kotlinx.coroutines.launch

class ArticleDetailViewModel(val article: Article,
                             private val addToBookmarksUsecase: AddToBookmarks,
                             private val deleteFromBookmarksUsecase: DeleteFromBookmarks,
                             private val isBookmarkedUsecase: IsBookmarked
): ViewModel(){

    data class State(
        val isBookmarked :Boolean = false
    )

    // Mutable state for the current state
    private var _state by mutableStateOf(State())
    val state: State
        get() = _state

    fun doAction(action: ViewAction){
        when(action){
            is ViewAction.LoadData -> loadData()
            is ViewAction.ToggleBookmark -> toggleBookmark()
            is ViewAction.ShareArticle -> shareArticle(action.context)
            is ViewAction.ReadFullArticle -> openInBrowser(action.context)
        }
    }

    private fun loadData(){
        viewModelScope.launch {
            _state = _state.copy(isBookmarkedUsecase(article = article))
        }
    }

    private fun toggleBookmark(){
        _state =_state.copy(!state.isBookmarked)
        viewModelScope.launch {
            if (_state.isBookmarked){
                addToBookmarksUsecase(article = article)
            }else{
                deleteFromBookmarksUsecase(article = article)
            }
        }
    }

    private fun shareArticle(context: Context){
        Intent(Intent.ACTION_SEND).also {
            it.putExtra(Intent.EXTRA_TEXT, article.url)
            it.type = "text/plain"
            if (it.resolveActivity(context.packageManager) != null) {
                context.startActivity(it)
            }
        }
    }

    private fun openInBrowser(context: Context){
        Intent(Intent.ACTION_VIEW).also {
            it.data = Uri.parse(article.url)
            if (it.resolveActivity(context.packageManager) != null) {
                context.startActivity(it)
            }
        }
    }
}

sealed class ViewAction{
    object LoadData : ViewAction()
    object ToggleBookmark : ViewAction()
    data class ShareArticle(val context: Context) : ViewAction()
    data class ReadFullArticle(val context: Context) : ViewAction()
}
```
ArticleDetailsView
```kotlin
package com.trenser.newsapp.android.home.details

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.safeContentPadding
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material.icons.filled.Email
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.filled.FavoriteBorder
import androidx.compose.material.icons.filled.Share
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import coil.compose.AsyncImage
import coil.request.ImageRequest
import com.trenser.newsapp.domain.entity.Article

@Composable
fun ArticleDetailsView(
    viewModel: ArticleDetailViewModel,
    backClick: () -> Unit
){
    // Trigger LoadData action when the Composable is first launched
    LaunchedEffect(Unit) {
        viewModel.doAction(ViewAction.LoadData)
    }
    ArticleDetailsViewContent(viewModel.article,viewModel.state.isBookmarked,{
        viewModel.doAction(it)
    },{
        backClick.invoke()
    })
}

@Composable
fun ArticleDetailsViewContent(article: Article,
                              isBookmarked: Boolean,
                              doAction: (ViewAction)->Unit,
                              backClick: ()->Unit) {

    val context = LocalContext.current

    Column {
        ArticleDetailsTopBar(
            title = "Details",
            isBookmarked = isBookmarked,
            onBrowsingClick = {
                doAction(ViewAction.ReadFullArticle(context))
            },
            onShareClick = {
                doAction(ViewAction.ShareArticle(context))
            },
            onBookMarkClick = {
                doAction(ViewAction.ToggleBookmark)
            },
            onBackClick = {
                backClick.invoke()
            })
        AsyncImage(
            modifier = Modifier
                .height(200.dp)
                .clip(MaterialTheme.shapes.medium),
            model = ImageRequest.Builder(LocalContext.current).data(article.urlToImage).build(),
            contentDescription = null,
            contentScale = ContentScale.Crop
        )

        Spacer(modifier = Modifier.height(10.dp))

        Column(modifier = Modifier.safeContentPadding()) {
            Text(
                text = article.title,
                style = MaterialTheme.typography.displaySmall
            )
            Spacer(modifier = Modifier.height(10.dp))
            Text(
                text = article.content!!,
                style = MaterialTheme.typography.bodyMedium
            )
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ArticleDetailsTopBar(
    title:String,
    isBookmarked: Boolean,
    onBrowsingClick: () -> Unit,
    onShareClick: () -> Unit,
    onBookMarkClick: () -> Unit,
    onBackClick: () -> Unit,
) {
    TopAppBar(
        modifier = Modifier.fillMaxWidth(),
        title = { Text(text = title) },
        navigationIcon = {
            IconButton(onClick = onBackClick) {
                Icon(imageVector = Icons.AutoMirrored.Filled.ArrowBack, contentDescription = null)
            }
        },
        actions = {
            IconButton(onClick = onBookMarkClick) {
                if (isBookmarked) {
                    Icon(
                        imageVector = Icons.Default.Favorite,
                        contentDescription = null
                    )

                } else {
                    Icon(
                        imageVector = Icons.Default.FavoriteBorder,
                        contentDescription = null
                    )
                }
            }
            IconButton(onClick = onShareClick) {
                Icon(
                    imageVector = Icons.Default.Share,
                    contentDescription = null
                )
            }
            IconButton(onClick = onBrowsingClick) {
                Icon(
                    imageVector = Icons.Default.Email,
                    contentDescription = null
                )
            }
        },
    )
}
```

Create BookmarkListViewModel

```kotlin
package com.trenser.newsapp.android.home.bookmarks

import androidx.compose.runtime.mutableStateOf
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.trenser.newsapp.domain.entity.Article
import com.trenser.newsapp.domain.usecase.GetBookmarks
import kotlinx.coroutines.launch

class BookmarkListViewModel(private val getBookmarksUsecase: GetBookmarks) : ViewModel() {

    data class State(
        val bookmarkList: List<Article> = emptyList()
    )

    sealed class Action {
        object GetBookMarks : Action()
    }

    fun doAction(action: Action) {
        when (action) {
            is Action.GetBookMarks -> {
                getBookMarks()
            }
        }
    }

    private val _bookmarks = mutableStateOf<List<Article>>(emptyList())
    val state: State
        get() = State(bookmarkList = _bookmarks.value)

    private fun getBookMarks() {
        viewModelScope.launch {
            getBookmarksUsecase().collect { value ->
                _bookmarks.value = value
            }
        }
    }

}
```

Create BookmarkListView
```kotlin
package com.trenser.newsapp.android.home.bookmarks

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.PaddingValues
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.trenser.newsapp.android.MyApplicationTheme
import com.trenser.newsapp.android.home.articles.ArticleListItem
import com.trenser.newsapp.data.MockData
import com.trenser.newsapp.domain.entity.Article
import org.koin.androidx.compose.koinViewModel


@Composable
fun BookmarkListView(viewModel: BookmarkListViewModel = koinViewModel(),
                     showArticle: (article: Article)->Unit) {

    val bookmarks = viewModel.state.bookmarkList
    viewModel.doAction(BookmarkListViewModel.Action.GetBookMarks)
    BookmarkListViewContent(bookmarks,showArticle)
}

@Composable
fun BookmarkListViewContent(bookmarks: List<Article>,
                            showArticle: (article: Article)->Unit) {

    if (bookmarks.isNotEmpty()){
        LazyColumn(
            modifier = Modifier.fillMaxSize(),
            verticalArrangement = Arrangement.spacedBy(10.dp),
            contentPadding = PaddingValues(all = 10.dp)
        ) {
            items(
                count = bookmarks.size,
            ) {
                ArticleListItem(article = bookmarks[it],showArticle)
            }
        }
    }else{
        Text(text = "BookmarkListView")
    }
}

@Preview
@Composable
fun BookmarkListViewPreview() {
    MyApplicationTheme {
        Surface(modifier = Modifier.fillMaxSize(), color = MaterialTheme.colorScheme.background){
             BookmarkListViewContent(List<Article>(3) { MockData.article }){}
        }
    }
}
```
