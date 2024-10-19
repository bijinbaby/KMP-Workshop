# Session 2 - Shared Module (Domain and data layer)

## Folder structure generation
#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp
#### For Mac Machine
```kotlin
mkdir "domain" "domain/entity" "domain/repository" "domain/usecase" "data" "data/remote" "data/local" "data/repository" "data/mapper" "data/remote/dto" "data/local/db"

touch "domain/entity/NewsPage.kt" "domain/entity/Article.kt" "domain/entity/Source.kt" "domain/repository/NewsRepository.kt" "domain/repository/BookmarkRepository.kt" "domain/usecase/GetNews.kt" "domain/usecase/GetBookmarks.kt" "domain/usecase/AddToBookmarks.kt" "domain/usecase/DeleteFromBookmarks.kt" "domain/usecase/IsBookmarked.kt" "data/remote/ApiService.kt" "data/remote/dto/NewsApiResponseDto.kt" "data/remote/dto/ArticleDto.kt" "data/remote/dto/SourceDto.kt" "data/local/db/Database.kt" "data/local/db/DatabaseDriverFactory.kt" "data/repository/NewsRepositoryImpl.kt" "data/repository/BookmarkRepositoryImpl.kt" "data/mapper/DatabaseEntityMapper.kt" "data/mapper/ResponseDtoMapper.kt"
```

#### For Windows Machine
```kotlin
mkdir "domain","domain/entity","domain/repository","domain/usecase","data","data/remote","data/local","data/repository","data/mapper","data/remote/dto","data/local/db"
ni "domain/entity/NewsPage.kt","domain/entity/Article.kt","domain/entity/Source.kt","domain/repository/NewsRepository.kt","domain/repository/BookmarkRepository.kt","domain/usecase/GetNews.kt","domain/usecase/GetBookmarks.kt","domain/usecase/AddToBookmarks.kt","domain/usecase/DeleteFromBookmarks.kt","domain/usecase/IsBookmarked.kt","data/remote/ApiService.kt","data/remote/dto/NewsApiResponseDto.kt","data/remote/dto/ArticleDto.kt","data/remote/dto/SourceDto.kt","data/local/db/Database.kt","data/local/db/DatabaseDriverFactory.kt","data/repository/NewsRepositoryImpl.kt","data/repository/BookmarkRepositoryImpl.kt","data/mapper/DatabaseEntityMapper.kt","data/mapper/ResponseDtoMapper.kt"
```

## Domain Layer

### Entities

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/entity/NewsPage.kt
```kotlin
data class NewsPage( val page: Int,
                     val totalPages: Int,
                     val articles: List<Article>)
```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/entity/Article.kt
```kotlin
@Serializable
data class Article(
    val source: Source,
    val author: String?,
    val title: String,
    val description: String?,
    val url: String,
    val urlToImage: String?,
    val publishedAt: String,
    val content: String?
)

```

#### [Project Root]shared/src/commonMain/kotlin/com/trenser/newsapp/domain/entity/Source.kt
```kotlin
@Serializable
data class Source(
    val id: String,
    val name: String
)
```

### Repository interfaces

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/repository/NewsRepository.kt
```kotlin
interface NewsRepository {
    suspend fun getNews(sources: String, page: Int): NewsPage
    suspend fun searchNews(searchQuery: String, sources: String, page: Int): NewsPage
}
```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/repository/BookmarkRepository.kt
```kotlin
interface BookmarkRepository {
    fun getBookmarks(): Flow<List<Article>>
    suspend fun isBookmarked(article: Article):Boolean
    suspend fun saveBookmark(article: Article)
    suspend fun removeBookmark(article: Article)
}
```

### Usecases

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/usecase/GetNews.kt

```kotlin
class GetNews(
    private val newsRepository: NewsRepository
) {
    suspend operator fun invoke(sources: String, page: Int): NewsPage {
        return newsRepository.getNews(sources,page)
    }
}
```


