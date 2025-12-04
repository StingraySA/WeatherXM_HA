# WeatherXM Home Assistant Integration

This repo was created to help WeatherXM users retrieve data that is streamed from the weather station to the base station while it's plugged into your RPi or whatever you are using to host your Home Assistant installation.

**This does not affect your ability to mine at all. We are merely listening on the wire as the data is streamed from the weather station**

**Data streaming is sporadic during the first day that your stations is up and running. As soon as it's verified it's location, the data streams in real time**

![Sensors](./WeatherXM%20Sensors.png)

Here's an example of a weather dashboard created with the data:

![Dash](./WeatherXM%20Weather%20Dash.png)

### Start Here

First plug in the WeatherXM LCD screen into your RPi. Use the USB connector on the back of the unit. The one on the bottom doesn't have debugging enabled (Well on mine at least). Plug the other side into one of the USB 3.0 ports (The blue ones) on the RPi.

In Home Assistant, create a new serial sensor in the `configuration.yaml` file. If you have a separate `sensors.yaml` file, do it there.

Next, find out which port your WeatherXM is plugged into. Go to **Settings -> System -> Hardware -> All Hardware** and look for USB. There should be an entry with `usb-Espressif_USB_JTAG...` in the description. Click on the arrow to the right and copy the device path. It should be something like `/dev/ttyACM0`.

Next, add the sensor to the `.yaml` file as shown below. If you don't have a `sensors.yaml` file, do this in the `configuration.yaml` file under the `sensors` section.

```yaml
- platform: serial
  serial_port: /dev/ttyACM0
  name: WeatherXM
  value_template: "{{ value[:255] }}"
```

Then, add the following template sensors to extract the values from the debug output of the WeatherXM panel.

