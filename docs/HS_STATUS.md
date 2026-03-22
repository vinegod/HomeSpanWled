# HomeSpan Status LED and Control Button

In addition to keeping track of all the HomeKit Accessories, Services, and Characteristics you've implemented in a HomeSpan sketch, HomeSpan also keeps track of its global operating **state**.  This state can be read programmatically from within your sketch (see the [Reference API](Reference.md) for details) but is more often communicated visually via different blinking patterns of an *optional* Status LED you can implement on your device.  This LED can be a simple analog single-color LED or an addressable Pixel LED. Many manufacturers include one or both of these on their ESP32 boards, though you can use a separate LED if desired.

To enable HomeSpan's Status LED functionality, simple call `homeSpan.enableStatusLED()` near the top of your HomeSpan sketch.  This function takes different types of parameters depending on the specific type of LED you are using.  Please see the [Reference API](Reference.md) for details on the various options available for this function.

### Normal Operating States

The first column in the table below lists all possible states for HomeSpan when it is running in its normal operating mode.[^normal]  The second column provides a brief description of each state.  And the third column provides a graphic representation of the associated flashing pattern HomeSpan displays on the Status LED (if implemented) when HomeSpan is in each state.  Note that graphic representations are all scaled to show the first 6 seconds of each pattern, which should be always sufficient to make the pattern (number of blinks, duration, etc.) obvious.

[^normal]:  HomeSpan is in its normal operating mode when it is repeatedly calling `homeSpan.poll()`, either because you explicitly added `homeSpan.poll()` to the main `loop()` of your HomeSpan sketch, or you called `homeSpan.autoPoll()` at the end of the `setup()` portion of your sketch, which spawns a separate task that repeatedly calls `homeSpan.poll()` for you in the background.

<div align="center">

#### Normal Operating States

|HomeSpan Status|Description|Status LED Pattern|
|---|---|---|
|HS_WIFI_NEEDED|WiFi Credentials Needed|<img src="images/ledPatterns/slowSingleBlink.svg" width=300>|
|HS_WIFI_CONNECTING|WiFi Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|HS_ETH_CONNECTING|Ethernet Connecting|<img src="images/ledPatterns/slowFlashing.svg" width=300>|
|HS_PAIRING_NEEDED|Device not yet Paired|<img src="images/ledPatterns/slowDoubleBlink.svg" width=300>|
|HS_PAIRED|Paired and waiting for HomeKit|<img src="images/ledPatterns/slowDoubleBlinkInverted.svg" width=300>|
|HS_CONNECTED|Device is Connected to HomeKit|<img src="images/ledPatterns/on.svg" width=300>|
|HS_WIFI_SCANNING|WiFi Scanning Started|<img src="images/ledPatterns/longTripleBlink.svg" width=300>|

</div>

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