#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/usecase/GetBookmarks.kt
```kotlin
class GetBookmarks(
    private val bookmarkRepository: BookmarkRepository
){
    operator fun invoke(): Flow<List<Article>> {
        return bookmarkRepository.getBookmarks()
    }
}

```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/usecase/AddToBookmarks.kt
```kotlin
class AddToBookmarks(
    private val bookmarkRepository: BookmarkRepository
){
    suspend operator fun invoke(article: Article){
        return bookmarkRepository.saveBookmark(article)
    }
}

```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/usecase/DeleteFromBookmarks.kt
```kotlin
class DeleteFromBookmarks(
    private val bookmarkRepository: BookmarkRepository
){
    suspend operator fun invoke(article: Article){
        return bookmarkRepository.removeBookmark(article)
    }
}

```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/domain/usecase/IsBookmarked.kt
```kotlin
class IsBookmarked(
    private val bookmarkRepository: BookmarkRepository
){
    suspend operator fun invoke(article: Article):Boolean{
        return bookmarkRepository.isBookmarked(article)
    }
}

```


## Data Layer

### Repository Implementation
#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/repository/NewsRepositoryImpl.kt

```kotlin
class NewsRepositoryImpl:NewsRepository {
    
}
```
```kotlin
class NewsRepositoryImpl:NewsRepository {
    override suspend fun getNews(sources: String, page: Int): NewsPage {
        TODO("Not yet implemented")
    }

    override suspend fun searchNews(searchQuery: String, sources: String, page: Int): NewsPage {
        TODO("Not yet implemented")
    }

}
```

## Network
### API Services 
#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/remote/ApiService.kt
```kotlin
class ApiService(private val client: HttpClient) {

    private val BASE_URL = "https://newsapi.org/v2/"
    private val API_KEY = "[Your API key here]"

    suspend fun getNews(sources: String,
                        page: Int,
                        apiKey: String = API_KEY): NewsApiResponseDto{
        return client.get("${BASE_URL}everything") {
            parameter("sources", sources)
            parameter("page", page)
            parameter("apiKey", apiKey)
        }.body()
    }

    suspend fun searchNews(searchQuery: String,
                           sources: String,
                           page: Int,
                           apiKey: String = API_KEY): NewsApiResponseDto{
        return client.get("${BASE_URL}everything") {
            parameter("q", searchQuery)
            parameter("sources", sources)
            parameter("page", page)
            parameter("apiKey", apiKey)
        }.body()
    }
}
```

### DTO classes
#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/remote/dto/NewsApiResponseDto.kt

```kotlin
@Serializable
data class NewsApiResponseDto(
    val status: String,
    val totalResults: Int,
    val articles: List<ArticleDto>
)
```
#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/remote/dto/ArticleDto.kt

```kotlin
@Serializable
data class ArticleDto(
    val author: String?,
    val content: String,
    val description: String,
    val publishedAt: String,
    val source: SourceDto,
    val title: String,
    val url: String,
    val urlToImage: String
)
```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/remote/dto/SourceDto.kt

```kotlin
@Serializable
data class SourceDto(
    val id: String,
    val name: String
)
```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/Platform.kt

```kotlin
expect fun createHttpClient(): HttpClient
```

### Android platform specific implementation
#### [Project Root]/shared/src/androidMain/kotlin/com/trenser/newsapp/Platform.android.kt
```kotlin
actual fun createHttpClient(): HttpClient {
    return HttpClient {
        install(ContentNegotiation) {
            json(Json {
                prettyPrint = true
                isLenient = true
                ignoreUnknownKeys = true

            })
        }
        install(HttpRedirect) {
            checkHttpMethod = false // Allow redirect for non-GET methods if needed
        }
    }
}
```

### IOS platform specific implementation
#### [Project Root]/shared/src/iosMain/kotlin/com/trenser/newsapp/Platform.ios.kt
```kotlin
actual fun createHttpClient(): HttpClient{
    return HttpClient(Darwin) {
        install(ContentNegotiation) {
            json(Json {
                prettyPrint = true
                isLenient = true
                ignoreUnknownKeys = true
            })
        }
        install(HttpRedirect) {
            checkHttpMethod = false // Allow redirect for non-GET methods if needed
        }
    }
}
```

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/repository/NewsRepositoryImpl.kt