```yaml
    # ——— WEATHERXM SENSORS – PRESERVE LAST VALUE WHEN FIELD MISSING ———
    - name: WeatherXM Temperature
      unique_id: weatherxm_temperature
      unit_of_measurement: "°C"
      device_class: temperature
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_temperature') | float(default=none) %}
        {% if '"temperature":' in v %}{{ v.split('"temperature": ')[1].split(',')[0] | float }}
        {% elif 'Temperature:' in v %}{{ v.split('Temperature: ')[1].split(' C')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Humidity
      unique_id: weatherxm_humidity
      unit_of_measurement: "%"
      device_class: humidity
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_humidity') | float(default=none) %}
        {% if '"humidity":' in v %}{{ v.split('"humidity": ')[1].split(',')[0] | float }}
        {% elif 'Humidity:' in v %}{{ v.split('Humidity: ')[1].split(' %')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Wind Speed
      unique_id: weatherxm_wind_speed
      unit_of_measurement: "m/s"
      icon: mdi:weather-windy
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_wind_speed') | float(default=0) %}
        {% if '"wind_speed":' in v %}
          {{ v.split('"wind_speed": ')[1].split(',')[0] | float(default=cur) }}
        {% elif 'Wind speed:' in v %}
          {{ v.split('Wind speed: ')[1].split(' m/s')[0] | trim | replace(' m/S', '') | float(default=cur) }}
        {% else %}
          {{ cur }}
        {% endif %}

    - name: WeatherXM Wind Gust
      unique_id: weatherxm_wind_gust
      unit_of_measurement: "m/s"
      icon: mdi:weather-windy
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_wind_gust') | float(default=0) %}
        {% if '"wind_gust":' in v %}
          {{ v.split('"wind_gust": ')[1].split(',')[0] | float(default=cur) }}
        {% elif 'Gust:' in v %}
          {{ v.split('Gust: ')[1].split(' m/s')[0] | trim | replace(' m/S', '') | float(default=cur) }}
        {% else %}
          {{ cur }}
        {% endif %}

    - name: WeatherXM Wind Direction
      unique_id: weatherxm_wind_direction
      unit_of_measurement: "°"
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_wind_direction') | float(default=none) %}
        {% if '"wind_direction":' in v %}{{ v.split('"wind_direction": ')[1].split(',')[0] | float }}
        {% elif 'Wind Dir:' in v %}{{ v.split('Wind Dir: ')[1].split(' deg')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Illuminance
      unique_id: weatherxm_illuminance
      unit_of_measurement: lx
      device_class: illuminance
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_illuminance') | float(default=none) %}
        {% if '"illuminance":' in v %}{{ v.split('"illuminance": ')[1].split(',')[0] | float }}
        {% elif 'Illuminance:' in v %}{{ v.split('Illuminance: ')[1].split(' lux')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Solar Irradiance
      unique_id: weatherxm_solar_irradiance
      unit_of_measurement: "W/m²"
      device_class: irradiance
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_solar_irradiance') | float(default=none) %}
        {% if '"solar_irradiance":' in v %}{{ v.split('"solar_irradiance": ')[1].split(',')[0] | float }}
        {% elif 'Solar rad:' in v %}{{ v.split('Solar rad: ')[1].split(' w/m^2')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM UV Index
      unique_id: weatherxm_uv_index
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_uv_index') | float(default=none) %}
        {% if '"uv_index":' in v %}{{ v.split('"uv_index": ')[1].split(',')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Precipitation Accumulated
      unique_id: weatherxm_precipitation_accumulated
      unit_of_measurement: mm
      device_class: precipitation
      state_class: total_increasing
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_precipitation_accumulated') | float(default=none) %}
        {% if '"precipitation_accumulated":' in v %}{{ v.split('"precipitation_accumulated": ')[1].split(',')[0] | float }}
        {% elif 'Rain (cml):' in v %}{{ v.split('Rain (cml): ')[1].split(' mm')[0] | float }}
        {% else %}{{ cur if cur is not none else 0 }}{% endif %}

    - name: WeatherXM Daily Rainfall
      unique_id: weatherxm_daily_rainfall
      unit_of_measurement: mm
      device_class: precipitation
      state_class: total_increasing
      icon: mdi:weather-rainy
      state: >-
        {% set total = states('sensor.weatherxm_precipitation_accumulated') | float(default=none) %}
        {% set stored = states('input_number.weatherxm_rain_accumulated_storage') | float(default=0) %}
        {% if total is none %}0{% else %}{{ (total - stored) | max(0) | round(2) }}{% endif %}

    - name: WeatherXM Battery Voltage
      unique_id: weatherxm_battery_voltage
      unit_of_measurement: mV
      device_class: voltage
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_battery_voltage') | int(default=none) %}
        {% if '"ws_bat_mv":' in v %}{{ v.split('"ws_bat_mv": ')[1].split(',')[0] | int }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Pressure
      unique_id: weatherxm_pressure
      unit_of_measurement: hPa
      device_class: pressure
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_pressure') | float(default=none) %}
        {% if '"pressure":' in v %}{{ v.split('"pressure": ')[1].split(',')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Precipitation Rate
      unique_id: weatherxm_precipitation_rate
      unit_of_measurement: "mm/h"
      state_class: measurement
      state: >-
        {% set v = states('sensor.weatherxm') %}
        {% set cur = states('sensor.weatherxm_precipitation_rate') | float(default=none) %}
        {% if '"precipitation_rate":' in v %}{{ v.split('"precipitation_rate": ')[1].split(',')[0] | float }}
        {% else %}{{ cur if cur is not none else none }}{% endif %}

    - name: WeatherXM Wind Direction Compass
      unique_id: weatherxm_wind_compass
      icon: mdi:compass
      state: >-
        {% set d = states('sensor.weatherxm_wind_direction') | float(default=0) %}
        {% if d >= 337.5 or d < 22.5 %}N{% elif d < 67.5 %}NE{% elif d < 112.5 %}E{% elif d < 157.5 %}SE{% elif d < 202.5 %}S{% elif d < 247.5 %}SW{% elif d < 292.5 %}W{% elif d < 337.5 %}NW{% else %}-{% endif %}

    - name: WeatherXM Wind Speed (Knots)
      unique_id: weatherxm_wind_speed_knots
      unit_of_measurement: kt
      icon: mdi:weather-windy
      state_class: measurement
      state: >-
        {{ (states('sensor.weatherxm_wind_speed') | float(default=0) * 1.94384) | round(2) }}

    - name: WeatherXM Wind Speed (km/h)
      unique_id: weatherxm_wind_speed_kmh
      unit_of_measurement: "km/h"
      icon: mdi:weather-windy
      state_class: measurement
      state: >-
        {{ (states('sensor.weatherxm_wind_speed') | float(default=0) * 3.6) | round(2) }}
```

### Excluding from Recorder

The serial sensor generates a lot of entries in your logbook. To prevent this, edit your `configuration.yaml` file under `recorder`:

```yaml
recorder:
  exclude:
    entities: sensor.weatherxm
```

### Make it Look Nice

Add the following to your `configuration.yaml` file:

```yaml
homeassistant:
  customize: !include customize.yaml
```

Next create a file `customize.yaml` if it does not exist. If it does exist, just add below:

