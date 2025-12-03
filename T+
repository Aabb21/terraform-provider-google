name: Android Release Build & Firebase Distribution

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build_release:
    runs-on: ubuntu-latest
    env:
      ANDROID_SDK_ROOT: /usr/local/lib/android/sdk
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Install Node (if needed)
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install Android SDK packages (via sdkmanager)
        run: |
          sudo apt-get update
          sudo apt-get install -y unzip
          yes | sdkmanager --licenses || true
        shell: bash

      - name: Decode keystore
        if: secrets.ANDROID_KEYSTORE_BASE64 != ''
        env:
          KEY_B64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
        run: |
          echo "$KEY_B64" | base64 --decode > android/app/thone-release.jks

      - name: Set gradle properties for signing
        run: |
          cat > android/gradle.properties <<EOF
          MYAPP_RELEASE_STORE_FILE=thone-release.jks
          MYAPP_RELEASE_STORE_PASSWORD=${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          MYAPP_RELEASE_KEY_ALIAS=${{ secrets.ANDROID_KEY_ALIAS }}
          MYAPP_RELEASE_KEY_PASSWORD=${{ secrets.ANDROID_KEY_PASSWORD }}
          EOF

      - name: Build release APK
        run: |
          cd android
          ./gradlew assembleRelease --no-daemon -Pandroid.useAndroidX=true

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: thone-release-apk
          path: android/app/build/outputs/apk/release/app-release.apk

      - name: Firebase App Distribution (using token)
        if: secrets.FIREBASE_TOKEN != ''
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          token: ${{ secrets.FIREBASE_TOKEN }}
          groups: testers
          file: android/app/build/outputs/apk/release/app-release.apk

      - name: Firebase App Distribution (using service account)
        if: secrets.GCP_SA_KEY != ''
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          serviceCredentials: ${{ secrets.GCP_SA_KEY }}
          groups: testers
          file: android/app/build/outputs/apk/release/app-release.apk
