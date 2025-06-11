🍏 Fixes for IOS

I have uploaded `ffmpeg-kit-full-gpl-6.0-ios-xcframework.zip` to my GitHub repo:

https://github.com/dipakatkari/ffmpeg-kit-ios-full-gpl/archive/refs/tags/latest.zip

**Step 1:**
Create `ffmpeg-kit-ios-full-gpl.podspec` file in project's `ios` folder and add the following code in it.

```
Pod::Spec.new do |s|
    s.name             = 'ffmpeg-kit-ios-full-gpl'
    s.version          = '6.0'   # Must match what ffmpeg-kit-react-native expects.
    s.summary          = 'Custom full-gpl FFmpegKit iOS frameworks from dipakatkari.'
    s.homepage         = 'https://github.com/dipakatkari/ffmpeg-kit-ios-full-gpl'
    s.license          = { :type => 'LGPL' }
    s.author           = { 'dipakatkari' => 'https://github.com/dipakatkari' }
    s.platform         = :ios, '12.1'
    s.static_framework = true
  
    # Use the HTTP source to fetch the zipped package directly.
    s.source           = { :http => 'https://github.com/dipakatkari/ffmpeg-kit-ios-full-gpl/archive/refs/tags/latest.zip' }
  
    # Because the frameworks are inside the extracted archive under:
    # ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/
    # we list each of the needed frameworks with the full relative path.
    s.vendored_frameworks = [
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/libswscale.xcframework',
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/libswresample.xcframework',
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/libavutil.xcframework',
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/libavformat.xcframework',
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/libavfilter.xcframework',
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/libavdevice.xcframework',
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/libavcodec.xcframework',
      'ffmpeg-kit-ios-full-gpl-latest/ffmpeg-kit-ios-full-gpl/6.0-80adc/ffmpegkit.xcframework'
    ]
  end
```

---

**Step 2:**
Modify `Podfile`

```
/* Add this line above */
pod 'ffmpeg-kit-ios-full-gpl', :podspec => './ffmpeg-kit-ios-full-gpl.podspec'

/* Already exists in the pod file -- Don't change anything */
pod 'ffmpeg-kit-react-native', :subspecs => ['full-gpl'], :podspec => '../node_modules/ffmpeg-kit-react-native/ffmpeg-kit-react-native.podspec'
```

---

**Step 3:**
Run the following commands

```
# Remove Pods and Podfile.lock
rm -rf ios/Pods
rm -f ios/Podfile.lock

# Remove node_modules
rm -rf node_modules

# Optional: Remove derived data (Xcode cache)
rm -rf ~/Library/Developer/Xcode/DerivedData

# Reinstall everything
npm install or yarn install
cd ios
pod install
cd ..
```

Run the project for IOS and the pod structure will look like below

![1_wU4RN1O_BK_e4JK3wCYQrw](https://github.com/user-attachments/assets/5d4cb49e-75dc-4cfe-998d-aa5b9e69c105)

---

🤖 Fixes for Android

I have uploaded `ffmpeg-kit-full-gpl.aar` to my GitHub repo:

https://github.com/dipakatkari/ffmpeg-kit-full-gpl/releases/download/v1.0.0/ffmpeg-kit-full-gpl.aar

**Step 1:**
Modify `android/app/build.gradle`

```
import java.net.URL // Add this line on top of the file

android {
    .......

    // Add this code for accessing the file locally after downloading
    repositories {
        flatDir {
            dirs "$rootDir/libs"
        }
    }
}

dependencies {

    // Add the following dependencies
    implementation(name: 'ffmpeg-kit-full-gpl', ext: 'aar')
    implementation 'com.arthenica:smart-exception-java:0.2.1'

    ........
}

// Add the following script to download the file from the cloud
afterEvaluate {
    def aarUrl = 'https://github.com/dipakatkari/ffmpeg-kit-full-gpl/releases/download/v1.0.0/ffmpeg-kit-full-gpl.aar'
    def aarFile = file("${rootDir}/libs/ffmpeg-kit-full-gpl.aar")

    tasks.register("downloadAar") {
        doLast {
             if (!aarFile.parentFile.exists()) {
                println "📁 Creating directory: ${aarFile.parentFile.absolutePath}"
                aarFile.parentFile.mkdirs()
            }
            if (!aarFile.exists()) {
                println "⏬ Downloading AAR from $aarUrl..."
                new URL(aarUrl).withInputStream { i ->
                    aarFile.withOutputStream { it << i }
                }
                println "✅ AAR downloaded to ${aarFile.absolutePath}"
            } else {
                println "ℹ️ AAR already exists at ${aarFile.absolutePath}"
            }
        }
    }

    // Make sure the AAR is downloaded before compilation begins
    preBuild.dependsOn("downloadAar")
}
```

---

**Step 2:**
Modify `android/build.gradle`

```
buildscript {
    ext {
        .....
        /* Remove the following line */
        ffmpegKitPackage = "full-gpl"
    }
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        ....

    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
        flatDir {
            dirs "$rootDir/libs"
        }
    }
}

apply plugin: "com.facebook.react.rootproject"
```

---

**Step 3:**
Modify `package.json` and add the following script to download the file from the cloud after executing **yarn install** or **npn install**

```
"scripts": {
    "postinstall": "cd android && ./gradlew :app:downloadAar"
  }
```

---

**Step 4:**
Modify `ffmpeg-kit-react-native` package’s `android/build.gradle` then update the dependency and make a patch

```
android {
  ....

  defaultConfig {
    // minSdkVersion safeExtGet('ffmpegKitPackage', 'https').contains("-lts") ? 16 : 24
    minSdkVersion 24
    ....
  }
}

dependencies {
  ....

  /* Remove the following line */
  implementation 'com.arthenica:ffmpeg-kit-' + safePackageName(safeExtGet('ffmpegKitPackage', 'https')) + ':' + safePackageVersion(safeExtGet('ffmpegKitPackage', 'https'))
  
  /* Add this line */
  implementation(name: 'ffmpeg-kit-full-gpl', ext: 'aar')
}
```

Try to run the project for Android, after executing **yarn install** or **npn install** It will download the **“.aar”** file from the cloud and add to the following path
`“android/libs/ffmpeg-kit-full-gpl.aar”`

✍️ Verify the Build

After applying these changes:

✅ iOS: Should compile without CocoaPods errors.

✅ Android: Should fetch .aar files successfully.
