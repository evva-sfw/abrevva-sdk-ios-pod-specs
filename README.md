<p align="center">
  <h1 align="center">EVVA Abrevva iOS SDK</h1>
</p>

<p align="center">
  <a href="#quick-start"><img src="https://img.shields.io/badge/package-CocoaPods-4BC51D.svg" alt="Package managers"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-EVVALicense-blue.svg" alt="EVVA License"></a>
</p>

The EVVA Abrevva iOS SDK is a collection of tools to work with electronical EVVA access components. It allows for scanning and connecting via BLE.

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Examples](#examples)

## Features

- BLE Scanner for EVVA components in range
- Localize EVVA components encountered by a scan
- Disengage EVVA components encountered by a scan
- Read / Write data via BLE

## Requirements

| Platform  | Minimum Swift Version | Installation            | Status                   |
|-----------| --------------------- |-------------------------| ------------------------ |
| iOS 15.0+ | 5.7.1 / Xcode 14.1    | [CocoaPods](#cocoapods) | Fully Tested             |
| Android   | see [EVVA Abrevva Android SDK](https://github.com/evva-sfw/abrevva-sdk-android)

## Installation

### Cocoapods

[CocoaPods](https://cocoapods.org) is a dependency manager for Cocoa projects. For usage and installation instructions, visit their website. To integrate EVVA Abrevva iOS SDK into your Xcode project using CocoaPods, specify the source repo and pod in your `Podfile`:

```ruby
source 'https://github.com/evva-sfw/abrevva-sdk-ios-pod-specs.git'
pod 'AbrevvaSDK'
```

## Examples

### Initialize BleManager

To start off first initialize the SDK BleManager. You can pass an init callback closure for success indication.

```swift
import AbrevvaSDK

public class Example {

    private var bleManager: BleManager?
    private var bleDeviceMap = [String: BleDevice]()

    func initialize() {
        self.bleManager = BleManager { success, message in
            debugPrint("BleManager initialized /w success=\(success)")
        }
    }
}
```

### Scan for EVVA components

Use the BleManager to scan for components in range. You can pass several callback closures to react to the different events when scanning or connecting to components.

```swift
func requestLEScan() {
    let timeout = 10_000

    self.bleManager?.startScan(
        nil,        // name filter
        nil,        // name prefix filter
        false,      // allow duplicates
        { success in
            debugPrint("Scan started /w success=\(success)")
        }, 
        { device, advertisementData, rssi in
            debugPrint("Found device /w address=\(device.getAddress())")
            self.bleDeviceMap[device.getAddress()] = device
        },
        { address in
            debugPrint("Connected to device /w address=\(address)")
        },
        { address in
            debugPrint("Disconnected from device /w address=\(address)")
        },
        timeout
    )
}
```

### Localize EVVA component

With the signalize method you can localize EVVA components. On a successful signalization the component will emit a melody indicating its location.

```swift
 func signalize(deviceID: String) async {
    guard let device = self.bleDeviceMap[deviceID] else { return }

    let success = await self.bleManager?.signalize(device)
    debugPrint("Signalized /w success=\(success)")
}
```
### Perform disengage for EVVA components

For the component disengage you have to provide access credentials to the EVVA component. Those are generally acquired in the form of access media metadata from the Xesar software.

```swift
 func disengage(_ deviceID: String) async {
    guard let device = self.bleDeviceMap[deviceID] else { return }

    let mobileId = ""           // hex string
    let mobileDeviceKey = ""    // hex string
    let mobileGroupID = ""      // hex string
    let mobileAccessData = ""   // hex string
    let isPermanentRelease = false
    let timeout = 10_000

    let status = await self.bleManager?.disengage(
        device,
        mobileID,
        mobileDeviceKey,
        mobileGroupID,
        mobileAccessData,
        isPermanentRelease,
        timeout
    )
    debugPrint("Disengage /w status=\(status)")
}
```
There are several access status types upon attempting the component disengage.
```swift
public enum DisengageStatusType : String {
    case ERROR
    case AUTHORIZED
    case AUTHORIZED_PERMANENT_ENGAGE
    case AUTHORIZED_PERMANENT_DISENGAGE
    case AUTHORIZED_BATTERY_LOW
    case AUTHORIZED_OFFLINE
    case UNAUTHORIZED
    case UNAUTHORIZED_OFFLINE
    case SIGNAL_LOCALIZATION
    case MEDIUM_DEFECT_ONLINE
    case MEDIUM_BLACKLISTED
    case UNKNOWN_STATUS_CODE
    case UNABLE_TO_CONNECT
    case TIMEOUT
}
```
