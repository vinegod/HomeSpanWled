# HomeSpan States, the HomeSpan Status LED and the HomeSpan Control Button

In addition to keeping track of all the HomeKit Accessories, Services, and Characteristics you've implemented in a HomeSpan sketch, HomeSpan also keeps track of its global operating **state** (e.g. connecting to WiFi, updating via OTA, etc.).  This state can be read programmatically from within your sketch (see [Reading the HomeSpan State]() below for details) but is more often communicated visually via different blinking patterns of an *optional* Status LED you can implement on your device.  This LED can be a simple analog single-color LED or an addressable Pixel LED. Many manufacturers include one or both of these on their ESP32 boards, though you can use a separate LED if desired.

To enable HomeSpan's Status LED functionality, simple call `homeSpan.enableStatusLED()` near the top of your HomeSpan sketch.  This function takes different types of parameters depending on the specific type of LED you are using.  Please see the [Reference API](Reference.md) for details on the various options available for this function.

The tables below present all possible HomeSpan states, a brief description of each state, and a graphic representation of the associated flashing pattern HomeSpan displays on the Status LED (if implemented) in each state.  Note that graphic representations are all scaled to show the first 6 seconds of each pattern, which should be always sufficient to make the pattern (number of blinks, duration, etc.) obvious.

### Primary Operating States

The primary operating states below indicate increasing levels of readiness for a fully complete and secure connection between your HomeSpan device and your HomeKit hub(s).

<div align="center">

|HomeSpan Status|Description|Status LED Pattern|
|---|---|---|
|HS_SETUP|Setting up|<img src="images/ledPatterns/off.svg" width=300>|
|HS_ETH_CONNECTING|Ethernet Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|HS_WIFI_NEEDED|WiFi Credentials Needed|<img src="images/ledPatterns/slowSingleBlink.svg" width=300>|
|HS_WIFI_CONNECTING|WiFi Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|HS_PAIRING_NEEDED|Device not yet Paired|<img src="images/ledPatterns/slowDoubleBlink.svg" width=300>|
|HS_PAIRED|Paired and waiting for HomeKit|<img src="images/ledPatterns/slowDoubleBlinkInverted.svg" width=300>|
|HS_CONNECTED|Device is Connected to HomeKit|<img src="images/ledPatterns/on.svg" width=300>|
|HS_WIFI_SCANNING|WiFi Scanning Started|<img src="images/ledPatterns/longTripleBlink.svg" width=300>|

</div>

When HomeSpan first starts its state is set to **HS_SETUP** and it executes all the code in the `setup()` portion of your sketch.  Once complete, HomeSpan starts executing the code in the 'loop()` portion of your sketch, which should include a call to `homeSpan.poll()` unless you already called `homeSpan.autoPoll()` in the `setup()` portion of the sketch.  In either case, upon the very first call to `homeSpan.poll()`, HomeSpan begins by trying to connect to your home network.

If you've configured HomeSpan to use an Ethernet interface, HomeSpan sets its state to **HS_ETH_CONNECTING** and tries to connect to your home network via Ethernet.  Otherwise, HomeSpan checks for stored WiFi Credentials and, if found, sets its state to **HS_WIFI_CONNECTING** and tries to connect to your home network via WiFi using those credentials.  If you've not yet stored any WiFi Credentialds (not have explicitly specified them in your sketch), HomeSpan sets its state to **HS_WIFI_NEEDED**, alerting you that it cannot proceed to connect to your home network until you provide your WiFI Credentials (or, alternatively, configure an Ethernet interface).  Please see [HomeSpan WiFi and Ethernet Connectivity](Networks.md) for complete details on how to specify and establish connectivity between HomeSpan and your home network.

Once connectivity between HomeSpan and your home network have been established, if you have already paired the device with HomeKit, HomeSpan sets it state to **HS_PAIRED**, which indicates it is ready to receive secure connection requests from your HomeKit Hub(s).  If instead you have not yet paired the device with HomeKit, HomeSpan sets its state to **HS_PAIRING_NEEDED**, alerting you that it cannot connect to HomeKit until you use the Home App on your iPhone to pair the device.

While in the **HS_PAIRED** state, HomeSpan waits for an inbound connection request from a HomeKit Hub.  This is because HomeKit devices cannot *initiate* an outbound connection directly to a HomeKit Hub.  Bi-directional communication between HomeKit and a HomeKit Hub is of course possible, but only once HomeSpan receives an inbound request from a HomeKit Hub and has verfified the authentication key provided match those of the pairing data stored.

Upon receiving and verifying its first connection from a HomeKit Hub, HomeSpan sets its state to **HS_CONNECTED**.  **This is the desired operating state of HomeSpan. It indicates your device has fully established one or more secure connections to one or more HomeKit Hubs and can be controlled via your Home App or Siri.*

Note the that HomeSpan states above can change at any time in response to changes in connectivity.  For example, if your HomeKit Hub terminates all of its secure connections with your device (for whatever reason) even though you did not unpair the device, HomeSpan will revert its state to **HS_PAIRED**, alerting you that it is waiting for HomeKit to re-establish a secure connection.  Until it does, you will not be able to control the device from your Home App.  As described more fully below, you can add optional logic to your sketch to check for such an occurance and take action (such as forcing the device to reboot) if HomeKit does not re-establish a secure connection to your device after a period of time.

Loss of network connectivity will also trigger state changes.  For example, if WiFi connectivity is lost, HomeSpan will reset its state to **HS_WIFI_CONNECTING** as it tries to re-connect, after which it then follows the remaining processes described above.

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
