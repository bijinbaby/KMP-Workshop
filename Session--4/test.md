

#### [Project Root]/shared/src/commonTest/domain/usecase/ExampleUnitTest.kt
```kotlin
@Test
fun addition_isCorrect() {
    assertEquals(4, 2 + 2)
}

```



#### [Project Root]/shared/src/commonTest/domain/usecase/GetNewsTest.kt
```kotlin
@Test
fun testGetNews() = runTest {
    val newsRepository = NewsRepositoryImpl()
    val news = newsRepository.getNews("bbc-news",1)
    assertNotEquals(0,news.totalPages)
}

```
