# GC-20
### Portable Geiger counter, dosimeter and monitoring station with features such as:
- Touchscreen controlled, intuitive GUI
- Displays counts per minute, current dose, and accumulated dose on homescreen
- Sensitive and reliable SBM-20 Geiger-Muller tube
- Variable integration time for averaging dose rate
- Timed count mode for measuring low doses
- Choose between Sieverts and Rems as the units for the displayed dose rate
- User adjustable alert threshold
- Adjustable calibration to relate CPM to dose rate for various isotopes
- Audible clicker and LED indicator toggled on and off from homescreen
- Offline data logging
- Post bulk logged data to cloud service (ThingSpeak) to graph, analyze and/or save to computer
- MQTT publishing for integration with Home Assistant and other home automation systems
- Monitoring Station mode: device stays connected to WiFi and publishes ambient radiation data via MQTT every 30 seconds

### Overview and User's Guide

![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/homepage.jpg)

The homepage displays the current effective dose rate, counts per minute, and the cumulative dose since the device was turned on. The integration time can be changed by tapping the "INT 60 s" button and the user can choose between 60, 180 and 5 seconds of integration. A shorter time allows faster response to changing radiation levels at the expense of accuracy. Using 180 seconds gives the least amount random fluctuation.

![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/timed_count_setup.jpg)
![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/timed_count_running.jpg)

The timed count mode allows for the measurement of very low doses of radiation such as the radiation coming from bananas, brazil nuts, or granite. It can also be used to test for high levels of Radon. Select the time interval in five minute increments, hit begin, and the GC-20 starts counting and displaying the accumulated counts over time. It also displays the real time CPM during the timed count. 
The WiFi symbol next to the battery level indicator means that the GC-20 is currently in monitoring station mode and is connected to WiFi. More on that below.

![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/settings.jpg)

The settings menu has four options: Dose Units, Alert Threshold, Calibration, and Logging and WiFi. 

![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/units.jpg)
![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/threshold.jpg)
![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/calibration.jpg)

The Dose Units option allows the user to choose between either Sieverts or Rems for the effective dose rate and the accumulated dose.


The user can adjust the threshold at which the home page displays a "High Radiation Level" alert in red, in increments of 1 uSv/hr.



The calibration factor that relates CPM to uSv/hr can be changed using this option. The geiger tube has different sensitivities to different isotopes, so to keep the dose readings reasonably accurate, a default conversion factor of 175 is used. This will accurately report the gamma dose from Cesium-137. 

![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/logging_off.jpg)
![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/logging_on.jpg)

Finally, tapping the Logging and WiFi button offers a further sub-menu of options related to setting up wifi for the first time, logging data, uploading logged data, and setting the device mode. When data logging is on, an "L" symbol appears on the top bar.

![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/wifi_setup.jpg)

To set up WiFi and MQTT, tap on WiFi setup and the GC-20 will enter AP mode. Using any WiFi-connected device with a web browser, like a phone or a laptop, search local networks and connect to the open network named "GC20". The browser will either automatically take you to the setup page, or if it doesn't, enter the url 192.168.4.1. Select your network name from the scanned list, enter the password, and configure your MQTT broker settings (broker IP/hostname, port, username, password, and topic). Hit save and the GC-20 will save the settings to permanent memory and reset itself. You can update WiFi or MQTT settings at any time by repeating the above. To update only MQTT settings without changing WiFi, simply submit the form without selecting a WiFi network.

![test](https://raw.githubusercontent.com/pra22/GC-20/master/Images/device_mode.jpg)

In the Device Mode menu option, the user can choose between the portable Geiger counter mode, or configure the device as a radiation monitoring station. In the monitoring station mode, the GC-20 is always connected to WiFi and publishes radiation data to the configured MQTT broker every 30 seconds. If it can't connect to WiFi during startup, it waits for 30 seconds before starting in portable Geiger counter mode.

### MQTT & Home Assistant Integration

The GC-20 publishes radiation data as JSON to an MQTT topic, making it easy to integrate with Home Assistant or any MQTT-compatible system. Each message contains:

```json
{
  "cpm": 23.00,
  "dose_rate": 0.131,
  "dose_rate_unit": "uSv/h",
  "total_dose": 0.0108,
  "total_dose_unit": "uSv"
}
```

To add the GC-20 as a sensor in Home Assistant, add the following to your `configuration.yaml`:

```yaml
mqtt:
  sensor:
    - name: "GC-20 CPM"
      state_topic: "homeassistant/sensor/gc20"
      value_template: "{{ value_json.cpm }}"
      unit_of_measurement: "CPM"
      icon: mdi:radioactive

    - name: "GC-20 Dose Rate"
      state_topic: "homeassistant/sensor/gc20"
      value_template: "{{ value_json.dose_rate }}"
      unit_of_measurement: "uSv/h"
      icon: mdi:radiation

    - name: "GC-20 Total Dose"
      state_topic: "homeassistant/sensor/gc20"
      value_template: "{{ value_json.total_dose }}"
      unit_of_measurement: "uSv"
      icon: mdi:counter
```

> **Note:** If the device is set to Rems mode, change the units to `mR/h` and `mR` respectively.

Logged data can also be uploaded via MQTT. Each stored data point is published as an individual message with a `"logged": true` field to distinguish it from live readings.

