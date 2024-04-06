---
title: Flutter
description: 
published: true
date: 2024-04-06T13:28:17.053Z
tags: 
editor: markdown
dateCreated: 2024-03-31T20:59:41.700Z
---

# Flutter

# Установка

## android

1. Установить переменную окружения [ANDROID_HOME](https://developer.android.com/tools/variables) с путём до SDK
```
export ANDROID_HOME="$HOME/.local/android"
export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin:$ANDROID_HOME/platform-tools"
```
2. Скачать commandline-tools https://developer.android.com/studio
3. Распаковать содержимое архива в `$ANDROID_HOME/cmdline-tools/latest/` https://developer.android.com/studio/command-line/sdkmanager/
4. Установить SDK
```
$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager --install "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```
5. Установить JAVA Runtime Environment
6. Проверить устновку https://github.com/flutter/flutter/issues/130515
```
flutter doctor --verbose
flutter analyze --suggestions
```
7. При проблемах проверить совместимость JRE с gradle https://docs.gradle.org/current/userguide/compatibility.html