# VisualnoteIosBle

## Requirements

info.plist
NSBluetoothAlwaysUsageDescription 

## Installation

VisualnoteIosBle is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'VisualnoteIosBle', '0.1.4', :source => "https://github.com/Visual-Note/visualnote-ios-ble-specs.git"
```

## API

First of all import the library
```swift
import VisualnoteIosBle
import CoreBluetooth
```

then you must implement the protocol, for example in a viewcontroller

```swift
@objc public protocol VisualnoteProtocol {
    func didInitError(error: String)
    func didStartScan()
    func didConnect()
    func didDisconnect()
    func didSessionStarted()
    func didSessionTerminated()
    func didBLEStateChange(state: CBManagerState)
    func deviceDiscovered(device: String)
    func didScanTimeout()
}
```

to connect and send data, you must in sequence call:
```swift
visualnote.initBle(token: "MY-TOKEN")  // initialize BLE and ask user for permmission
visualnote.scan()                      // scan of available devices in the surrounding area
visualnote.connect(device: nextDevice) // connect to the chosen device
```

those calls will have a callback each
```swift
    // BLE initialized
    public func didBLEStateChange(state: CBManagerState) {
        var message = ""
    
        switch state {
        case  .poweredOff:
            message = "Bluetooth on this device is currently powered off"
        case .resetting:
            message = "BLE manager is resetting, please try again in a moment"
        case .unauthorized:
            message = "Bluetooth usage is not authorized for this app"
        case .unknown:
            message = "The BLE manager state is unknown"
        case .unsupported:
            message = "This device does not support Bluetooth Low Energy"
        case .poweredOn:
            message = "BLE is turned on and ready for communication"
        }
    
        os_log(.info,"ViewController::didBLEStateChange %s", message)
    }

    // init BLE Error 
    public func didInitError(error: String) {
        os_log(.info,"ViewController::didInitError %s", error)
    }

    // scan started
    public func didStartScan() {
        os_log(.info,"ViewController::didStartScan")
    }

    // connected
    public func didConnect() {
        os_log(.info,"ViewController::didConnect")
    }
    
    // scan ended
    public func didScanTimeout() {
        os_log(.info,"ViewController::didScanTimeout")
    }

    // device discovred: this one is sent for every device discovered in the surrounding area
    public func deviceDiscovered(device: String) {
        os_log(.info,"ViewController::deviceDiscovered %s", device)
        deviceToConnect = device
    }
```

after connection you can start a session and send data
```swift
visualnote.startSession()

public func didSessionStarted() {
    os_log(.info,"ViewController::didSessionStarted")
}

// turn on two led lights, on 1st fret 6th string, index finger, and 2nd fret 5th string, thumb finger
visualnote.sendManyLeds(fretNotes: [(fret: 0, string: 6, finger: .index), (fret: 2, string: 5, finger: .thumb)])

// these are the available
// finger:colors
public enum Finger : Int {
    case thumb = 0
    case index = 1
    case middle = 2
    case ring = 3
    case pinkie = 4
    case open = 5
    case defaultColor = 6
}

```

when you're done playing a song you can close the session, so to minimize battery usage
```swift
visualnote.stopSession()
```

## Full Example
```swift
import VisualnoteIosBle

class ViewController: UIViewController, VisualnoteProtocol {
private let visualnote = VisualnoteBle.sharedInstance
private var deviceToConnect: String?
internal var timer:Timer = Timer();

    override func viewDidLoad() {
        super.viewDidLoad()
        visualnote.delegate = self
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    @IBAction func onClickInitBLE(_ sender: UIButton) {
        visualnote.initBle(token: "MY-TOKEN")
    }

    @IBAction func onClickScan(_ sender: UIButton) {
        visualnote.scan()
    }

    @IBAction func onClickConnect(_ sender: UIButton) {
        guard let nextDevice = deviceToConnect else {
            return
        }
        visualnote.connect(device: nextDevice)
    }

    @IBAction func onClickStartSession(_ sender: UIButton) {
        visualnote.startSession()
    }
    
    @IBAction func onClickSendData(_ sender: UIButton) {
        // visualnote.sendManyLeds(fretNotes: [(fret: 0, string:1, finger: .index), (fret: 1, string:1, finger: .index)])
        // visualnote.sendSingleLed(fretNote: (fret: 2, string:6, finger: .index))
        // visualnote.sendManyLeds(fretNotes: [(fret: 0, string:6, finger: .index), (fret: 21, string:6, finger: .index)])
        timer = Timer.scheduledTimer(withTimeInterval: 0.050, repeats: true, block: { (timer) -> Void in self.sendNotes() })
    }

    private var loopInterval:Int = 0
    public func sendNotes() {
        os_log(.info,"ViewController::sendNotes RANDOM")
        // visualnote.sendManyLeds(fretNotes: [(fret: loopInterval, string: 1, finger: .index), (fret: 2, string: 5, finger: .index)])
        visualnote.sendManyLeds(fretNotes: [(fret: loopInterval, string: 1, finger: .index)])
        loopInterval += 1
        if loopInterval > 21 {
            loopInterval = 0
        }
    }

    @IBAction func onClickEndSession(_ sender: UIButton) {
        timer.invalidate()
        visualnote.stopSession()
    }

    @IBAction func onClickDisconnect(_ sender: UIButton) {
        visualnote.disconnect()
    }

    @IBAction func onClickState(_ sender: Any) {
        var message = ""

        switch visualnote.state {
        case  .poweredOff:
            message = "Bluetooth on this device is currently powered off"
        case .resetting:
            message = "BLE manager is resetting, please try again in a moment"
        case .unauthorized:
            message = "Bluetooth usage is not authorized for this app"
        case .unknown:
            message = "The BLE manager state is unknown"
        case .unsupported:
            message = "This device does not support Bluetooth Low Energy"
        case .poweredOn:
            message = "BLE is turned on and ready for communication"
        }

        os_log(.info,"ViewController::didBLEStateChange %s", message)
    }

    public func didConnect() {
        os_log(.info,"ViewController::didConnect")
    }

    public func didDisconnect() {
        os_log(.info,"ViewController::didDisconnect")
    }

    public func didSessionStarted() {
        os_log(.info,"ViewController::didSessionStarted")
    }

    public func didSessionTerminated() {
        os_log(.info,"ViewController::didSessionTerminated")
        timer.invalidate()
    }

    public func didStartScan() {
        os_log(.info,"ViewController::didStartScan")
    }

    public func didInitError(error: String) {
        os_log(.info,"ViewController::didInitError %s", error)
    }

    public func didScanTimeout() {
        os_log(.info,"ViewController::didScanTimeout")
    }

    public func deviceDiscovered(device: String) {
        os_log(.info,"ViewController::deviceDiscovered %s", device)
        deviceToConnect = device
    }

    public func didBLEStateChange(state: CBManagerState) {
        var message = ""

        switch state {
        case  .poweredOff:
            message = "Bluetooth on this device is currently powered off"
        case .resetting:
            message = "BLE manager is resetting, please try again in a moment"
        case .unauthorized:
            message = "Bluetooth usage is not authorized for this app"
        case .unknown:
            message = "The BLE manager state is unknown"
        case .unsupported:
            message = "This device does not support Bluetooth Low Energy"
        case .poweredOn:
            message = "BLE is turned on and ready for communication"
        }

        os_log(.info,"ViewController::didBLEStateChange %s", message)
    }
}
```

## Author

Visual Note SRL, giona.granata@gmail.com

## License

Copyright (c) 2021 Visual Note SRL

Authorized Users only.
Fully protected copyright VisualNote SRL
