---
author: elamalani
ms.author: emalani
ms.date: 11/25/2019
ms.topic: include
---

## Session and device information

Once you add App Center Analytics to your app and start the SDK, it will automatically track sessions and device properties like OS Version, model, etc. without any additional code.

The SDK automatically reports a user's country code if the device has a mobile data modem and a SIM card installed. WiFi-only devices will not report a country code by default. To set the country code of those users, you must retrieve your user's location yourself and use the `setCountryCode:` method in the SDK. Our advice is to be mindful about tracking your users location and use a low location resolution. The sample below uses `kCLLocationAccuracyKilometer`.

* Make sure that you [enable Location Services](https://support.apple.com/en-us/HT204690) on the device.
* Get the device's current location using `CLLocationManager`.
* Convert the location into an ISO country code using `CLGeocoder`.
* Override the carrier country code using the SDK's `setCountryCode` method.

Use the following code to get the device's location and override the carrier country code in the app:

Add the CLLocationManagerDelegate protocol to the AppDelegate and add the locationManager property:

```objc
@interface AppDelegate () <CLLocationManagerDelegate>
@property(nonatomic) CLLocationManager *locationManager;
@end
```
```swift
class AppDelegate: CLLocationManagerDelegate {
  private var locationManager: CLLocationManager = CLLocationManager()
}
```

In the didFinishLaunchingWithOptions: method set up the location manager:

```objc
  self.locationManager = [[CLLocationManager alloc] init];
  self.locationManager.delegate = self;
  self.locationManager.desiredAccuracy = kCLLocationAccuracyKilometer;
  [self.locationManager requestWhenInUseAuthorization];
```
```swift
  self.locationManager.delegate = self
  self.locationManager.desiredAccuracy = kCLLocationAccuracyKilometer
  self.locationManager.requestWhenInUseAuthorization()
```

> [!NOTE]
> The `requestWhenInUseAuthorization` method is unavailable for macOS. Remove calls to that method when developing for macOS.

Add the delegate methods:

```objc
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status {
  if (status == kCLAuthorizationStatusAuthorizedWhenInUse) {
    [manager requestLocation];
  }
}

- (void)locationManger:(CLLocationManager *)manager didUpdateLocations:(NSArray<CLLocation *> *)locations {
  CLLocation *location = [locations lastObject];
  CLGeocoder *geocoder = [[CLGeocoder alloc] init];
  [geocoder reverseGeocodeLocation:location
                 completionHandler:^(NSArray *placemarks, NSError *error) {
                   if (placemarks.count == 0 || error)
                     return;
                   CLPlacemark *pm = [placemarks firstObject];
                   [MSAppCenter setCountryCode:pm.ISOcountryCode];
                 }]
}

- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error {
  NSLog(@"Failed to find user's location: \(error.localizedDescription)");
}
```
```swift
func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
  if (status == kCLAuthorizationStatusAuthorizedWhenInUse) {
    manager.requestLocation()
  }
}

func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
  let userLocation:CLLocation = locations[0] as CLLocation
  CLGeocoder().reverseGeocodeLocation(userLocation) { (placemarks, error) in
    if error == nil {
      MSAppCenter.setCountryCode(placemarks?.first?.isoCountryCode)
    }
  }
}
  
func locationManager(_ Manager: CLLocationManager, didFailWithError error: Error) {
  print("Failed to find user's location: \(error.localizedDescription)")
}
```

## Custom events

You can track your own custom events with **up to twenty properties** to know what's happening in your app, understand user actions, and see the aggregates in the App Center portal.

Once you have started the SDK, use the `trackEvent:withProperties` method to track your events with properties. You can send **up to 200 distinct event names**. Also, there is a maximum limit of 256 characters per event name and 125 characters per event property name and event property value.

```objc
NSDictionary *properties = @{@"Category" : @"Music", @"FileName" : @"favorite.avi"};
[MSAnalytics trackEvent:@"Video clicked" withProperties: properties];
```
```swift
MSAnalytics.trackEvent("Video clicked", withProperties: ["Category" : "Music", "FileName" : "favorite.avi"])
```

Properties for events are entirely optional – if you just want to track an event, use this sample instead:

```objc
[MSAnalytics trackEvent:@"Video clicked"];
```
```swift
MSAnalytics.trackEvent("Video clicked")
```

## Event priority and persistence

You can track business critical events that have higher importance than other events.

* Developers can set priority of events as **Normal** (`MSFlagsNormal` in the API) or **Critical** (`MSFlagsCritical` in the API).
* Events with priority set as **Critical** will be retrieved from storage first and sent before **Normal** events.
* When the local storage is full and new events needs to be stored, the oldest events that have the lowest priority are deleted first to make room for the new ones.
* If the storage is full of logs with **Critical** priority, then tracking an event with
**Normal** priority will fail as the SDK cannot make room in that case.
* If you also use the **Crashes** service, please note that crash logs are set as **Critical** and share the same storage as events.
* The transmission interval is only applied to **Normal** events, **Critical** events will be sent after 3 seconds.

You can use the following API to track an event as **Critical**:

```objc
NSDictionary *properties = @{@"Category" : @"Music", @"FileName" : @"favorite.avi"};
[MSAnalytics trackEvent:@"Video clicked" withProperties:properties flags:MSFlagsCritical];

// If you are using name only, you can pass nil as properties.
```
```swift
let properties = ["Category" : "Music", "FileName" : "favorite.avi"];
MSAnalytics.trackEvent("Video clicked", withProperties: properties, flags: .critical)

// If you are using name only, you can pass nil as properties.
```

## Pause and resume sending logs

Pausing the event transmission can be useful in scenarios when the app needs to control the network bandwidth for more business critical needs. You can pause sending logs to the App Center backend. When paused, events can still be tracked and saved, but they are not sent right away. Any events your app tracks while paused will only be sent once you call `resume`.

```objc
[MSAnalytics pause];
[MSAnalytics resume];
```
```swift
MSAnalytics.pause()
MSAnalytics.resume()
```

## Enable or disable App Center Analytics at runtime

You can enable and disable App Center Analytics at runtime. If you disable it, the SDK will not collect any more analytics information for the app.

```objc
[MSAnalytics setEnabled:NO];
```
```swift
MSAnalytics.setEnabled(false)
```

To enable App Center Analytics again, use the same API but pass `YES`/`true` as a parameter.

```objc
[MSAnalytics setEnabled:YES];
```
```swift
MSAnalytics.setEnabled(true)
```

The state is persisted in the device's storage across application launches.

> [!NOTE]
> This method must only be used after `MSAnalytics` has been started.

## Check if App Center Analytics is enabled

You can also check if App Center Analytics is enabled or not.

```objc
[MSAnalytics isEnabled];
```
```swift
MSAnalytics.isEnabled()
```

> [!NOTE]
> This method must only be used after `MSAnalytics` has been started, it will always return `NO` or `false` before start.

## Local storage size

By default, the SDK stores all logs up to 10MB. Developers can use an API to increase the [storage size](../../other-apis/ios.md#storage-size) and the SDK will keep storing logs until the storage is full.

## No internet access

When there is no network connectivity, the SDK saves up to 10MB of logs in the local storage. Once the storage is full, the SDK starts discarding old logs to make room for the new logs. Once network connectivity returns, the SDK sends logs in the batch of 50 or after every 3 seconds (by default).

## Batching event logs

The App Center SDK uploads logs in a batch of 50 and if the SDK doesn't have 50 logs to send, it will still send logs after 3 seconds (by default). There can be a maximum of 3 batches sent in parallel.
The transmission interval can be changed:

```objc
// Change transmission interval to 10 minutes.
[MSAnalytics setTransmissionInterval:600];
```
```swift
// Change transmission interval to 10 minutes.
MSAnalytics.setTransmissionInterval(600)
```

The transmission interval value must be between 3 seconds and 86400 seconds (1 day) and this method must be called before the service is started.

## Retry and back-off logic

App Center SDK supports back-off retries on recoverable network errors. Below is the retry logic:

* 3 tries maximum per request.
* Each request has its own retry state machine.
* All the transmission channels are disabled (until next app process) after 1 request exhausts all its retries.

Back-off logic

* 50% randomization, 1st retry between 5 and 10s, second retry between 2.5 and 5 minutes, last try between 10 and 20 minutes.
* If network switches from off to on (or from wi-fi to mobile), retry states are reset and requests are retried immediately.
