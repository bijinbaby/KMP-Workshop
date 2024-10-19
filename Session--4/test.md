#### [Project Root]/shared/src/commonTest/ExampleUnitTest.kt
```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals

// Function to be tested
fun add(a: Int, b: Int): Int {
    return a + b
}

// Unit test for the add function
class MathUtilsTest {

    @Test
    fun testAddition() {
        val result = add(2, 3)
        assertEquals(5, result)
    }

}

```

#### [Project Root]/shared/src/commonTest/ExampleUnitTest.kt
```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals

// Unit test for the add function
class MathTest {

    @Test
    fun addition() {
        assertEquals(4, 2 + 2)
    }
}

```



#### [Project Root]/shared/src/commonTest/domain/usecase/GetNewsTest.kt
```kotlin
import com.trenser.newsapp.data.repository.NewsRepositoryImpl
import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertNotEquals

// Unit test for the add function
// Total - 1453
class NewsTest {

    @Test
    fun testGetNews() = runTest {
        val newsRepository = NewsRepositoryImpl()
        val news = newsRepository.getNews("bbc-news", 1)
        println("Total ${news.totalPages}")
        assertNotEquals(0, news.totalPages)
    }

}
```
