# HomeSpan Status and the HomeSpan Status LED

In addition to keeping track of all the HomeKit Accessories, Services, and Characteristics you've implemented in a HomeSpan sketch, HomeSpan also keeps track of its global operating status (e.g. connecting to WiFi, updating via OTA, etc.).  The current HomeSpan status, represented as an *enum* of type **HS_STATUS**, can be read directly from within your sketch as well as communicated visually via different blinking patterns of an *optional* Status LED you can implement on your device.  This LED can be a simple analog single-color LED or an addressable Pixel LED. Many manufacturers include one or both of these on their ESP32 boards, though you can use a separate LED if desired.

To read the status from within your sketch, call `homeSpan.getStatus()`.  This function returns a `std::pair<HS_STATUS, uint32_t>` where the first element indicates the current status and the second element indicates duration (in seconds) that HomeSpan has been in its current state.  This duration resets to zero whenever the HomeSpan status changes, and can also be reset manually by calling `homeSpan.resetStatusDuration()`.

For example, a adding the following code to the main `loop()` in your sketch will monitor HomeSpan status and output a warning every 30 seconds if the device has not yet been paired:

```C++
void loop(){

  homeSpan.poll();

  auto [ status, duration] = homeSpan.getStatus();      // using "auto" is an easy way to read the data from a function that returns a std::pair
  if(status==HS_PAIRING_NEEDED && duration > 30){
    Serial.printf("Warning: HomeSpan is not paired.\n");
    homeSpan.resetStatusDuration();
  }  
}
```
Additionally, the optional ***homeSpan*** method, `void setStatusCallback(void (*func)(HS_STATUS status))`, can be used to create a callback function, *func*, that HomeSpan calls whenever its status changes.  HomeSpan passes *func* a single argument, *status*, of type *HS_STATUS*, defined as follows:
To enable HomeSpan's Status LED functionality, simply call `homeSpan.enableStatusLED()` near the top of your HomeSpan sketch.  This function takes different types of parameters depending on the specific type of LED you are using.  Please see the [Reference API](Reference.md) for details on the various options available for this function.


The tables below present all possible HomeSpan states, a brief description of each state, and a graphic representation of the associated flashing pattern HomeSpan displays on the Status LED (if implemented) in each state.  Note that graphic representations are all scaled to show the first 6 seconds of each pattern, which should be always sufficient to make the pattern (number of blinks, duration, etc.) obvious.