```kotlin
class NewsRepositoryImpl:NewsRepository {

    private val apiService = ApiService(createHttpClient())

    override suspend fun getNews(sources: String,
                                 page: Int): NewsPage {
        val response = apiService.getNews(sources,page)
        return if (response.status == "ok") {
            response.toNewsPage(page)
        } else {
            throw Exception("")
        }
    }

    override suspend fun searchNews(searchQuery: String,
                                    sources: String,
                                    page: Int): NewsPage {
        val response = apiService.searchNews(searchQuery,sources,page)
        return if (response.status == "ok") {
            response.toNewsPage(page)
        } else {
            throw Exception("")
        }
    }
}
```


### DTO to Entity Mapper
#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/mapper/ResponseDtoMapper.kt
```kotlin
fun NewsApiResponseDto.toNewsPage(page: Int): NewsPage {
    return NewsPage(
        page=page,
        totalPages = totalResults,
        articles =  articles.map { it.toArticle() }
    )
}

fun ArticleDto.toArticle(): Article {
    return Article(source = source.toSource(),author, title, description, url, urlToImage, publishedAt, content)
}

fun SourceDto.toSource(): Source {
    return Source(id, name)
}
```



###  SQLDelight Database Scheme and Configuration
#### [Project Root]/shared/build.gradle.kts

-The code snippet you provided is a configuration block for SQLDelight, a library used for working with SQLite databases in Kotlin Multiplatform projects.

```kotlin
sqldelight {
    databases {
        create("AppDatabase") {
            packageName.set("com.trenser.newsapp")
        }
    }
}
```

#### [Project Root]/shared/src/commonMain

## Folder structure generation
#### For Mac Machine
```kotlin
mkdir "sqldelight" "sqldelight/com" "sqldelight/com/trenser" "sqldelight/com/trenser/newsapp"

touch "sqldelight/com/trenser/newsapp/AppDatabase.sq"
```

#### For Windows Machine
```kotlin
mkdir "sqldelight","sqldelight/com","sqldelight/com/trenser","sqldelight/com/trenser/newsapp"

ni "sqldelight/com/trenser/newsapp/AppDatabase.sq"
```

#### [Project Root]/shared/src/commonMain/sqldelight/com/trenser/newsapp/AppDatabase.sq

- SQLDelight Schema (`bookmarks.sq`):
  
  ```sql
    import kotlinx.serialization.descriptors.PrimitiveKind.BOOLEAN;
  
  
  -- Source.sq
  
  CREATE TABLE SourceEntity (
      id TEXT PRIMARY KEY,
      name TEXT NOT NULL
  );
  
  -- Article.sq
  
  CREATE TABLE ArticleEntity (
      title TEXT PRIMARY KEY,
      sourceId TEXT NOT NULL,
      author TEXT,
      description TEXT,
      url TEXT NOT NULL,
      urlToImage TEXT,
      publishedAt TEXT NOT NULL,
      content TEXT,
  
      FOREIGN KEY(sourceId) REFERENCES SourceEntity(id) ON DELETE CASCADE
  );
  
  getAllArticles:
  SELECT ArticleEntity.*, SourceEntity.*
  FROM ArticleEntity
  JOIN SourceEntity ON ArticleEntity.sourceId = SourceEntity.id;
  
  insertArticleEntity:
  INSERT OR REPLACE INTO ArticleEntity (title, sourceId, author, description, url, urlToImage, publishedAt, content)
  VALUES (?, ?, ?, ?, ?, ?, ?, ?);
  
  insertSourceEntity:
  INSERT OR REPLACE INTO SourceEntity (id, name)
  VALUES (?, ?);
  
  deleteArticleEntityByTitle:
  DELETE FROM ArticleEntity WHERE title = ?;
  
  getArticleByTitle:
  SELECT ArticleEntity.*, SourceEntity.*
  FROM ArticleEntity
  JOIN SourceEntity ON ArticleEntity.sourceId = SourceEntity.id
  WHERE ArticleEntity.title = ?;


    ```

### 7. Database Class 

