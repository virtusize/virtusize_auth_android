# Virtusize Auth SDK for Android

## Description

The Virtusize Auth SDK for Android is a closed-source library that handles the SNS authentication
process for our [Virtusize Android Integration](https://github.com/virtusize/integration_android).

## Requirements

- minSdkVersion >= 21

- compileSdkVersion >= 34

- Setup in AppCompatActivity

## Getting Started

### 1. Installation

1. In your *root* build.gradle file or settings.gradle file, edit the `repositories` section to include the following:

- Groovy (build.gradle)

  ```groovy
  allprojects {
      repositories {
          maven {
              url "https://github.com/virtusize/virtusize_auth_android/raw/main"
          }
      }
  }
  ```

- Kotlin (settings.gradle.kts)

  ```kotlin
  dependencyResolutionManagement {
      repositories {
          maven { url = uri("https://github.com/virtusize/virtusize_auth_android/raw/main") }
      }
  }
  ```

2. In your *app* build.gradle file, edit the `dependencies` section to include the following:

- Groovy (build.gradle)

  ```groovy
  dependencies {
      implementation "com.virtusize.android:virtusize-auth:1.0.6"
  }
  ```

- Kotlin (build.gradle.kts)

  ```kotlin
  dependencies {
      implementation("com.virtusize.android:virtusize-auth:1.0.6")
  }
  ```

### 2. Create a Custom URL Scheme for Virtusize SNS Auth

The SNS authentication flow requires opening a Chrome Custom Tab, which will load a web page for the
user to login with their SNS account. In order to return the login response from a Chrome Custom Tab
to your app, a custom URL scheme must be defined inside the manifest.

Edit your `AndroidManifest.xml` file to include an intent filter and a `<data>` tag for the custom
URL scheme.

```xml

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.your-company.your-app">

    <activity android:name="com.virtusize.android.auth.views.VitrusizeAuthActivity"
        android:launchMode="singleTask" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />

            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />

            <data android:host="sns-auth" android:scheme="com.your-company.your-app.virtusize" />
        </intent-filter>
    </activity>

</manifest>
```

**❗IMPORTANT**

1. The URL host has to be `sns-auth`
2. The URL scheme must begin with your app's package ID (com.your-company.your-app) and **end with
   .virtusize**
3. The scheme which you define must use all **lowercase** letters

## Enable Virtusize SNS Login for your WebView app

Use either of the following methods to enable Virtusize SNS login

### Method 1: Use the VirtusizeWebView

##### **Step 1: Replace your `WebView` with `VirtusizeWebView`**

To enable users to sign up or log in with the web version of Virtusize integration in your webview,
please replace your `WebView` with **`VirtusizeWebView`** in your Kotlin or Java file and XML file
to fix and enable SNS login in Virtusize.

- Kotlin/Java

  ```diff
  // Kotlin
  - var webView: WebView
  + var webView: VirtusizeWebView
  
  // Java
  - WebView webView;
  + VirtusizeWebView webView;
  ```

and

- XML

  ```diff
  - <WebView
  + <com.virtusize.libsource.VirtusizeWebView
      android:id="@+id/webView"
      android:layout_width="match_parent"
      android:layout_height="match_parent" />
  ```

##### Step 2: Set the Virtusize SNS auth activity result launcher to `VirtusizeWebView`

- Kotlin

  ```kotlin
  // Register the Virtusize SNS auth activity result launcher
  private val virtusizeSNSAuthLauncher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
      // Handle the SNS auth result of the VirtusizeAuthActivity by passing the webview and the result to the `VirtusizeAuth.handleVirtusizeSNSAuthResult` function                                                                                                
      VirtusizeAuth.handleVirtusizeSNSAuthResult(webView, result.resultCode, result.data)
  }
  
  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
      super.onViewCreated(view, savedInstanceState)
      // ... other code
    
      // Set the activity result launcher to the webView
      webView.setVirtusizeSNSAuthLauncher(virtusizeSNSAuthLauncher)
  }
  ```

- Java

  ```java
  // Register the Virtusize SNS auth activity result launcher
  private ActivityResultLauncher<Intent> mLauncher = 
      registerForActivityResult(
          new ActivityResultContracts.StartActivityForResult(), 
          (ActivityResultCallback<ActivityResult>) result ->
              VirtusizeAuth.INSTANCE.handleVirtusizeSNSAuthResult(webView, result.getResultCode(), result.getData())
  );
  
  
  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
      super.onViewCreated(view, savedInstanceState)
      // ... other code
    
      // Set the activity result launcher to the webView
      webView.setVirtusizeSNSAuthLauncher(virtusizeSNSAuthLauncher)
  }
  ```

### or

### Method 2: Use WebView

##### Step 1: Make sure your WebView enables the following settings:

```kotlin
webView.settings.javaScriptEnabled = true
webView.settings.domStorageEnabled = true
webView.settings.databaseEnabled = true
webView.settings.setSupportMultipleWindows(true)
```

##### Step 2: Add the following code

```kotlin
// Register the Virtusize SNS auth activity result launcher
private val virtusizeSNSAuthLauncher =
    registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
        VirtusizeAuth.handleVirtusizeSNSAuthResult(webView, result.resultCode, result.data)
    }

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    webView.webViewClient = object : WebViewClient() {
        override fun onPageFinished(view: WebView?, url: String?) {
            // Enable SNS buttons in Virtusize
            webView.evaluateJavascript("javascript:window.virtusizeSNSEnabled = true;", null)

            // The rest of your code ..... 
        }
    }

    webView.webChromeClient = object : WebChromeClient() {
        override fun onCreateWindow(
            view: WebView,
            dialog: Boolean,
            userGesture: Boolean,
            resultMsg: Message
        ): Boolean {
            // Obtain the popup window link or link title
            val message = view.handler.obtainMessage()
            view.requestFocusNodeHref(message)
            val url = message.data.getString("url")
            val title = message.data.getString("title")
            if (resultMsg.obj != null && resultMsg.obj is WebView.WebViewTransport && VirtusizeURLCheck.isLinkFromVirtusize(url, title)) {
                val popupWebView = WebView(view.context)
                popupWebView.settings.javaScriptEnabled = true
                popupWebView.webViewClient = object : WebViewClient() {
                    override fun shouldOverrideUrlLoading(view: WebView, url: String): Boolean {
			if (VirtusizeURLCheck.isExternalLinkFromVirtusize(url)) {
                            runCatching {
                                val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
                                startActivity(intent)
                                return true
                            }
                        }
                        return VirtusizeAuth.isSNSAuthUrl(context, virtusizeSNSAuthLauncher, url)
                    }
                }
                popupWebView.webChromeClient = object : WebChromeClient() {
                    override fun onCloseWindow(window: WebView) {
                        webView.removeAllViews()
                    }
                }
                val transport = resultMsg.obj as WebView.WebViewTransport
                view.addView(popupWebView)
                transport.webView = popupWebView
                resultMsg.sendToTarget()
                return true
            }

            // The rest of your code ..... 

            return super.onCreateWindow(view, dialog, userGesture, resultMsg)
        }
    }
}
```
