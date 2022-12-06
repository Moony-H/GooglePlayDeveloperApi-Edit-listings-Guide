# Google Play Developer Api Edit-listings Guide

이번에는 Edit 중에서 listings를 사용하는 API를 설명하겠습니다.


## 사용 환경

이번 글에서 사용할 언어는 Kotlin 입니다. compiler로는 Intellij CE를 사용합니다.

또한 Retrofit2를 이용하여 Rest API를 호출할 것입니다.

이 글을 보기 전에 앞서 설명한 [Google Play Console API Guide](https://github.com/Moony-H/GooglePlayConsoleAPIGuide)의 내용을 먼저 보시기 바랍니다.

또한 Edits API의 이해와 소스 코드가 필요합니다.

[Google Play Developer API - Edit Guide](https://github.com/Moony-H/GooglePlayDeveloperApi-Edit-Guide)를 먼저 보시기 바랍니다.

## listings 설명

listings는 앱의 제목, 자세한 설명, 간단한 설명 등 기본적인 앱의 스토어 등록 정보를 다루는 API 입니다.

이를 활용하여 국가 언어 별 등록 정보를 얻고 삭제, 수정 등의 행동을 할 수 있습니다.



## listings get

앱의 등록 정보인 listings를 가져오는 API 입니다.

먼저 아래와 같은 data class를 생성합니다.

**Listing.kt**

```kotlin
package data.postbody

import com.google.gson.annotations.SerializedName

data class Listing(
    @SerializedName("language")
    val language:String,
    @SerializedName("title")
    val title:String,
    @SerializedName("fullDescription")
    val fullDescription:String,
    @SerializedName("shortDescription")
    val shortDescription:String,
    @SerializedName("video")
    val video:String?
)
```

그 다음 새로운 interface를 생성하여 아래와 같이 메소드를 작성합니다.

**ListingAPI**

```kotlin
import data.Listing
import retrofit2.Response
import retrofit2.http.GET
import retrofit2.http.Header
import retrofit2.http.Path

interface ListingAPI {
    @GET("{packageName}/edits/{editId}/listings/{language}")
    suspend fun getEditsListing(
        @Header("Authorization")
        token: String,
        @Path("packageName")
        packageName: String,
        @Path("editId")
        editId: String,
        @Path("language")
        language: String,
    ): Response<Listing>
}
```

그 다음 전에 작성했던 코드를 아래와 같이 수정합니다.

**Main.kt**

```kotlin
import api.EditsAPI
import api.ListingAPI
import com.google.auth.oauth2.ServiceAccountCredentials
import com.jakewharton.retrofit2.adapter.kotlin.coroutines.CoroutineCallAdapterFactory
import kotlinx.coroutines.*
import okhttp3.OkHttpClient
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.io.FileInputStream
import kotlin.system.exitProcess

fun main(args: Array<String>) {

    runBlocking {
        val token = getAccessToken()

        val editsRetrofit = Retrofit.Builder()
            .baseUrl("https://androidpublisher.googleapis.com/androidpublisher/v3/applications/")
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(CoroutineCallAdapterFactory())
            .client(OkHttpClient.Builder().build())
            .build()
            .create(EditsAPI::class.java)

        val listingRetrofit=Retrofit.Builder()
            .baseUrl("https://androidpublisher.googleapis.com/androidpublisher/v3/applications/")
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(CoroutineCallAdapterFactory())
            .client(OkHttpClient.Builder().build())
            .build()
            .create(ListingAPI::class.java)

        val editInsertResponse = editsRetrofit.postInsertEdit("Bearer $token", "com.dpectrum.holymolysheet")

        println("edit insert response code: ${editInsertResponse.code()}")
        println("edit insert response body: ${editInsertResponse.body()}")

        val editInsertBody= editInsertResponse.body() ?: exitProcess(0)




        val editValidateResponse=editsRetrofit.postValidateEdit("Bearer $token","com.dpectrum.holymolysheet",editInsertBody.id)

        println("edit validate response code: ${editValidateResponse.code()}")
        println("edit validate response body: ${editValidateResponse.body()}")

        val listingGetResponse=listingRetrofit.getEditsListing("Bearer $token", "com.dpectrum.holymolysheet",editInsertBody.id,"ko-KR")


        println("listing get response code: ${listingGetResponse.code()}")
        println("listing get response body: ${listingGetResponse.body()}")


        exitProcess(0)
    }


}

fun getAccessToken(): String {
    val credentials =
        ServiceAccountCredentials.fromStream(FileInputStream("/Users/hanmunhwi/Desktop/Google Play Console Key/pc-api-5791105689854140514-65-de5fdf8795d0.json"))
            .createScoped(setOf("https://www.googleapis.com/auth/androidpublisher"))
    credentials.refreshIfExpired()
    return credentials.accessToken.tokenValue
}

```

위와 같이 되었다면 앱의 기본 등록 정보인 listing을 받을 수 있습니다.

