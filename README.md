# Android Spotify SDK Tutorial

## 1. Make a new project

To make your new project, get started by creating an empty views activity using `Java` and `Groovy DSL` as the build configuration language.

We highly recommend using `API 33 ("Tiramisu")` to avoid dependency issues and to get the best support from TA team.

<img src="./img/setup.png" width=800 />

## 2. Setting up your Gradle Files

To integrate the Spotify API into your Android project, we need to include the following external dependencies in your Gradle file:

```gradle
dependencies {
    ...
    implementation 'com.spotify.android:auth:2.1.1'
    implementation 'com.squareup.okhttp3:okhttp:4.9.3'
}
```

The Spotify Auth package is the official method of interfacing with the Spotify API on Android and removes a lot of the complexity in authenticating requests. The `OkHttp package` simplifies making requests to the API.

We now need to configure the `manifestPlaceholders` so that the redirect URI for Spotify Authentication works properly. **The `redirectSchemeName` can be anything. We prefer your App Name!**

```gradle
manifestPlaceholders = [redirectSchemeName: "<Insert App Name>", redirectHostName: "auth"]
```

After all of this, your Gradle file should look similar to the example below. Make sure to sync your Gradle project before moving on!

```gradle
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.example.spotify_sdk'  // <<---- YOUR PACKAGE NAME
    compileSdk 33

    defaultConfig {
        applicationId "com.example.spotify_sdk"
        minSdk 33
        targetSdk 33
        versionCode 1
        versionName "1.0"

        manifestPlaceholders = [redirectSchemeName: "SPOTIFY-SDK", redirectHostName: "auth"]

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation 'com.spotify.android:auth:2.1.1'
    implementation 'com.squareup.okhttp3:okhttp:4.9.3'

    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
```

## Setup AndroidManifest.xml

```xml
...
<activity
    android:name=".MainActivity"
    android:exported="true">

    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

    <intent-filter>
        // Match the scheme and host of the redirect URI
        // Must be the same in MainAcitivity, Gradle, and Spotify Dashboard!
        <data android:host="auth" android:scheme="SPOTIFY-SDK"/>
    </intent-filter>

</activity>
...
```

## 3. Register Your App on Spotify for Developers