```yaml
sensor.weatherxm_temperature:
  icon: mdi:thermometer
sensor.weatherxm_humidity:
  icon: mdi:water-percent
sensor.weatherxm_wind_speed:
  icon: mdi:weather-windy
sensor.weatherxm_wind_gust:
  icon: mdi:weather-windy-variant
sensor.weatherxm_wind_direction:
  icon: mdi:compass
sensor.weatherxm_illuminance:
  icon: mdi:brightness-5
sensor.weatherxm_solar_irradiance:
  icon: mdi:solar-power
sensor.weatherxm_uv_index:
  icon: mdi:white-balance-sunny
sensor.weatherxm_precipitation_accumulated:
  icon: mdi:weather-rainy
sensor.weatherxm_battery_voltage:
  icon: mdi:battery
sensor.weatherxm_precipitation_rate:
  icon: mdi:weather-pouring
sensor.weatherxm_pressure:
  icon: mdi:gauge
sensor.weatherxm_wind_compass:
  icon: mdi:compass
sensor.weatherxm_wind_speed_knots:
  icon: mdi:speedometer
sensor.weatherxm_wind_speed_kmh:
  icon: mdi:speedometer
```

*Important: Restart Home Assistant for this to take effect or go to Developer Tools -> YAML -> Location & Customisations*

### Sorting Out Rainfall

It seems the rainfall doesn't get reset daily from the sensor, so let's do this in Home Assistant with some Automations:
First edit your `configuration.yaml` file, and add two input sensors:

```yaml
input_number:
  weatherxm_rain_accumulated_storage:
    name: WeatherXM Daily Rain Accumulation Storage
    min: 0
    max: 100000
    step: 1
    unit_of_measurement: "mm"

  weatherxm_daily_rainfall:
    name: WeatherXM Daily Rainfall
    min: 0
    max: 1000
    step: 1
    unit_of_measurement: "mm"
```

*Important: Restart Home Assistant so the new sensors can be enabled or go to Developer Tools -> YAML -> Input Numbers*

Next up we need to make the automations to calculate the daily rainfall and reset it at 0:00.
Go to Settings -> Automations & scenes -> Create Automation
Click on Add Trigger and select Entity -> State
Under Entity, select `weatherxm_precipitation_accumulated`.
Scroll down to Then Do and click on Add Action.
Type `input number` in the search box and select `Input Number: Set`
Click on the three dots on the right of the heading and select Edit in YAML.
Paste below:

```yaml
action: input_number.set_value
metadata: {}
data:
  value: >-
    {{ ((states('sensor.weatherxm_precipitation_accumulated') | float) -
        (states('input_number.weatherxm_rain_accumulated_storage') | float)) | round(2) }}
target:
  entity_id: input_number.weatherxm_daily_rainfall
```

Save the automation as `Update Daily Rainfall`. Next we need to reset the daily rainfall at 0:00.
Create a new Automation and then click on Add Trigger.
Select Time and Location -> Time.
Select fixed time, and change the time to 00:00:00

Now we need to add a condition, so the value can persist a restart, so click on Add Condition -> Other Conditions -> Template
and paste below in the Value Template field:

```yaml
{{ not is_state('sensor.uptime', 'unavailable') }}
```


Next Add Action then once again type `input number` in the search box and select `Input Number: Set`
Again click on the three dots, and Edit in YAML.
Paste below:

```yaml
action: input_number.set_value
metadata: {}
data:
  value: "{{ states('sensor.weatherxm_precipitation_accumulated') | float(0) }}"
target:
  entity_id: input_number.weatherxm_rain_accumulated_storage
```

Add another action and again search for `input number` and select `Input Number: Set`
This time, click on Choose Entity and select `input_number.weatherxm_daily_rainfall`.
In Value just enter a zero.

That's it. Now the accumulated rainfall number from the WeatherXM station will be saved to
input_number.weatherxm_rain_accumulated_storage, and the daily rainfall will be zero'd.
When the weatherxm_precipitation_accumulated changes, the stored value will be subtracted
from the reading received from the station, giving you an accurate value for your daily rainfall.

### Max Wind Speed

Now that we've got the basics down, let's add some nice to have sensors. First we'll do the max wind speed. Once again add an input sensor to the `configuration.yaml` file:

```yaml
weatherxm_daily_max_wind_speed:
  name: WeatherXM Daily Maximum Wind Speed
  min: 0
  max: 200
  step: 1
  unit_of_measurement: "km/h"
```
Once the input number has been added, go to Developer Tools and click on Input Numbers to load the new input number.
Next we need to add an automation to update our sensor value. So go to Settings -> Automations & scenes and then click on Create Automation.
In the top right hand corner, click on the three dots and select Edit in YAML.

