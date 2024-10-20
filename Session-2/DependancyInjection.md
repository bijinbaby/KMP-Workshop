
## Dependency Injection using Koin

### Step 1 : Define Koin modules
#### Step 1.1 : Define Koin for shared-module 
path : [project root]/shared/src/commonMain/kotlin/com/trenser/newsapp/Platform.kt

- Add the below code to the file at above path
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
path : [project root]/shared/src/commonMain/kotlin/com/trenser/newsapp/Platform.kt

- Add the below code to the file at above path
```kotlin
expect val platformSpecificSharedModule: Module
```

#### Step 1.3 : Override the plaform-specific-module variable inside Android module
path : [project root]/shared/src/androidMain/kotlin/com/trenser/newsapp/Platform.android.kt

- Add the below code to the file at above path
```kotlin
actual val platformSpecificSharedModule = module {
    single<BookmarkRepository> { BookmarkRepositoryImpl(
        databaseDriverFactory = AndroidDatabaseDriverFactory(androidContext())
    ) }
}
```

#### Step 1.4 : Override the plaform-specific-module variable inside iOS module
path : [project root]/shared/src/iosMain/kotlin/com/trenser/newsapp/Platform.ios.kt
```kotlin
actual val platformSpecificSharedModule = module {
    single<BookmarkRepository> { BookmarkRepositoryImpl(
        databaseDriverFactory = IOSDatabaseDriverFactory()
    ) }
}
```


### Step 2 : Start Koin DI
#### Step 2.1 : Create a class inside iosMain to help start Koin for iOS

##### Folder structure generation
- Right click 'shared/src/iosMain/kotlin/com/trenser/newsapp/data' folder and open in terminal
- Execute the below code to create a new file named 'KoinHelper.kt'
  
For Mac machines :
```kotlin
touch "KoinHelper.kt"
```
For Windows Machines :
```kotlin
ni "KoinHelper.kt"
```

##### Paste the below code to the created file
path : [project root]/shared/src/iosMain/kotlin/com/trenser/newsapp/data/KoinHelper.kt
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
- Why do we need this class?
    - Koin is a Kotlin library so we cannot use it directly inside iOS application
    - We will call KoinHelper.initKoin() inside the entry point of the swiftUI app

#### Step 2.2 : Create a class inside iosMain to use Koin inside iOS app
##### Folder structure generation
- Right click '[project root]/shared/src/iosMain/kotlin/com/trenser/newsapp/data' folder and open in terminal (skip this if you are already at this path)
- Execute the below code to create a new file named 'ViewModelHelper.kt'

For Mac machines :
```kotlin
touch "ViewModelHelper.kt"
```
For Windows machines :
```kotlin
ni "ViewModelHelper.kt"
```

path : [project root]/shared/src/iosMain/kotlin/com/trenser/newsapp/data/ViewModelHelper.kt
- We will write a class and some variables for injecting dependencies into the iOS app inside the created file
```kotlin
sealed class ViewModelHelper: KoinComponent {

    data object ArticleListViewModelHelper: ViemodelHelper() {
        val getNewsUsecase: GetNews by inject<GetNews>()
    }

    data object BookmarkListViewModelHelper : ViewModelHelper() {
        val getBookmarksUsecase: GetBookmarks by inject<GetBookmarks>()
    }

    data object ArticleDetailsViewModelHelper : ViewModelHelper() {
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
- Right click '[project root]/androidApp/src/main/java/com/trenser/newsapp' folder and open in terminal
- Execute the below code inside terminal to create a new file named 'ViewModels.kt'
```kotlin
touch "ViewModels.kt"
```

##### Paste the below code to the created file
path : [project root]/androidApp/src/main/java/com/trenser/newsapp/ViewModels.kt

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

#### Step 3.2 : Define Koin for Android-module
##### Folder structure generation
- Create a new file named 'AppModule.kt' inside the same folder used in the previous step(step 3.1.1)
- Execute the below code inside terminal to create the file

For Mac machines :
```kotlin
touch "AppModule.kt"
```
For Windows machines :
```kotlin
ni "AppModule.kt"
```

##### Paste the below code to the created file
path : [project root]/androidApp/src/main/java/com/trenser/newsapp/AppModule.kt
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
- We do this to inject ViewModels to the Android application
- Since Koin is a Kotlin library, we will use a Koin module to inject View models directly to Android app

#### Step 3.3 : Create MyApplication class and Start DI for Android(File will already be present). 
path : [project root]/androidApp/src/main/java/com/trenser/newsapp/android/MainActivity.kt
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