1. Go to [Spotify for Developer Dashboard](https://developer.spotify.com/dashboard)
2. Click on `Create App` on the top right. On the next screen, fill out some necessary information:
    - `App Name`: Insert your app name
    - `App description`: Describe what your app does
    - `Redirect URIs`: [redirectSchemeName]://[redirectHostName]
        - Check your `manifestPlaceholders`!
        - As seen above, redirectSchemeName = “**SPOTIFY-SDK**” and redirectHostName = “**auth**”
        - Therefore, my redirect URI is `SPOTIFY-SDK://auth`
    - `APIs used`: select `Android`, `Web Playback SDK`, and `Web API`
3. Read Spotify's Developer Terms of Service and Design Guidelines carefully (😂). Click on “I understand and agree with Spotify's Developer Terms of Service and Design Guidelines”
4. Click `Save` to create your application
5. Click `Setting` to retrieve the `CLIENT-ID` for your application. This is necessary for connecting to the API and will be used in the demo code below

<img src="./img/sdk2.png" width=800 />

### Redirect URI Modifications

You can modify change the redirect URI at any time on the Spotify Developer Dashboard. However, it is important to ensure that it matches in all relevant places if you change it! See the image below for an explanation.

<img src="./img/redirect_uri.png" width=800 />

## 4. Create Demo App

### 1. Create Basic Layout

Inside an empty `ConstraintLayout`, we will populate with a Linear Layout containing 3 buttons and 3 TextView for the purposes of showing some basic information such as Token, Code, and User Profile you can get from the API request.

[You can copy the XML layout from here](./activity_main.xml)

### 2. Create Basic Activity

We provide ActivityMain.java file that provides a simple demo of how to interact with the API in Java. We highly recommend factoring any API interfacing functions into a separate class from the activities to avoid coupling issues, but to keep the demo simple we avoiding doing that here.

**Make sure to plug in your Client ID and redirect URI into the Java file before running the demo!**

[You can copy the Java code from here](./MainActivity.java)

### 3. Sample Output

If you are having any issues with the API, please check the "Troubleshotting Tips" section before asking any TAs for help

<img src="./img/sample.png" width=400 />

## API Request Explanation: Fetch User's Profile

### 1. Look at Spotify's documentation for User Profile

[You can view the User profile documentation here](https://developer.spotify.com/documentation/web-api/reference/get-current-users-profile). When attempting to make API requests, the documentation is your greatest tool!

### 2. Obtain Authorization Token

Before making any requests to the Spotify API, you need to obtain an authorization token used to authenticate your requests. However, not all tokens are created equally. You need to specify the scope of the token you want! To figure out what scope you should request, look at the "Authorization scopes" section of a request in the documentation.

Scopes are used to limit access of some parts of the API to tokens without the necessary permissions. Some API calls will require more/less invasive scopes for security reasons! [You can read more about Scopes here.](https://developer.spotify.com/documentation/web-api/concepts/scopes)

[In Spotify’s documentation for the User Profile](https://developer.spotify.com/documentation/web-api/reference/get-current-users-profile), there are 2 scopes we need to consider in the request (picture attached below). We will use `user-read-email` over `user-read-private` since we don’t want to display subscription details.

<img src="./img/sdk3.png" width=300 />

Below is how the demo code requested a token of scope `user-read-email`

```java
private AuthorizationRequest getAuthenticationRequest(AuthorizationResponse.Type type) {
       return new AuthorizationRequest.Builder(CLIENT_ID, type, getRedirectUri().toString())
               .setShowDialog(false)
               .setScopes(new String[] { "user-read-email" })
               .setCampaign("your-campaign-token")
               .build();
   }
```

### 3. Get the Endpoint URL and Authorization Header

On the documentation page, locate the "Request with Authorization" section. This section provides details on how to construct a request to retrieve a user's profile.

<img src="./img/request.png" width=800 />

<br />

The `Request with Authorization` includes 2 parts:

-   Endpoint URL: This is the URL where you send your request to retrieve the user's profile. In this case, it is https://api.spotify.com/v1/me.
-   Authorization Header: **This header includes the authorization token**. It should be formatted as `"Bearer <authorization_token>"`

### 4. Implement the Request in Java

Below is how the demo code combined the `Endpoint URL` and the `Authorization Header` in a Request

```java
// Create a request to get the user profile
        final Request request = new Request.Builder()
                .url("https://api.spotify.com/v1/me")
                .addHeader("Authorization", "Bearer " + mAccessToken)
                .build();
```

## What to do next?

#### 1. Read over the [Spotify Web API](https://developer.spotify.com/documentation/web-api) documentation to get a better sense of the API's capabilities and what information you can request.

<img src="./img/webapi.png" width=800 />

#### 2. Think about how to parse and visualize the information from the API

Example:

<img src="./img/sample.jpeg" width=800 />

## Troubleshooting Tips:

The following guides cover most errors you will run into. If you still have issues after following them, check on Ed Discussion for any posts with solutions for your specific error.

[**If you still can't find a solution, please open an issue on this repo with the following information:**](https://github.com/thuanvoit/spotify-sdk-tutorial/issues)


### Issues Getting Token/Code from Spotify

Your issues may include
-  Blank screen with no redirect when attempting to get token/code
- "Redirect URI Mismatch" error
-  Blank Spotify page
-  Unable to press login button
-  Unable to grant permission
-  Stuck in Chrome browser

<details>
<summary><strong>Click here for instructions on fixing these issues</strong></summary>

#### 1. **Please follow the instructions below step by step! There is no need to continue if one step fixes your problems!**

#### 2.  Verify You Don't Have a URI Mismatch

-   If you encounter a "Redirect URI Mismatch" error, make sure that the redirect URI specified in your Spotify Developer Dashboard matches in your:
    -   Gradle file
    -   Java files
    -   AndroidManifest.xml file
-   Triple-check all redirect URIs match before proceeding!


#### 3. Use your `PACKAGE_NAME` as redirectSchemeName

-   Check if you are using `redirectSchemeName` = **spotify-sdk** or any other non-package name
-   Replace redirectSchemeName with your APP_PACKAGE_NAME (can be found in your Gradle file)


#### 4. Manually Add the Android Package to your Dashboard:

-   Go to Android Studio, Open Gradle manager on right sidebar, run the command:

```bash
gradle signingreport
```

<img src='./img/gradle-key.png' width=800 />

A terminal will open with SHA1 like below:

<img src='./img/sha1.png' width=800 />

Copy `PACKAGE_NAME` and the `SHA1` key and put it on Spotify App's Developer Setting.

<img src='./img/dev-sha1.png' width=800 />

</details>

### Emulator Crashes, DNS errors, Browser Slow/Blank at Authentication

<details>
<summary><strong>Click here for instructions on fixing these issues</strong></summary>

#### 1. **Please follow the instructions below step by step! There is no need to continue if one step fixes your problems!**

#### 2. Close all unnecessary applications to reduce memory/CPU use

#### 3. Delete your previous emulator device and make a new one, preferably based off an older phone like the Pixel 3A

#### 4. Edit your emulator settings to increase your VM heap. Default VM heap is about ~200 MB, increase it to 512 MB or higher

<img width="618" alt="Screenshot 2024-03-25 at 2 45 06 PM" src="https://github.com/thuanvoit/spotify-sdk-tutorial/assets/21033636/56ab1843-2e69-4a2f-a66b-55238eab61dd">

</details>

### 429 Error Response

<details>
<summary><strong>Click here for instructions on fixing this issue</strong></summary>

-   Spotify's Web API implements rate limits to ensure the reliability of its services and to promote responsible usage among third-party developers. The rate limit is calculated based on the number of API calls made by an application within a rolling 30-second window.

-   When an application surpasses Spotify's rate limit, it receives a 429 error response from the Web API. This indicates that the application has reached its rate limit and needs to throttle its requests.

-   For more details about rate limits and strategies to overcome the error, please review [Spotify Rate Limit](https://developer.spotify.com/documentation/web-api/concepts/rate-limits)

</details>
