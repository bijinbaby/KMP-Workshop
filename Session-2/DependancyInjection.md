
## Dependency Injection using Koin

### Step 1 : Define Koin modules
#### Step 1.1 : Define Koin for shared-module 
path : shared/src/commonMain/kotlin/com/trenser/newsapp/Platform.kt

- Add the below code to the path
```kotlin
val sharedModule = module {
    // Provide Repository
    single<NewsRepository> { NewsRepositoryImpl() }

    // Provide UseCase
    factory { GetNews(get()) }
    factory { GetBookmarks(get()) }
    factory { AddToBookmarks(get()) }
    factory { DeleteFromBookmarks(get()) }
    factory { IsBookmarked(get()) }
}
```

#### Step 1.2 : Define Koin module variable for platform-specific-modules
path : shared/src/commonMain/kotlin/com/trenser/newsapp/Platform.kt

- Add the below code to the path
```kotlin
expect val platformSpecificSharedModule: Module
```

#### Step 1.3 : Override the plaform-specific-module variable inside Android module
path : shared/src/androidMain/kotlin/com/trenser/newsapp/Platform.android.kt

- Add the below code to the path
```kotlin
actual val platformSpecificSharedModule = module {
    single<BookmarkRepository> { BookmarkRepositoryImpl(
        databaseDriverFactory = AndroidDatabaseDriverFactory(androidContext())
    ) }
}
```

#### Step 1.4 : Override the plaform-specific-module variable inside iOS module. path :
shared/src/iosMain/kotlin/com/trenser/newsapp/Platform.ios.kt
```kotlin
actual val platformSpecificSharedModule = module {
    single<BookmarkRepository> { BookmarkRepositoryImpl(
        databaseDriverFactory = IOSDatabaseDriverFactory()
    ) }
}
```


### Step 2 : Start Koin DI
#### Step 2.1 : Write a class inside iosMain to help start Koin for iOS inside path :
shared/src/iosMain/kotlin/com/trenser/newsapp/data/local/db/di/KoinHelper.kt

Why do we need this class?
- Koin is a Kotlin library so we cannot use it directly inside iOS application
- We will call KoinHelper.initKoin() inside the entry point of the swiftUI app

```kotlin
class KoinHelper : KoinComponent {
    fun initKoin() {
        startKoin {
            modules(
                listOf(
                    sharedModule,
                    platformSpecificSharedModule
                )
            )
        }
    }
}
```

#### Step 2.2 : Write a class inside iosMain to use Koin inside iOS app inside path :
 shared/src/iosMain/kotlin/com/trenser/newsapp/data/local/db/di/ViemodelHelper.kt
- We will write a class and some variables for injecting dependencies into iOS app
```kotlin
sealed class ViemodelHelper:KoinComponent{

    data object ArticleListViewModelHelper: ViemodelHelper(){
        val getNewsUsecase: GetNews by inject<GetNews>()
    }

    data object BookmarkListViewModelHelper : ViemodelHelper() {
        val getBookmarksUsecase: GetBookmarks by inject<GetBookmarks>()
    }

    data object ArticleDetailsViewModelHelper : ViemodelHelper() {
        val addToBookmarksUsecase: AddToBookmarks by inject<AddToBookmarks>()
        val deleteFromBookmarksUsecase: DeleteFromBookmarks by inject<DeleteFromBookmarks>()
        val isBookmarkedUsecase: IsBookmarked by inject<IsBookmarked>()
    }
}
```

# Session 3

## Dependency Injection - Continuation
### Step 3 : Use Koin DI in android project

#### Step 3.1 : Create viemodels for Android with dependencies
##### Step 3.1.1 : Create a class to hold all the viewModels for Android

##### Folder structure generation
- Open the terminal inside android studio
- Drag 'androidApp/src/main/java/com/trenser/newsapp' folder to the terminal
```kotlin
touch "ViewModels.kt"
```


```kotlin
class ArticleListViewModel(private val getNewsUsecase: GetNews): ViewModel(){}

class BookmarkListViewModel(private val getBookmarksUsecase: GetBookmarks): ViewModel(){}

class ArticleDetailViewModel(val article: Article,
                             private val addToBookmarksUsecase: AddToBookmarks,
                             private val deleteFromBookmarksUsecase: DeleteFromBookmarks,
                             private val isBookmarkedUsecase: IsBookmarked): ViewModel(){}

class SplashViewModel(): ViewModel(){}

class ContentViewModel(): ViewModel(){}
```

#### Step 3.2 : Define Koin for Android-module in AppModule file inside path :
##### Folder structure generation
- Open the terminal inside android studio
- Drag 'androidApp/src/main/java/com/trenser/newsapp' folder to the terminal
```kotlin
touch "AppModule.kt"
```
####Paste the below code to the created file
path : androidApp/src/main/java/com/trenser/newsapp/AppModule.kt

- We do this to inject ViewModels to the Android application
- Since Koin is a Kotlin library, we will use a Koin module to inject View models directly to Android app
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

#### Step 3.3 : Create MyApplication class and Start DI for Android(Path will already be present). 
path : androidApp/src/main/java/com/trenser/newsapp/android/MainActivity.kt
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
