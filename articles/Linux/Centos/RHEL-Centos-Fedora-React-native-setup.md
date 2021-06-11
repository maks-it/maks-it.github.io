


Finally after several hours of investigation I think I have another solution for everyone having issues with AVD Manager "Unable to locate adb".

I know we have the setting for the SDK in File -> Settings -> Appearance & Behavior -> System Settings -> Android SDK. This it seems is not enough! It appears that Android Studio (at least the new version 4) does not give projects a default SDK, despite the above setting.

So, you also (for each project) need to go to File -> Project Structure -> Project Settings -> Project, and select the Project SDK, which is set to [No SDK] by default.

If there's nothing in the drop-down box, then select New, select Android SDK, and navigate to your Android SDK location (normally C:\Users[username]\AppData\Local\Android\Sdk on Windows). You will then be able to select the Android API xx Platform. You now should not get this annoying adb error.



npm react-native start

adb reverse tcp:8081 tcp:8081;adb reverse tcp:3000 tcp:3000;adb reverse tcp:9090 tcp:9090



npx react-native run-android
