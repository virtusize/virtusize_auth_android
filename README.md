Virtusize Auth SDK for Android

## Description

The Virtusize Auth SDK for Android is a closed-source library that handles the SNS authentication process for our [Virtusize Android Integration](https://github.com/virtusize/integration_android).

## Getting Started

1. In your root build.gradle file, edit the `repositories` section to include the following:

```groovy
allprojects {
    repositories {
        maven {
            url "https://github.com/virtusize/virtusize_auth_android/raw/main"
        }
    }
}
```

2. In your root build.gradle file, edit the `dependencies` section to include the following:

```groovy
dependencies {
    implementation "com.virtusize.android:virtusize_auth_android:1.0.1"
}
```
