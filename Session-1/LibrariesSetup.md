# Session 1
## Set Dependecies in the project</summary>

### [Project Root]/gradle/libs.versions.toml</summary>

```kotlin
[versions]
agp = "8.5.2"
kotlin = "2.0.0"
compose = "1.6.8"
compose-material3 = "1.2.1"
androidx-activityCompose = "1.9.1"

coroutines = "1.9.0-RC"
ktorVersion = "2.3.7"
navigationCompose = "2.7.7"
sqldelight = "2.0.0"
koin-bom = "3.5.3"
skie = "0.8.4"        

[libraries]
kotlin-test = { module = "org.jetbrains.kotlin:kotlin-test", version.ref = "kotlin" }
androidx-activity-compose = { module = "androidx.activity:activity-compose", version.ref = "androidx-activityCompose" }
compose-ui = { module = "androidx.compose.ui:ui", version.ref = "compose" }
compose-ui-tooling = { module = "androidx.compose.ui:ui-tooling", version.ref = "compose" }
compose-ui-tooling-preview = { module = "androidx.compose.ui:ui-tooling-preview", version.ref = "compose" }
compose-foundation = { module = "androidx.compose.foundation:foundation", version.ref = "compose" }
compose-material3 = { module = "androidx.compose.material3:material3", version.ref = "compose-material3" }

ktor-client-core-v237 = { module = "io.ktor:ktor-client-core", version.ref = "ktorVersion" }
ktor-client-android = { module = "io.ktor:ktor-client-android", version.ref = "ktorVersion" }
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktorVersion" }
ktor-client-content-negotiation-v237 = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktorVersion" }
ktor-serialization-kotlinx-json-v237 = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktorVersion" }

kotlinx-coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutines" }
kotlinx-coroutines-android = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android", version.ref = "coroutines" }
kotlinx-coroutines-test = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-test", version.ref = "coroutines" }

sqldelight-android-driver = { module = "app.cash.sqldelight:android-driver", version.ref = "sqldelight" }
sqldelight-native-driver = { module = "app.cash.sqldelight:native-driver", version.ref = "sqldelight" }
sqldelight-runtime = { module = "app.cash.sqldelight:runtime", version.ref = "sqldelight" }
coroutines-extensions = { module = "app.cash.sqldelight:coroutines-extensions", version.ref = "sqldelight" }

koin-bom = { module = "io.insert-koin:koin-bom", version.ref = "koin-bom" }
koin-core = { module = "io.insert-koin:koin-core" }
koin-android = { module = "io.insert-koin:koin-android" }
koin-androidx-compose = { module = "io.insert-koin:koin-androidx-compose" }


[plugins]
androidApplication = { id = "com.android.application", version.ref = "agp" }
androidLibrary = { id = "com.android.library", version.ref = "agp" }
kotlinAndroid = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlinMultiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlinCocoapods = { id = "org.jetbrains.kotlin.native.cocoapods", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }

sqldelight = {id ="app.cash.sqldelight", version.ref = "sqldelight"}
skie = { id = "co.touchlab.skie", version.ref = "skie" }
```


### [Project Root]/build.gradle.kts
```kotlin
plugins {
    //trick: for the same plugin versions in all sub-modules
    alias(libs.plugins.androidApplication).apply(false)
    alias(libs.plugins.androidLibrary).apply(false)
    alias(libs.plugins.kotlinAndroid).apply(false)
    alias(libs.plugins.kotlinMultiplatform).apply(false)
    alias(libs.plugins.compose.compiler).apply(false)

    alias(libs.plugins.sqldelight) apply false
    alias(libs.plugins.skie) apply false
}
```

### [Project Root]/shared/build.gradle.kts
```kotlin
import org.jetbrains.kotlin.gradle.ExperimentalKotlinGradlePluginApi
import org.jetbrains.kotlin.gradle.dsl.JvmTarget

plugins {
    alias(libs.plugins.kotlinMultiplatform)
    alias(libs.plugins.androidLibrary)

    kotlin("plugin.serialization") version "2.0.0"
    alias(libs.plugins.sqldelight)
    alias(libs.plugins.skie)
}

kotlin {
    androidTarget {
        @OptIn(ExperimentalKotlinGradlePluginApi::class)
        compilerOptions {
            jvmTarget.set(JvmTarget.JVM_1_8)
        }
    }
    
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach {
        it.binaries.framework {
            baseName = "shared"
            isStatic = false
        }
    }

    sourceSets {
        commonMain.dependencies {
            //put your multiplatform dependencies here
            implementation(libs.ktor.client.core.v237)
            implementation(libs.ktor.client.content.negotiation.v237)
            implementation(libs.ktor.serialization.kotlinx.json.v237)

            implementation(libs.kotlinx.coroutines.core)

            implementation(libs.sqldelight.runtime)
            implementation(libs.coroutines.extensions)

            implementation(project.dependencies.platform(libs.koin.bom))
            implementation(libs.koin.core)
        }

        androidMain.dependencies {
            implementation(libs.ktor.client.android)

            implementation(libs.kotlinx.coroutines.android)

            implementation(libs.sqldelight.android.driver)

            implementation(libs.koin.androidx.compose)
        }

        iosMain.dependencies {
            implementation(libs.ktor.client.darwin)

            implementation("co.touchlab:stately-common:2.0.6")
            implementation(libs.sqldelight.native.driver)
        }

        commonTest.dependencies {
            implementation(libs.kotlin.test)
            implementation(libs.kotlinx.coroutines.test)
        }
    }
}

android {
    namespace = "com.trenser.newsapp"
    compileSdk = 34
    defaultConfig {
        minSdk = 29
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
}

//sqldelight {
//    databases {
//        create("AppDatabase") {
//            packageName.set("com.trenser.newsapp")
//        }
//    }
//}
```

### [Project Root]/androidApp/build.gradle.kts
```kotlin
plugins {
    alias(libs.plugins.androidApplication)
    alias(libs.plugins.kotlinAndroid)
    alias(libs.plugins.compose.compiler)
}

android {
    namespace = "com.trenser.newsapp.android"
    compileSdk = 34
    defaultConfig {
        applicationId = "com.trenser.newsapp.android"
        minSdk = 29
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
    buildFeatures {
        compose = true
    }
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
    buildTypes {
        getByName("release") {
            isMinifyEnabled = false
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {
    implementation(projects.shared)
    implementation(libs.compose.ui)
    implementation(libs.compose.ui.tooling.preview)
    implementation(libs.compose.material3)
    implementation(libs.androidx.activity.compose)
    debugImplementation(libs.compose.ui.tooling)
}

dependencies{
    //Paging 3
    implementation("androidx.paging:paging-runtime:3.3.2")
    implementation("androidx.paging:paging-compose:3.3.2")

    //Coil
    implementation("io.coil-kt:coil-compose:2.4.0")

    //koin
    implementation(libs.koin.android)
    implementation("io.insert-koin:koin-androidx-compose:3.4.2")

    //Navigation
    //implementation(libs.androidx.navigation.compose)
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-core:1.6.3")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.3")
}
```