#### [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/local/db/Database.kt
- The Database class manages interactions with your SQLDelight database, encapsulating database access and operations.
  
```kotlin
internal class Database(databaseDriverFactory: DatabaseDriverFactory) {
    private val database = AppDatabase(databaseDriverFactory.createDriver())
    private val dbQuery = database.appDatabaseQueries


    fun getBookmarks(): Flow<List<GetAllArticles>> {
        return dbQuery.getAllArticles().asFlow().mapToList(Dispatchers.IO)
    }

    fun addBookmark(article: Article){
        dbQuery.transaction {
            dbQuery.insertSourceEntity(article.source.id,article.source.name)
            dbQuery.insertArticleEntity(
                article.title,
                article.source.id,
                article.author,
                article.description,
                article.url,
                article.urlToImage,
                article.publishedAt,
                article.content)
        }
    }

    fun removeBookmark(article: Article){
        dbQuery.deleteArticleEntityByTitle(article.title)
    }

    fun isBookmarked(article: Article):Boolean{
        val articleByTitle = dbQuery.getArticleByTitle(article.title).executeAsOneOrNull()
        return articleByTitle != null
    }
}
```

  

  ### DatabaseDriverFactory Interface
  - The code defines an interface named DatabaseDriverFactory:
    -- his interface serves as a contract for creating SQL drivers. SQL drivers are responsible for interacting with the underlying database implementation
    
    - [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/local/db/DatabaseDriverFactory.kt
      
  ```kotlin
    interface DatabaseDriverFactory {
      fun createDriver(): SqlDriver
    }
  ```
    
- Initialize the SQLDelight database differently for Android and iOS.
- The createDatabase() function then uses this generated code to create an actual database instance on the device using the platform-specific driver.
    
   - [Project Root]/shared/src/iosMain/kotlin/com/trenser/newsapp/data/local/db/DatabaseDriverFactory.kt
   - For Android, use `AndroidSqliteDriver`:
     
    ```kotlin
    class AndroidDatabaseDriverFactory(private val context: Context) : DatabaseDriverFactory {
          override fun createDriver(): SqlDriver {
              return AndroidSqliteDriver(AppDatabase.Schema, context, "NewsApp.db")
          }
      }
    ```
    
    - [Project Root]/shared/src/androidMain/kotlin/com/trenser/newsapp/data/local/db/DatabaseDriverFactory.kt
    - For iOS, use `NativeSqliteDriver`:
      
    ```kotlin
    class IOSDatabaseDriverFactory : DatabaseDriverFactory {
      override fun createDriver(): SqlDriver {
          return NativeSqliteDriver(AppDatabase.Schema, "NewsApp.db")
      }
  }
      
    ```

- [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/repository/BookmarkRepositoryImpl.kt
  
### Implementation of repository 
  
  ```kotlin
class BookmarkRepositoryImpl(val databaseDriverFactory: DatabaseDriverFactory): BookmarkRepository {
    private val database = Database(databaseDriverFactory)

    override fun getBookmarks(): Flow<List<Article>> {
        return database
            .getBookmarks()
            .map { getAllArticles ->
                getAllArticles.map { user ->
                    user.toArticle()
                }
            }
    }

    override suspend fun isBookmarked(article: Article): Boolean {
        return database.isBookmarked(article)
    }

    override suspend fun saveBookmark(article: Article) {
        return database.addBookmark(article)
    }

    override suspend fun removeBookmark(article: Article) {
        return database.removeBookmark(article)
    }

}
   ```

### toArticle() Extension Function

The `toArticle()` extension function is used to convert an instance of `GetAllArticles` (likely a data class representing an article retrieved from the database) into an instance of `Article` (likely a domain model class).

- [Project Root]/shared/src/commonMain/kotlin/com/trenser/newsapp/data/mapper/DatabaseEntityMapper.kt
  
#### DatabaseEntityMapper.kt
```kotlin
    fun GetAllArticles.toArticle():Article{
    return Article(
        source = Source(id,name),
        author = author,
        title = title,
        description = description,
        url = url,
        urlToImage = urlToImage,
        publishedAt = publishedAt,
        content = content
    )
}
```







