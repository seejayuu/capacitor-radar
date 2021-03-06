![Radar](https://github.com/radarlabs/capacitor-radar/raw/master/logo.png)

[![npm](https://img.shields.io/npm/v/capacitor-radar.svg)](https://www.npmjs.com/package/capacitor-radar)

[Radar](https://radar.io) is the location platform for mobile apps.

## Installation

Install the package from npm:

```bash
npm install --save capacitor-radar
```

Then, update dependencies:

```bash
npx cap sync
```

If successful, you will see:

```bash
✔ Updating Android plugins
  Found 1 Capacitor plugin for android:
    capacitor-radar
✔ update android
...
✔ Updating iOS plugins
  Found 1 Capacitor plugin for ios:
    capacitor-radar
✔ Updating iOS native dependencies with "pod install" (may take several minutes)
✔ update ios
```

On iOS, complete the steps in [Configure project](https://radar.io/documentation/sdk#ios-configure), then add the SDK to your project using CocoaPods. Initialize the SDK in `application:didFinishLaunchingWithOptions:` in  your `AppDelegate`, passing in your Radar publishable API key:

```swift
import Capacitor
import RadarSDK

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    Radar.initialize(publishableKey: PUBLISHABLE_KEY)

    return true
  }

}
```

On Android, complete the steps in [Configure project](https://radar.io/documentation/sdk#android-configure), then add the SDK to your project using Gradle. Initialize the SDK and the plugin in your `MainActivity`, passing in your Radar publishable API key:

```java
import io.radar.sdk.Radar;
import io.radar.capacitor.RadarPlugin;

public class MainActivity extends BridgeActivity {

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    Radar.initialize(PUBLISHABLE_KEY);

    this.init(savedInstanceState, new ArrayList<Class<? extends Plugin>>() {{
      add(RadarPlugin.class);
    }});
  }

}
```

To get a Radar publishable API key, [sign up for a Radar account](https://radar.io).

## Usage

### Import module

First, import the module:

```javascript
import { Plugins } from '@capacitor/core';
import 'capacitor-radar';

const { RadarPlugin } = Plugins;
```

### Initialize SDK

Initialize the SDK again in JavaScript:

```javascript
RadarPlugin.initialize({ publishableKey });
```

where `publishableKey` is a string containing your publishable API key.

### Identify user

Until you identify the user, Radar will automatically identify the user by device ID.

To identify the user when logged in, call:

```javascript
RadarPlugin.setUserId({ userId });
```

where `userId` is a stable unique ID string for the user.

Do not send any PII, like names, email addresses, or publicly available IDs, for `userId`. See [privacy best practices](https://help.radar.io/privacy/what-are-privacy-best-practices-for-radar) for more information.

To set an optional dictionary of custom metadata for the user, call:

```javascript
RadarPlugin.setMetadata({ metadata });
```

where `metadata` is a JSON object with up to 16 keys and values of type string, boolean, or number.

Finally, to set an optional description for the user, displayed in the dashboard, call:

```javascript
RadarPlugin.setDescription({ description });
```

where `description` is a string.

You only need to call these functions once, as these settings will be persisted across app sessions.

### Request permissions

Before tracking the user's location, the user must have granted location permissions for the app.

To determine the whether user has granted location permissions for the app, call:

```javascript
RadarPlugin.getLocationPermissionsStatus().then((result) => {
  // do something with result.status
});
```

`result.status` will be a string, one of:

- `GRANTED`
- `DENIED`
- `UNKNOWN`

To request location permissions for the app, call:

```javascript
RadarPlugin.requestLocationPermissions({ background });
```

where `background` is a boolean indicating whether to request background location permissions or foreground location permissions.

On Android and web, `background` will be ignored.

### Foreground tracking

Once you have initialized the SDK, you have identified the user, and the user has granted permissions, you can track the user's location.

To track the user's location in the foreground, call:

```javascript
RadarPlugin.trackOnce().then((result) => {
  // do something with result.location, result.events, result.user.geofences
}).catch((err) => {
  // optionally, do something with err
});
```

`err` will be a string, one of:

- `ERROR_PUBLISHABLE_KEY`: the SDK was not initialized
- `ERROR_PERMISSIONS`: the user has not granted location permissions for the app
- `ERROR_LOCATION`: location services were unavailable, or the location request timed out
- `ERROR_NETWORK`: the network was unavailable, or the network connection timed out
- `ERROR_UNAUTHORIZED`: the publishable API key is invalid
- `ERROR_SERVER`: an internal server error occurred
- `ERROR_UNKNOWN`: an unknown error occurred

### Background tracking (iOS and Android only)

On iOS and Android, once you have initialized the SDK, you have identified the user, and the user has granted permissions, you can start tracking the user's location in the background.

To start tracking the user's location in the background, call:

```javascript
RadarPlugin.startTracking();
```

Assuming you have configured your project properly, the SDK will wake up while the user is moving (usually every 3-5 minutes), then shut down when the user stops (usually within 5-10 minutes). To save battery, the SDK will not wake up when stopped, and the user must move at least 100 meters from a stop (sometimes more) to wake up the SDK. **Note that location updates may be delayed significantly by iOS [Low Power Mode](https://support.apple.com/en-us/HT205234), by Android [Doze Mode and App Standby](https://developer.android.com/training/monitoring-device-state/doze-standby.html) and [Background Location Limits](https://developer.android.com/about/versions/oreo/background-location-limits.html), or if the device has connectivity issues, low battery, or wi-fi disabled. These constraints apply to all uses of background location services on iOS and Android, not just Radar. See more about [accuracy and reliability](https://radar.io/documentation/sdk#accuracy).**

Optionally, you can configure advanced tracking options. See the [iOS background tracking documentation](https://radar.io/documentation/sdk#ios-background) and [Android background tracking documentation](https://radar.io/documentation/sdk#android-background) for descriptions of these options.

```javascript
RadarPlugin.startTracking({
  priority: 'responsiveness', // // use 'efficiency' to avoid Android vitals bad behavior thresholds (ignored on iOS)
  sync: 'possibleStateChanges', // use 'all' to sync all location updates ('possibleStateChanges' recommended)
  offline: 'replayStopped' // use 'replayOff' to disable offline replay ('replayStopped' recommended)
});
```

To stop tracking the user's location in the background (e.g., when the user logs out), call:

```javascript
RadarPlugin.stopTracking();
```

You only need to call these methods once, as these settings will be persisted across app sessions.

On web, background tracking is not supported and these calls will be ignored.

### Manual tracking

You can manually update the user's location by calling:

```javascript
const latitude = 39.2904;
const longitude = -76.6122;
const accuracy = 65; // meters

Radar.updateLocation({ latitude, longitude, accuracy }).then((result) => {
  // do something with result.events, result.user.geofences
}).catch((err) => {
  // optionally, do something with err
});
```
