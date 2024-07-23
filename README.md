# CI/CD Pipeline for Flutter Apps Development

## Overview
This guide outlines the process of setting up a CI/CD pipeline for a Flutter mobile application using Fastlane, GitHub Actions, and Firebase App Distribution. The pipeline automates building, testing, and distributing the app, streamlining development and ensuring rapid, consistent deployments.

## Table of Contents
- [Setting Up Flavors in Flutter](#setting-up-flavors-in-flutter)
- [Firebase Configuration](#firebase-configuration)
- [Installing Fastlane](#installing-fastlane)
- [Integrating Firebase with Fastlane](#integrating-firebase-with-fastlane)
- [Setting Up GitHub Actions](#setting-up-github-actions)
- [Running the Pipeline](#running-the-pipeline)
- [Conclusion](#conclusion)
- [References](#references)

## Setting Up Flavors in Flutter

### `..\.vscode\launch.json` Configuration
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "DocDoc Development",
            "request": "launch",
            "type": "dart",
            "program": "lib/main_development.dart",
            "args": ["--flavor", "Development", "--target", "lib/main_development.dart"]
        },
        {
            "name": "DocDoc Production",
            "request": "launch",
            "type": "dart",
            "program": "lib/main_production.dart",
            "args": ["--flavor", "Production", "--target", "lib/main_production.dart"]
        }
    ]
}
```
### `..\android\app\build.gradle` Configuration
```groovy
flavorDimensions "default"
productFlavors {
    production {
        dimension "default"
        resValue "string", "app_name", "DocDoc Production"
    }
    development {
        dimension "default"
        applicationIdSuffix ".dev"
        resValue "string", "app_name", "DocDoc Development"
    }
}
```

### `..\android\app\src\main\AndroidManifest.xml` Configuration
```xml
<application
    android:label="@string/app_name"
    ...>
    ...
</application>
```

Build Command
```sh
flutter run --release -t lib/main_production.dart --flavor production
```

## Firebase Configuration

1. Create a new Firebase project on the Firebase console.
   
2. Login to Firebase:
   
   - ```sh
     firebase login
     ```
     
3. Install and configure FlutterFire CLI:
   
   - ```sh
     dart pub global activate flutterfire_cli
     flutterfire configure
     flutter pub add firebase_core
     flutterfire configure
     ```

## Installing Fastlane

1. Install Ruby:

   - [Ruby Installer](https://rubyinstaller.org/)

2. Install Fastlane:
   
   - ```sh
     cd android
     fastlane init
     ```

## Integrating Firebase with Fastlane

1. Add the Firebase App Distribution plugin:

- ```sh
     fastlane add_plugin firebase_app_distribution
     ```
   
2. Login to Firebase CLI:
   
   - ```sh
     firebase login:ci
     ```
     Save the token for later use.
     
3. The Fastfile will be automatically created, update it with the following:
   
   - ```ruby
     default_platform(:android)

     platform :android do
       desc "Lane for Android Firebase App Distribution"
       lane :firebase_distribution do
          sh "flutter clean"
          sh "flutter build apk --release --flavor production --target lib/main_production.dart --no-tree-shake-icons"
          firebase_app_distribution(
            app: "1:383408612193:android:da0f9d86c0a8051e4c5b20",
            firebase_cli_token: ENV["FIREBASE_CLI_TOKEN"],
            android_artifact_type: "APK",
            android_artifact_path: "../build/app/outputs/flutter-apk/app-production-release.apk",
            testers: "omar.ameer2333@gmail.com",
            release_notes: "YOUR_RELEASE_NOTES_HERE"
          )
        end
      end
     ```

## Setting Up GitHub Actions

1. Secure your data in GitHub Secrets:

   - Firebase options (4 secrets)
     - FIREBASE_OPTIONS_ANDROID_API_KEY
     - FIREBASE_OPTIONS_ANDROID_APP_ID
     - FIREBASE_OPTIONS_IOS_API_KEY
     - FIREBASE_OPTIONS_IOS_APP_ID
       
   - Fastfile (2 secrets)
     - FIREBASE_CLI_TOKEN
     - APP_ID

2. Create a GitHub Actions workflow file (.github\workflows\android_fastlane_firebase_app_distribution_workflow.yml):

   - ```yaml
      name: Android Fastlane with Firebase App Distribution Workflow
      
      on:
        push:
          branches:
            - mobile-app-stable # The branch that you would like to run the workflow when you push into it.
      
      jobs:
        distribute_to_firebase:
          runs-on: ubuntu-latest
          steps:
            - name: Checkout my repo code
              uses: actions/checkout@v2
      
            - name: Set up JDK 11
              uses: actions/setup-java@v2
              with:
                java-version: "11"
                distribution: "temurin"
      
            - name: Install Flutter
              uses: subosito/flutter-action@v2
              with:
                channel: stable
      
            # If there was an error, then run this: "cd android" then "bundle lock --add-platform x86_64-linux"
            - name: Setup Ruby
              uses: ruby/setup-ruby@v1
              with:
                ruby-version: "3.3.4"
                bundler-cache: true
                working-directory: android
      
            - name: Build and Distribute App
              env:
                APP_ID: ${{ secrets.APP_ID }}
                FIREBASE_CLI_TOKEN: ${{ secrets.FIREBASE_CLI_TOKEN }}
                FIREBASE_OPTIONS_ANDROID_API_KEY: ${{ secrets.FIREBASE_OPTIONS_ANDROID_API_KEY }}
                FIREBASE_OPTIONS_ANDROID_APP_ID: ${{ secrets.FIREBASE_OPTIONS_ANDROID_APP_ID }}
                FIREBASE_OPTIONS_IOS_API_KEY: ${{ secrets.FIREBASE_OPTIONS_IOS_API_KEY }}
                FIREBASE_OPTIONS_IOS_APP_ID: ${{ secrets.FIREBASE_OPTIONS_IOS_APP_ID }}
              run: |
                bundle exec fastlane android firebase_distribution
              working-directory: android
     ```

## Running the Pipeline

1. Push your code to GitHub.
2. GitHub Actions will trigger the workflow on the `mobile-app-stable` branch.
3. The workflow will build the APK and deploy it to Firebase App Distribution.
4. Testers will receive the APK via Firebase App Distribution.

Now, no more sending APKs via WhatsApp! 😂

## Conclusion

Setting up a CI/CD pipeline with Fastlane, GitHub Actions, and Firebase App Distribution can greatly streamline your mobile app development process. It ensures that your code is always in a deployable state, automates repetitive tasks, and enables rapid, consistent releases.

Happy Coding! 🚀

## References

- [Fastlane Documentation](https://docs.fastlane.tools/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
- [Firebase App Distribution Documentation](https://firebase.google.com/docs/app-distribution)
