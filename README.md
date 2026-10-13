
name: IPTV Android and Server CI

on:
  push:
    branches: [main, master]
  pull_request:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  android-build:
    name: Build Android APK
    runs-on: ubuntu-latest
    timeout-minutes: 35

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Check Gradle project
        run: gradle --no-daemon help

      - name: Build debug APK
        run: gradle --no-daemon clean assembleDebug

      - name: Run Android lint
        run: gradle --no-daemon lint
        continue-on-error: true

      - name: Build release APK
        run: gradle --no-daemon assembleRelease

      - name: Upload APKs and reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: iptv-android-build
          if-no-files-found: warn
          path: |
            app/build/outputs/apk/debug/*.apk
            app/build/outputs/apk/release/*.apk
            app/build/reports/lint-results*.html
            app/build/reports/lint-results*.xml
          retention-days: 14

  server-check:
    name: Check Node.js server
    runs-on: ubuntu-latest
    timeout-minutes: 15

    defaults:
      run:
        working-directory: server

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm install --no-audit --no-fund

      - name: Check JavaScript syntax
        run: node --check src/server.js

      - name: Run automated tests
        run: npm test
