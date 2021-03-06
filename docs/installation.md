# Installation

First, download the library from npm:

```
npm install react-native-maps --save
```


## Get a Google Maps API key

Go to https://developers.google.com/maps/documentation/ios-sdk/get-api-key and https://developers.google.com/maps/documentation/android-api/signup to get your keys for both iOS and Android.

Make sure that Google Maps Android API and Google Maps SDK for iOS are enabled for the current project.
https://console.developers.google.com/apis/library/maps-android-backend.googleapis.com/
https://console.developers.google.com/apis/library/maps-ios-backend.googleapis.com

Without an API key the Google Maps map won't render anything.


## iOS - CocoaPods

Setup your `Podfile` (found at `/ios/Podfile` as below, replace all references to `_YOUR_PROJECT_TARGET_` with your project target (it's the same as project name by default), and then run `pod install` while in the `ios` folder.

Please make sure to use `.xcworkspace`, not `.xcproject` after that.

~~~
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target '_YOUR_PROJECT_TARGET_' do
  rn_path = '../node_modules/react-native'
  rn_maps_path = '../node_modules/react-native-maps'

  pod 'yoga', path: "#{rn_path}/ReactCommon/yoga/yoga.podspec"
  pod 'React', path: rn_path, subspecs: [
    'Core',
    'RCTActionSheet',
    'RCTAnimation',
    'RCTGeolocation',
    'RCTImage',
    'RCTLinkingIOS',
    'RCTNetwork',
    'RCTSettings',
    'RCTText',
    'RCTVibration',
    'RCTWebSocket',
    'BatchedBridge', # REMOVE THIS if you use RN 0.54+
  ]

  pod 'react-native-maps', path: rn_maps_path

  pod 'GoogleMaps'  # Remove this line if you don't want to support Google Maps on iOS
  pod 'react-native-google-maps', path: rn_maps_path  # Remove this line if you don't want to support Google Maps on iOS
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    if target.name == 'react-native-google-maps'
      target.build_configurations.each do |config|
        config.build_settings['CLANG_ENABLE_MODULES'] = 'No'
      end
    end
    if target.name == "React"
      target.remove_from_project
    end
  end
end
~~~



## IMPORTANT!!

**!!  DO NOT USE  !!** `react-native link`

Have ran it already? Read [this](#on-ios).

## If you want to use Google maps


Add to `ios/_YOUR_PROJECT_NAME_/AppDelegate.m:

```
+ @import GoogleMaps; //add this line if you want to use Google Maps

@implementation AppDelegate
...

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
+  [GMSServices provideAPIKey:@"_YOUR_API_KEY_"]; // add this line using the api key obtained from Google Console
...
```

## Notes on running on a real ios device

The steps are as described in https://facebook.github.io/react-native/docs/running-on-device.html , however instead of opening the .xcodeproj file you should open .xcworkspace file.


## Android

1. In your `android/app/build.gradle` add:

   ```groovy
   ...
   dependencies {
     ...
     implementation project(':react-native-maps')
   }
   ```

If you've defined *[project-wide properties](https://developer.android.com/studio/build/gradle-tips.html)* (**recommended**) in your root `build.gradle`, this library will detect the presence of the following properties:

    ```groovy
    buildscript {...}
    allprojects {...}
    
    /**
     + Project-wide Gradle configuration properties
     */
    ext {
        compileSdkVersion   = 26
        targetSdkVersion    = 26
        buildToolsVersion   = "26.0.2"
        supportLibVersion   = "26.1.0"
        googlePlayServicesVersion = "11.8.0"
        androidMapsUtilsVersion = "0.5+"
    }
    ```

   If you do **not** have *project-wide properties* defined and have a different play-services version than the one included in this library, use the following instead (switch 10.0.1 for the desired version):

   ```groovy
   ...
   dependencies {
       ...
       implementation(project(':react-native-maps')){
           exclude group: 'com.google.android.gms', module: 'play-services-base'
           exclude group: 'com.google.android.gms', module: 'play-services-maps'
       }
       implementation 'com.google.android.gms:play-services-base:10.0.1'
       implementation 'com.google.android.gms:play-services-maps:10.0.1'
   }
   ```

2. In your `android/settings.gradle` add:

   ```groovy
   ...
   include ':react-native-maps'
   project(':react-native-maps').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-maps/lib/android')
   ```

3. Specify your Google Maps API Key:

   Add your API key to your manifest file (`android/app/src/main/AndroidManifest.xml`):

   ```xml
   <application>
       <!-- You will only need to add this meta-data tag, but make sure it's a child of application -->
       <meta-data
         android:name="com.google.android.geo.API_KEY"
         android:value="Your Google maps API Key Here"/>
   </application>
   ```
   > Note: As shown above, com.google.android.geo.API_KEY is the recommended metadata name for the API key. A key with this name can be used to authenticate to multiple Google Maps-based APIs on the Android platform, including the Google Maps Android API. For backwards compatibility, the API also supports the name com.google.android.maps.v2.API_KEY. This legacy name allows authentication to the Android Maps API v2 only. An application can specify only one of the API key metadata names. If both are specified, the API throws an exception.
Source: https://developers.google.com/maps/documentation/android-api/signup


4. Add `import com.airbnb.android.react.maps.MapsPackage;` and `new MapsPackage()` in your `MainApplication.java` :

   ```
   import com.airbnb.android.react.maps.MapsPackage;
   ...
   @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
                    new MapsPackage()
            );
        }
   ```


5. Ensure that you have Google Play Services installed:
  * For Genymotion you can follow [these instructions](https://www.genymotion.com/help/desktop/faq/#google-play-services).
  * For a physical device you need to search on Google for 'Google Play Services'. There will be a link that takes you to the Play Store and from there you will see a button to update it (do not search within the Play Store).

## Troubleshooting

If you have a blank map issue, ([#118](https://github.com/airbnb/react-native-maps/issues/118), [#176](https://github.com/airbnb/react-native-maps/issues/176), [#684](https://github.com/airbnb/react-native-maps/issues/684)), try the following lines :

### On iOS:

If you have ran 'react-native link` by mistake:

1. delete node_modules
2. delete ios/Pods
3. delete ios/Podfile.lock
4. open Xcode and delete `AIRMaps.xcodeproj` from Libraries if it exists
5. in Build Phases -> Link Binary With Libraries delete `libAIRMaps.a` if it exists
6. delete ios/build folder
7. start again with the installation steps

If you use Xcode with version less than 9 you may get `use of undeclared identifier 'MKMapTypeMutedStandard'` or `Entry, ":CFBundleIdentifier", Does Not Exist` errors. In this case you have to update your Xcode.

### On Android:

1. Be sure to have `new MapsPackage()` in your `MainApplication.java` :

   ```
   import com.airbnb.android.react.maps.MapsPackage;
   ...
   @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
                    new MapsPackage()
            );
        }
   ```

1. Set this Stylesheet in your map component
   ```
   import MapView from 'react-native-maps';
   ...
   const styles = StyleSheet.create({
     container: {
       ...StyleSheet.absoluteFillObject,
       height: 400,
       width: 400,
       justifyContent: 'flex-end',
       alignItems: 'center',
     },
     map: {
       ...StyleSheet.absoluteFillObject,
     },
   });

   export default class MyApp extends React.Component {
     render() {
       const { region } = this.props;
       console.log(region);

       return (
         <View style ={styles.container}>
           <MapView
             style={styles.map}
             region={{
               latitude: 37.78825,
               longitude: -122.4324,
               latitudeDelta: 0.015,
               longitudeDelta: 0.0121,
             }}
           >
           </MapView>
         </View>
       );
     }
   }
   ```
1. Run "android" and make sure all packages are up-to-date.
1.  If not installed yet, you have to install the following packages :
    - Extras / Google Play services
    - Extras / Google Repository
    - Android 6.0 (API 23) / Google APIs Intel x86 Atom System Image Rev. 19
    - Android SDK Build-tools 23.0.3
1. Go to [Google API Console](https://console.developers.google.com/flows/enableapi?apiid=maps_android_backend) and select your project, or create one.
Then, once enabled, select `Go to credentials`.
Select `Google Maps Android API` and create a new key.
Enter the name of the API key and create it.

1. Clean the cache :
  ```
   watchman watch-del-all
   npm cache clean
  ```

1. When starting emulator, make sure you have enabled `Wipe user data`.

1. Run `react-native run-android`

1. If you encounter `com.android.dex.DexException: Multiple dex files define Landroid/support/v7/appcompat/R$anim`, then clear build folder.
  ```
   cd android
   ./gradlew clean
   cd ..
  ```

1. If you are using Android Virtual Devices (AVD), ensure that `Use Host GPU` is checked in the settings for your virtual device.

1. If using an emulator and the only thing that shows up on the screen is the message: `[APPNAME] won't run without Google Play services which are not supported by your device.`, you need to change the emulator CPU/ABI setting to a system image that includes Google APIs.  These may need to be downloaded from the Android SDK Manager first.