```yaml
alias: "[WeatherXM] Update Max Wind Speed"
description: Updates daily max wind speed ONLY when a new record is reached
trigger:
  - platform: state
    entity_id: sensor.weatherxm_wind_speed_kmh   # ← this one MUST exist and be working
condition: []
action:
  - variables:
      current_wind: >-
        {{ states('sensor.weatherxm_wind_speed_kmh') | float(default=0) }}
      current_max: >-
        {{ states('input_number.weatherxm_daily_max_wind_speed') | float(default=0) }}
  - choose:
      - conditions: >-
          {{ current_wind > current_max }}
        sequence:
          - service: input_number.set_value
            target:
              entity_id: input_number.weatherxm_daily_max_wind_speed
            data:
              value: "{{ current_wind }}"
    default: []
mode: single
```

Now the sensor will update if the wind speed exceeds the value stored in our new input sensor. Now we need to reset this sensor every night at 0:00. So add another automation and click on the Edit in YAML.

```yaml
alias: Reset Daily Max Wind Speed - WeatherXM
description: ""
triggers:
  - trigger: time
    at: "00:00:00"
conditions: []
actions:
  - action: input_number.set_value
    metadata: {}
    data:
      value: 0
    target:
      entity_id: input_number.weatherxm_daily_max_wind_speed
mode: single
```

That's it. Now you've got a sensor to show the max wind speed for the day.

### Min/Max Temperatures

Right, on to min and max temps. Once again we'll use input numbers to save our values. Add below to your `configuration.yaml` file:

```yaml
  weatherxm_daily_min_temperature:
    name: Daily Minimum Temperature
    min: -50
    max: 100
    step: 0.1
    unit_of_measurement: "°C"

  weatherxm_daily_max_temperature:
    name: Daily Maximum Temperature
    min: -50
    max: 100
    step: 0.1
    unit_of_measurement: "°C"
```
Once the input numbers has been added, go to Developer Tools and click on Input Numbers to load the new input numbers.
As before we need to add automations to save our Min and Max temperatures. So add below automations as we did before:

```yaml
alias: Update Max Temperature - WeatherXM
description: Update Max Temperature - WeatherXM
trigger:
  - platform: state
    entity_id: sensor.weatherxm_temperature

condition:
  - condition: numeric_state
    entity_id: sensor.weatherxm_temperature
    above: input_number.weatherxm_daily_max_temperature
  - condition: numeric_state
    entity_id: sensor.weatherxm_temperature
    above: -50  # Ensures the sensor reports a valid temperature

action:
  - service: system_log.write
    data:
      message: "Triggered! Temperature: {{ states('sensor.weatherxm_temperature') }}"
      level: warning

  - service: input_number.set_value
    data:
      value: "{{ states('sensor.weatherxm_temperature') | float }}"
    target:
      entity_id: input_number.weatherxm_daily_max_temperature

mode: single
```

```yaml
alias: Update Min Temperature - WeatherXM
description: Update Max Temperature - WeatherXM
trigger:
  - platform: state
    entity_id: sensor.weatherxm_temperature

condition:
  - condition: numeric_state
    entity_id: sensor.weatherxm_temperature
    below: input_number.weatherxm_daily_min_temperature
  - condition: numeric_state
    entity_id: sensor.weatherxm_temperature
    above: -50  # Ensures the sensor reports a valid temperature

action:
  - service: system_log.write
    data:
      message: "Triggered! Temperature: {{ states('sensor.weatherxm_temperature') }}"
      level: warning

  - service: input_number.set_value
    data:
      value: "{{ states('sensor.weatherxm_temperature') | float }}"
    target:
      entity_id: input_number.weatherxm_daily_min_temperature

mode: single
```

All that's left now is to reset the Min/Max temps at 0:00. So add another automation:

```yaml
alias: Reset Daily Min/Max Temps - WeatherXM
description: ""
triggers:
  - trigger: time
    at: "00:00:00"
conditions: []
actions:
  - action: input_number.set_value
    metadata: {}
    data:
      value: "{{ states('sensor.weatherxm_temperature') | float }}"
    target:
      entity_id:
        - input_number.weatherxm_daily_max_temperature
        - input_number.weatherxm_daily_min_temperature
mode: single
```

### Final Steps

Restart Home Assistant, and you should see all your new weather sensors. Note that it may take some time for the initial values to be populated, as they depend on the next polling interval of the weather station.