|HomeSpan Status (HS_STATUS)|Status String|Status LED Pattern|
|---|---|---|
|<details><summary>HS_INITIAL_SETUP</summary><i>HomeSpan is running code in the setup() portion of the sketch</details>|WiFi Credentials Needed|<img src="images/ledPatterns/off.svg" width=300>|
|<details><summary>HS_WIFI_NEEDED</summary><i>WiFi Credentials have not yet been set/stored, and an Ethernet interface is not available</details>|WiFi Credentials Needed|<img src="images/ledPatterns/slowSingleBlink.svg" width=300>|
|<details><summary>HS_WIFI_SCANNING</summary><i>HomeSpan is in the process of scanning (or re-scanning) for WiFi network Access Points</details>|WiFi Scanning Started|<img src="images/ledPatterns/longTripleBlink.svg" width=300>|
|<details><summary>HS_WIFI_CONNECTING</summary><i>HomeSpan is trying to connect to the WiFi network specified in the stored WiFi Credentials</details>|WiFi Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|<details><summary>HS_ETH_CONNECTING</summary><i>HomeSpan is trying to connect to an Ethernet network using the Ethernet interface configured</details>|Ethernet Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|<details><summary>HS_PAIRING_NEEDED</summary><i>HomeSpan is connected to a network, but the device has not yet been paired with HomeKit</details>|Device not yet Paired|<img src="images/ledPatterns/slowDoubleBlink.svg" width=300>|
|<details><summary>HS_PAIRED</summary><i>HomeSpan is connected to a network and the device has been paired with HomeKit, but there are no active HomeKit connections</details>|Paired and waiting for HomeKit|<img src="images/ledPatterns/slowDoubleBlinkInverted.svg" width=300>|
|<details><summary>HS_CONNECTED</summary><i>HomeSpan is connected to a network, the device has been paired to HomeKit, and there is at least one active HomeKit connection</details>|Device is Connected to HomeKit|<img src="images/ledPatterns/on.svg" width=300>|
|<details><summary>HS_ENTERING_CONFIG_MODE</summary><i>User has requested the device to enter into Command Mode</details>|Entering Command Mode|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_EXIT</summary><i>HomeSpan is in Command Mode with "Exit Command Mode" specified as choice</details>|1. Exit Command Mode|<img src="images/ledPatterns/fastBlink1.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_REBOOT</summary><i>HomeSpan is in Command Mode with "Reboot" specified as choice</details>|2. Reboot Device|<img src="images/ledPatterns/fastBlink2.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_LAUNCH_AP</summary><i>HomeSpan is in Command Mode with "Launch Access Point" specified as choice</details>|3. Launch Access Point|<img src="images/ledPatterns/fastBlink3.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_UNPAIR</summary><i>HomeSpan is in Command Mode with "Unpair Device" specified as choice</details>|4. Unpair Device|<img src="images/ledPatterns/fastBlink4.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_ERASE_WIFI</summary><i>HomeSpan is in Command Mode with "Erase WiFi Credentials" specified as choice</details>|5. Erase WiFi Credentials|<img src="images/ledPatterns/fastBlink5.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_EXIT_SELECTED</summary><i>User has selected "Exit Command Mode"</details>|Exiting Command Mode...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_REBOOT_SELECTED</summary><i>User has select "Reboot" from the Command Mode</details>|Rebooting Device...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_LAUNCH_AP_SELECTED</summary><i>User has selected "Launch AP Access" from the Command Mode</details>|Launching Access Point...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_UNPAIR_SELECTED</summary><i>User has seleected "Unpair Device" from the Command Mode</details>|Unpairing Device...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|<details><summary>HS_CONFIG_MODE_ERASE_WIFI_SELECTED</summary><i>User has selected "Erase WiFi Credentials" from the Command Mode</details>|Erasing WiFi Credentials...|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|<details><summary>HS_REBOOTING</summary><i>HomeSpan is in the process of rebooting the device</details>|REBOOTING|<img src="images/ledPatterns/off.svg" width=300>|
|<details><summary>HS_FACTORY_RESET</summary><i>HomeSpan is in the process of performing a Factory Reset of device</details>|Performing Factory Reset...|<img src="images/ledPatterns/off.svg" width=300>|
|<details><summary>HS_AP_STARTED</summary><i>The HomeSpan Setup Access Point is started but no client has yet connected</details>|Access Point Started|<img src="images/ledPatterns/rapidDoubleBlink.svg" width=300>|
|<details><summary>HS_AP_CONNECTED</summary><i>The HomeSpan Setup Access Point is running and a client is connected</details>|Access Point Connected|<img src="images/ledPatterns/mediumDoubleBlink.svg" width=300>|
|<details><summary>HS_AP_TERMINATED</summary><i>The HomeSpan Setup Access Point has been terminated</details>|Access Point Terminated|<img src="images/ledPatterns/rapidFlashing.svg" width=300>|
|<details><summary>HS_OTA_STARTED</summary><i>HomeSpan is in the process of receiving an Over-the-Air software update</details>|OTA Update Started|<img src="images/ledPatterns/mediumTripleBlink.svg" width=300>|


The optional ***homeSpan*** method, `void setStatusCallback(void (*func)(HS_STATUS status))`, can be used to create a callback function, *func*, that HomeSpan calls whenever its status changes.  HomeSpan passes *func* a single argument, *status*, of type *HS_STATUS*, defined as follows:

```C++
enum HS_STATUS {
  HS_WIFI_NEEDED,                         // WiFi Credentials have not yet been set/stored
  HS_WIFI_CONNECTING,                     // HomeSpan is trying to connect to the network specified in the stored WiFi Credentials
  HS_PAIRING_NEEDED,                      // HomeSpan is connected to central WiFi network, but device has not yet been paired to HomeKit
  HS_PAIRED,                              // HomeSpan is connected to central WiFi network and ther device has been paired to HomeKit
  HS_CONNECTED,                           // HomeSpan has at least one verified client connection from HomeKit
  HS_ENTERING_CONFIG_MODE,                // User has requested the device to enter into Command Mode
  HS_CONFIG_MODE_EXIT,                    // HomeSpan is in Command Mode with "Exit Command Mode" specified as choice
  HS_CONFIG_MODE_REBOOT,                  // HomeSpan is in Command Mode with "Reboot" specified as choice
  HS_CONFIG_MODE_LAUNCH_AP,               // HomeSpan is in Command Mode with "Launch Access Point" specified as choice
  HS_CONFIG_MODE_UNPAIR,                  // HomeSpan is in Command Mode with "Unpair Device" specified as choice
  HS_CONFIG_MODE_ERASE_WIFI,              // HomeSpan is in Command Mode with "Erase WiFi Credentials" specified as choice
  HS_CONFIG_MODE_EXIT_SELECTED,           // User has selected "Exit Command Mode" 
  HS_CONFIG_MODE_REBOOT_SELECTED,         // User has select "Reboot" from the Command Mode
  HS_CONFIG_MODE_LAUNCH_AP_SELECTED,      // User has selected "Launch AP Access" from the Command Mode
  HS_CONFIG_MODE_UNPAIR_SELECTED,         // User has seleected "Unpair Device" from the Command Mode
  HS_CONFIG_MODE_ERASE_WIFI_SELECTED,     // User has selected "Erase WiFi Credentials" from the Command Mode
  HS_REBOOTING,                           // HomeSpan is in the process of rebooting the device
  HS_FACTORY_RESET,                       // HomeSpan is in the process of performing a Factory Reset of device
  HS_AP_STARTED,                          // HomeSpan has started the Access Point but no one has yet connected
  HS_AP_CONNECTED,                        // The Access Point is started and a user device has been connected
  HS_AP_TERMINATED,                       // HomeSpan has terminated the Access Point 
  HS_OTA_STARTED,                         // HomeSpan is in the process of receiving an Over-the-Air software update  
  HS_WIFI_SCANNING,                       // HomeSpan is in the process of scanning for WiFi networks
  HS_ETH_CONNECTING                       // HomeSpan is trying to connect to an Ethernet network
};
```

The ***homeSpan*** method `char* statusString(HS_STATUS s)`, is a convenience function for converting any of the above enumerations to short, pre-defined character string messages as follows:

```C++
const char* Span::statusString(HS_STATUS s){
  switch(s){
    case HS_WIFI_NEEDED: return("WiFi Credentials Needed");
    case HS_WIFI_CONNECTING: return("WiFi Connecting");
    case HS_ETH_CONNECTING: return("Ethernet Connecting");
    case HS_PAIRING_NEEDED: return("Device not yet Paired");
    case HS_PAIRED: return("Device Paired.  Waiting for HomeKit Connection");
    case HS_CONNECTED: return("Device is Connected to HomeKit");
    case HS_ENTERING_CONFIG_MODE: return("Entering Command Mode");
    case HS_CONFIG_MODE_EXIT: return("1. Exit Command Mode"); 
    case HS_CONFIG_MODE_REBOOT: return("2. Reboot Device");
    case HS_CONFIG_MODE_LAUNCH_AP: return("3. Launch Access Point");
    case HS_CONFIG_MODE_UNPAIR: return("4. Unpair Device");
    case HS_CONFIG_MODE_ERASE_WIFI: return("5. Erase WiFi Credentials");
    case HS_CONFIG_MODE_EXIT_SELECTED: return("Exiting Command Mode...");
    case HS_CONFIG_MODE_REBOOT_SELECTED: return("Rebooting Device...");
    case HS_CONFIG_MODE_LAUNCH_AP_SELECTED: return("Launching Access Point...");
    case HS_CONFIG_MODE_UNPAIR_SELECTED: return("Unpairing Device...");
    case HS_CONFIG_MODE_ERASE_WIFI_SELECTED: return("Erasing WiFi Credentials...");
    case HS_REBOOTING: return("REBOOTING!");
    case HS_FACTORY_RESET: return("Performing Factory Reset...");
    case HS_AP_STARTED: return("Access Point Started");
    case HS_AP_CONNECTED: return("Access Point Connected");
    case HS_AP_TERMINATED: return("Access Point Terminated");
    case HS_OTA_STARTED: return("OTA Update Started");
    case HS_WIFI_SCANNING: return("WiFi Scanning Started");
    default: return("Unknown");
  }
}
```

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
