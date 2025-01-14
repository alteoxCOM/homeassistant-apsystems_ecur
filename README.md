# Home-Assistant APSystems ECU-R, ECU-B and ECU-C Integration
This is a custom component for [Home-Assistant](http://home-assistant.io) that adds support for the [APsystems](http://www.apsystems.com) Energy Communication Unit (ECU) so that you are able to monitor your PV installation (inverters) in detail.
Note for later ECU-R (SunSpec logo on the back) and ECU-C owners: there might still be a problem with compatibility issues that must be solved, read on for more important info on this!


## Background & acknowledgement
This integration queries the ECU with a set interval for new data. This was done without a public API, and by listening to and interpreting the protocol the APSystems ECU phone app (ECUapp) uses when setting up the PV array.

This couldn't have been done without the hardwork of @checking12 and @HAEdwin on the home assistant forum, and all the other people from this forum (https://gathering.tweakers.net/forum/list_messages/2032302/1)

## Prerequisites
You own an APSystems ECU and any combination of YC600, YC1000, DS3 or QS1/QS1A inverter.
This component only works if the ECU is attached to your network by Wifi. To enable and configure WiFi on the ECU, use the ECUapp (downloadable via Appstore or Google Play) and temporarily enable the ECU's accesspoint by pressing the button on the side of the ECU. Then connect your phone's WiFi to the ECU's accesspoint to enable the ECUapp to connect and configure the ECU. Although there's no need to also attach the ECU by ethernet cable (for the ECU-B LAN ports are disabled), feel free to do so if you like but alway connect this integration to the ECU's WiFi enabled IP-Address.

## Release notes
### v1.2.13
* Corrected one flaw where the socket was left open when an exeption occured
* Bug fix calling the function to capture an exception when returned data=b''

### v1.2.12
* Added better handling of the socket for ECU-R (later models with SunSpec logo on the back) and ECU-C

### v1.2.11
* Remove async socket code and refactor some code to support the coming soon HTTP method for ECU_R_PRO (and possibly ECU-C).
* Removed the multiple retries if we get bad data from the ECU, just use cache, and hope next iteration is fine in attempt to provide more stability to the ECU

### v1.2.9 (beta)
* Remove async socket code from APSystemsECUR revert to using standard socket calls and using homeassistant's `async_add_executor_job` functionality
* Add QS1 type 805

### v1.2.2 (beta)
* Fix some other issues in error handling and caching

### v1.2.0 (beta)
New features
* Remove `apsystems_ecur.start_query` and `apsystems_ecu.end_query` service, now there is a switch on the ECU called `switch.ecu_query_device` if the switch is on, will query the device, if off it will read from cache.  Can be used with an automation to turn on/off querying when the sun has set or risen
* Add support for the download diagnostics button the on the devices - hopefully will help debugging when there are problems
* Add 2 additional sensors `sensor.ecu_inverters` and `sensor.ecu_inverters_online` to show you how man inverters have been configured in your ECU and how many are currently reporting data

Bug fixes
* Change code to always close socket between commands to ECU (before this was only done on the ECU-C and ECU_R_PRO)
* Change the socket code to be a little more simplified and hopefully work better
* Add more exception handling and detect error conditions

### v1.1.2
Make ECU-C devices behave like ECU_R_PRO devices and close the socket down between each query.  Add support for new ds3 inverter type 704

### v1.1.1
Added support to setup the integration in the new config flow GUI.  Fixed a caching issue when the ECU is down on startup leading to creation of sensor entries.  Once you install this update all configuration is done via the GUI.

### Old release notes
```
v1.0.0 First release
v1.0.1 Revised the readme, added support for YC1000 and added versioning to the manifest
v1.0.2 Added support for QS1A
v1.0.3 Added support for 2021.8.0 (including energy panel), fixed some issues with ECU_R_PRO
v1.0.4 Added optional scan_interval to config
v1.0.5 Fixed energy dashboard and added HACS setup option description in readme.md
v1.0.6 Replaces deprecated device_state_attributes, added ECU-B compatibility
2022.1.0 Improved configuration notes, applied CalVer, cleanup code, improvements on ECU data
2022.1.0 - [update] Attempt to fix issues with ECU_R_PRO, detect 0 from lifetime energy to prevent issues in energy dashboard
v1.0.7 - provide stateclass for current_power and today_energy and start git tagging version number
v1.0.8 - fix HA version in hacs.json file
```

## Setup
Option 1:
Easiest option, install the custom component using HACS by searching for "APSystems ECU-R". If you are unable to find the integration in HACS, select HACS in left pane, select Integrations. In the top pane right from the word Integrations you can find the menu (three dots above eachother). Select Custom Repositories and add the URL: https://github.com/ksheumaker/homeassistant-apsystems_ecur below that select category Integration.


## Configuration
choose [Configuration] > [Devices & Services] > [+ Add Integration] and search for search for "APSystems PV solar ECU" which enables you to configure the integration settings. Provide the WIFI IP-address from the ECU, and set the update interval (300 seconds is the default).
_Warning_ the ECU device isn't the most powerful querying it more frequently could lead to stability issues with the ECU and require a power cycle.

Although you can query the ECU 24/7, it is an option to stop the query after sunset and start the query again at sunrise. You can do this by adding automations and by triggering the ECU Query Device switch entity.  

Reason for this are the maintenance tasks that take place on the ECU around 02.45-03.15 AM local time. During this period the ECU does not provide data which results in error messages in the log if the integration tries to query for data. During maintenance, the ECU is checking whether all data to the EMA website has been updated, clearing cached data and the ECU is looking for software updates, updating the ECU firmware when applicable. Besides the log entries no harm is done if you query the ECU 24/7.

## Data available
The component supports getting data from the PV array as a whole as well as each individual inverter.

### Array Level Sensors

These sensors will show up under an `ECU [ID]` device, where [ID] is the unique ID of your ECU

* sensor.ecu_current_power - total amount of power (in W) being generated right now
* sensor.ecu_today_energy - total amount of energy (in kWh) generated today now
* sensor.ecu_lifetime_energy - total amount of energy (in kWh) generated from the lifetime of the array

### Inverter Level Sensors

A new device will be created for each inverter called `Inverter [UID]` where [UID] is the unique ID of the Inverter

* sensor.inverter_[UID]_frequency - the AC power frequency in Hz
* sensor.inverter_[UID]_voltage - the AC voltage in V
* sensor.inverter_[UID]_temperature - the temperature of the invertor in your local unit (C or F)
* sensor.inverter_[UID]_signal - the signal strength of the zigbee connection
* sensor.inverter_[UID]_power_ch_[1-4] - the current power generation (in W) of each channel of the invertor - number of channels will depend on inverter model

## TODO
1. Code cleanup - it probably needs some work
2. Improve ECU-R-PRO firmware compatibility and ECU-C compatibility. If you can contribute in bugfixing/testing you're invited to help
