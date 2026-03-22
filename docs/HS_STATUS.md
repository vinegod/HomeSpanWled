# HomeSpan States, Status LED, and Control Button

In addition to keeping track of all the HomeKit Accessories, Services, and Characteristics you've implemented in a HomeSpan sketch, HomeSpan also keeps track of its global operating **state** (e.g. connecting to WiFi, updating via OTA, etc.).  This state can be read programmatically from within your sketch (see below for details) but is more often communicated visually via different blinking patterns of an *optional* Status LED you can implement on your device.  This LED can be a simple analog single-color LED or an addressable Pixel LED. Many manufacturers include one or both of these on their ESP32 boards, though you can use a separate LED if desired.

To enable HomeSpan's Status LED functionality, simple call `homeSpan.enableStatusLED()` near the top of your HomeSpan sketch.  This function takes different types of parameters depending on the specific type of LED you are using.  Please see the [Reference API](Reference.md) for details on the various options available for this function.

The tables below present all possible HomeSpan states, a brief description of each state, and a graphic representation of the associated flashing pattern HomeSpan displays on the Status LED (if implemented) in each state.  Note that graphic representations are all scaled to show the first 6 seconds of each pattern, which should be always sufficient to make the pattern (number of blinks, duration, etc.) obvious.

### Connectivity States

The states below indicate increasing levels of readiness for a fully complete and secure connection between your HomeSpan device and your HomeKit hub(s).

|HomeSpan Status|Description|Status LED Pattern|
|---|---|---|
|HS_SETUP|Setting up|<img src="images/ledPatterns/off.svg" width=300>|
|||
|HS_ETH_CONNECTING|Ethernet Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|HS_WIFI_CONNECTING|WiFi Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|HS_WIFI_NEEDED|WiFi Credentials Needed|<img src="images/ledPatterns/slowSingleBlink.svg" width=300>|
|HS_PAIRING_NEEDED|Device not yet Paired|<img src="images/ledPatterns/slowDoubleBlink.svg" width=300>|
|HS_PAIRED|Paired and waiting for HomeKit|<img src="images/ledPatterns/slowDoubleBlinkInverted.svg" width=300>|
|HS_CONNECTED|Device is Connected to HomeKit|<img src="images/ledPatterns/on.svg" width=300>|
|||
|HS_WIFI_SCANNING|WiFi Scanning Started|<img src="images/ledPatterns/longTripleBlink.svg" width=300>|

Notes: 

* When HomeSpan first starts, its state is set to **HS_SETUP**, and all the code in the `setup()` portion of your sketch is executed.

* Next, if you have configured an Ethernet interface, HomeSpan sets its state to **HS_ETH_CONNECTING** and tries to connect to your Home network via Ethernet.  If an Ethernet interface is not found, HomeSpan sets its state to **HS_WIFI_CONNECTING** and tries to connect to your home network via WiFi, unless you have not yet saved or specified your WiFi Credentials, in which case the state is set to **HS_WIFI_NEEDED**. See [HomeSpan WiFi and Ethernet Connectivity](Networks.md) for complete details on all the options available for specifying and establishing network connectivity between HomeSpan and your home network.

* Upon establishing connectivity between HomeSpan and your home network, HomeSpan then sets its state to either **HS_PAIRING_NEEDED** or **HS_PAIRED** depending on whether or not you've already paired the device to HomeKit using the Home App on your iPhone.

* Once you pair the device with the Home App, or if it is already paired, HomeSpan then waits to receive *inbound* connection requests from HomeKit (usually from one of your HomeKit Hubs) and sets its state to **HS_CONNECTED** upon verifying and establishing its first direct secure connection to HomeKit.[^inbound]

[^inbound]: Apple does not allow HomeKit devices to initiate outbound connections to HomeKit.  Bi-directional connections between HomeKit and a HomeKit device require HomeKit to initiate the connection, which the HomeKit device authenticates against the secure keys established during the pairing process.  Unfortunately, there is no reliable way for a HomeKit device to prompt or force HomeKit to initiate a connection.  This is why the Home App sometimes indicates a device is "not responding" even though the device is ready and operating correctly.

* Whenever the HomeSpan state is set to **HS_CONNECTED**, the device will respond to Accessory control requests from the Home App or via Siri.  With the one exception below, the device will not respond to Accessory control requests from the Home App or Siri if HomeSpan is in any other state.[^weblog]

