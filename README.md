# OpenARMaps VPS SDK — iOS

Visual Positioning System SDK for native iOS apps. Integrates with ARKit to deliver centimeter-level 6DoF localization via the OpenARMaps spatial network.

## Features

- **5 cm accuracy** — centimeter-level 6DoF pose estimation
- **Auto-search** — no location ID required; VPS searches the entire network automatically
- **ARKit integration** — works with both SceneKit and RealityKit
- **UIKit & SwiftUI** — full support for both UI frameworks
- **Custom initial pose** — set a known position before the first localization request

## Requirements

- iOS 12.0+
- Xcode 12+
- Swift 5+
- ARKit-supported device

## Installation

### XCFramework

Download the latest `OAMVPS.xcframework` from [Releases](https://github.com/OpenARMaps/openarmaps-vps-ios/releases) and drag it into your Xcode project.

### Swift Package Manager

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/OpenARMaps/openarmaps-vps-ios.git", from: "1.0.0")
]
```

## Setup

Add to `Info.plist`:

```xml
<key>NSCameraUsageDescription</key>
<string>Camera is required for AR positioning.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location improves AR anchor accuracy.</string>
```

## Quick Start

### UIKit + SceneKit

```swift
import OAMVPS
import ARKit

class ARViewController: UIViewController, ARSCNViewDelegate {
    var arView: ARSCNView!
    var vps: VPSService?

    override func viewDidLoad() {
        super.viewDidLoad()
        arView.delegate = self

        VPSBuilder.initializeVPS(
            arSession:   arView.session,
            apiKey:      "oam_live_YOUR_KEY",
            locationIds: [],           // empty = auto-search
            url:         "https://api.openarmaps.org/api/v1",
            gpsUsage:    false,
            delegate:    self
        ) { self.vps = $0; self.vps?.start() }

        arView.session.run(VPSBuilder.getDefaultConfiguration()!)
    }

    func renderer(_ renderer: SCNSceneRenderer, updateAtTime time: TimeInterval) {
        vps?.frameUpdated()
    }
}

extension ARViewController: VPSServiceDelegate {
    func positionVPS(pos: ResponseVPSPhoto) {
        print("Localized:", pos)
        // pos contains the 6DoF pose — place AR content here
    }
    func error(err: NSError)                         { print("VPS error:", err) }
    func sending(requestData: UploadVPSPhoto? = nil) { }
}
```

### RealityKit

Replace the SceneKit renderer callback with `ARSessionDelegate`:

```swift
func session(_ session: ARSession, didUpdate frame: ARFrame) {
    vps?.frameUpdated()
}
```

### SwiftUI

```swift
import SwiftUI
import ARKit
import OAMVPS

struct ContentView: View {
    @StateObject var vm = VPSViewModel()

    var body: some View {
        ARViewRepresentable(vm: vm)
            .ignoresSafeArea()
            .overlay(alignment: .bottom) {
                Button(vm.isRunning ? "Stop VPS" : "Start VPS") {
                    vm.isRunning ? vm.vps?.stop() : vm.vps?.start()
                    vm.isRunning.toggle()
                }
                .padding()
            }
    }
}

class VPSViewModel: NSObject, ObservableObject, VPSServiceDelegate {
    @Published var isRunning = false
    var vps: VPSService?

    func initialize(session: ARSession) {
        VPSBuilder.initializeVPS(
            arSession:   session,
            apiKey:      "oam_live_YOUR_KEY",
            locationIds: [],
            url:         "https://api.openarmaps.org/api/v1",
            gpsUsage:    false,
            delegate:    self
        ) { self.vps = $0 }
    }

    func positionVPS(pos: ResponseVPSPhoto)           { print("Localized:", pos) }
    func error(err: NSError)                          { print("Error:", err) }
    func sending(requestData: UploadVPSPhoto? = nil)  { }
}
```

## Advanced

### Custom Initial Pose

Useful when you know where the user is before the first localization:

```swift
vps?.setCustomLocPosForFirstRequest(
    x: 10.0, y: 0.0, z: 5.0,
    roll: 0.0, pitch: 0.0, yaw: 90.0
)
vps?.start()
vps?.clearCustomLocPos() // clear when no longer needed
```

### Auto-search vs Specific Location

```swift
// Auto-search — no location ID needed, searches entire network
VPSBuilder.initializeVPS(locationIds: [], ...)

// Specific location — faster when you know where the user is
VPSBuilder.initializeVPS(locationIds: ["loc_abc123"], ...)
```

## Get an API Key

1. Sign up at [space.web-ar.studio](https://space.web-ar.studio)
2. Create a project → copy your `oam_live_...` key
3. Free tier: 1,000 localizations/month, unlimited public maps

## Contributing

See [CONTRIBUTING.md](https://github.com/OpenARMaps/openarmaps/blob/main/CONTRIBUTING.md).

## License

[MIT License](LICENSE)
