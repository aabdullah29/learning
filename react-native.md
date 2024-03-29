
## create new project
`npx react-native init [projecr-name]`

`npm start` or `npx react-native start`
`npx run ios` or `react-native run-ios`
`npx run android` or `react-native run-android`



## for android emulator:
install android studio and create virtual devise and add enviorment path is:

`nano ~/.zprofile` or `nano ~/.zshrc`

```
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

## for ios simulator:
install ruby, Xcode and CocoaPods

if clone a repo then install CocoaPods:

```
npx pod-install
ruby --version
pod --version
bundle install
```

#### metro.config.js:
for metro configuration and metro is a live server which run the code in developement mode on devices.

#### babel.config.js:
use as compiler it's handle the react-native code for android and ios natively.

#### Gemfile and Gemfile.lock:
use for installing the ios or CocoaPods dependencies and its use ruby or gem repository.

#### Podfile and Podfile.lock
use for handle the CocoaPods or ios dependency it's work as packeg.json work in node.

#### for run specific simulator 
- add this into packeg.json
```
"ios-15": "react-native run-ios --simulator=`iPhone 15`",
```

## [adb-commands](https://gist.github.com/ernestkamara/d0a11cc36f331ebb8a41aa36991eee9a)
## [auto-link](https://github.com/react-native-community/cli/blob/main/docs/autolinking.md)

## android build
```
cd android
./gradlew assembleRelease
```

build path: `[project root]/android/app/build/outputs/apk/release/app-release.apk` 