[^weblog]: If you have enabled HomeSpan's Web Log functionality, HomeSpan will response to HTTP requests for the Web Log in any of the following states: **HS_PAIRING_NEEDED**, **HS_PAIRED**, and **HS_CONNECTED**.

* *The exception*: If you are using a mesh WiFi network with multiple BSSIDs and have configured HomeSpan to perform optional periodic rescans to search for any access points with stronger signals, HomeSpan will temporarily set its state to **HS_WIFI_SCANNING** while these rescans are underway. This rescanning does *not* interfere with any existing or newly-requested connections.  Once rescanning is complete, HomeSpan will revert to its prior state if no access point with a stronger signal is found.  If, however, an access point with a stronger signal is found, HomeSpan will disconnect from the existing access point, set its state to **HS_WIFI_CONNECTING**, and try to re-connect to your home network.[^scanning]

[^scanning]: Note HomeSpan also sets its state to **HS_WIFI_SCANNING** whenever you start the process to enter your WiFi Credentials, either from the CLI or from within the HomeSpan Setup Access Point.

* If at any point network connectivity is lost, the device is unpaired, or HomeKit terminates all of its secure connections to your device, HomeSpan will reset its state accordingly and the above process picks up from that point.

|HomeSpan Status|Description|Status LED Pattern|
|---|---|---|
|HS_ENTERING_CONFIG_MODE|Entering Command Mode|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|||
|HS_CONFIG_MODE_EXIT|1. Exit Command Mode|<img src="images/ledPatterns/fastBlink1.svg" width=300>|
|HS_CONFIG_MODE_REBOOT|2. Reboot Device|<img src="images/ledPatterns/fastBlink2.svg" width=300>|
|HS_CONFIG_MODE_LAUNCH_AP|3. Launch Access Point|<img src="images/ledPatterns/fastBlink3.svg" width=300>|
|HS_CONFIG_MODE_UNPAIR|4. Unpair Device|<img src="images/ledPatterns/fastBlink4.svg" width=300>|
|HS_CONFIG_MODE_ERASE_WIFI|5. Erase WiFi Credentials|<img src="images/ledPatterns/fastBlink5.svg" width=300>|
|||
|HS_CONFIG_MODE_EXIT_SELECTED|Exiting Command Mode...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|HS_CONFIG_MODE_REBOOT_SELECTED|Rebooting Device...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|HS_CONFIG_MODE_LAUNCH_AP_SELECTED|Launching Access Point...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|HS_CONFIG_MODE_UNPAIR_SELECTED|Unpairing Device...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|HS_CONFIG_MODE_ERASE_WIFI_SELECTED|Erasing WiFi Credentials...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|

|HomeSpan Status|Description|Status LED Pattern|
|---|---|---|
|HS_OTA_STARTED|OTA Update Started|<img src="images/ledPatterns/mediumTripleBlink.svg" width=300>|
|HS_REBOOTING|REBOOTING|<img src="images/ledPatterns/off.svg" width=300>|
|HS_FACTORY_RESET|Performing Factory Reset...|<img src="images/ledPatterns/off.svg" width=300>|

|HomeSpan Status|Description|Status LED Pattern|
|---|---|---|
|HS_AP_STARTED|Access Point Started|<img src="images/ledPatterns/rapidDoubleBlink.svg" width=300>|
|HS_AP_CONNECTED|Access Point Connected|<img src="images/ledPatterns/mediumDoubleBlink.svg" width=300>|
|HS_AP_TERMINATED|Access Point Terminated|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|


The optional ***homeSpan*** method, `void setStatusCallback(void (*func)(HS_STATUS status))`, can be used to create a callback function, *func*, that HomeSpan calls whenever its status changes.  HomeSpan passes *func* a single argument, *status*, of type *HS_STATUS*, defined as follows:


The ***homeSpan*** method `char* statusString(HS_STATUS s)`, is a convenience function for converting any of the above enumerations to short, pre-defined character string messages as follows:


### Example:

```C++
#include "HomeSpan.h"

void setup(){
  homeSpan.setStatusCallback(statusUpdate);   // set callback function
  ...
  homeSpan.begin();
  ...
}

// create a callback function that simply prints the pre-defined short messages on the Serial Monitor whenever the HomeSpan status changes

void statusUpdate(HS_STATUS status){
  Serial.printf("\n*** HOMESPAN STATUS CHANGE: %s\n",homeSpan.statusString(status));
}
```

You can of course create any alternative messsages, or take any actions desired, in *func* and do not need to use the pre-defined strings above.

---

[↩️](Reference.md) Back to the Reference API page
