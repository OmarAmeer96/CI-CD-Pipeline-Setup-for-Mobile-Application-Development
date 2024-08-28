# CI/CD Pipeline for Flutter Apps Development

![Screenshot 2024-07-23 161614](https://github.com/user-attachments/assets/dff88e6c-bd72-41d3-bfd8-b892505503ea)

## Overview

This guide outlines the process of setting up a CI/CD pipeline for a Flutter mobile application using Fastlane, GitHub Actions, and Firebase App Distribution. The pipeline automates building, testing, and distributing the app, streamlining development and ensuring rapid, consistent deployments.

## Table of Contents

- [CI/CD Pipeline for Flutter Apps Development](#cicd-pipeline-for-flutter-apps-development)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Setting Up Flavors in Flutter](#setting-up-flavors-in-flutter)
    - [`..\.vscode\launch.json` Configuration](#vscodelaunchjson-configuration)
    - [`..\android\app\build.gradle` Configuration](#androidappbuildgradle-configuration)
    - [`..\android\app\src\main\AndroidManifest.xml` Configuration](#androidappsrcmainandroidmanifestxml-configuration)
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
            "name": "APP_NAME Development",
            "request": "launch",
            "type": "dart",
            "program": "lib/main_development.dart",
            "args": ["--flavor", "Development", "--target", "lib/main_development.dart"]
        },
        {
            "name": "APP_NAME Production",
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
        resValue "string", "app_name", "APP_NAME Production"
    }
    development {
        dimension "default"
        applicationIdSuffix ".dev"
        resValue "string", "app_name", "APP_NAME Development"
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

2. Install Fastlane (in cmd):

   - ```sh
     gem install fastlane
     ```

3. Initialize Fastlane (in project root):

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

3. The `..\android\fastlane\Fastfile` file will be automatically created, update it with the following:

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

2. Create a GitHub Actions workflow file `..\.github\workflows\android_fastlane_firebase_app_distribution_workflow.yml`:

   ```yaml
   name: Android Fastlane App Distribution Workflow

   on:
     push:
       branches:
         - mobile-app-stable # The branch that you would like to run the workflow when you push into it.

   jobs:
     distribution_to_firebase:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout My Repository
           uses: actions/checkout@v2

         - name: Setup JDK 11
           uses: actions/setup-java@v2
           with:
             distribution: 'temurin'
             java-version: '11'

         - name: Setup Flutter
           uses: subosito/flutter-action@v2
           with:
             channel: 'stable'

         - name: Setup Ruby
           uses: ruby/setup-ruby@v1
           with:
             ruby-version: '3.3.4'
             bundler-cache: false # Disable bundler-cache to manage dependencies manually

         - name: Update Gemfile.lock platforms
           run: |
             cd android
             bundle lock --add-platform x86_64-linux
             bundle lock --add-platform ruby

         - name: Install Ruby Dependencies
           run: |
             cd android
             bundle install

         - name: Flutter Build Android App And Upload To Firebase
           env:
             FIREBASE_CLI_TOKEN: ${{ secrets.FIREBASE_CLI_TOKEN }}
           run: |
             cd android
             bundle exec fastlane android firebase_distribution
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
