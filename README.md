# WeatherXM Home Assistant Integration

In Home Assistant, create a new serial sensor in the `configuration.yaml` file. If you have a separate `sensors.yaml` file, do it there.

Next, find out which port your WeatherXM is plugged into. Go to **Settings -> System -> Hardware -> All Hardware** and look for USB. There should be an entry with `usb-Espressif_USB_JTAG...` in the description. Click on the arrow to the right and copy the device path. It should be something like `/dev/ttyACM0`.

Next, add the sensor to the `.yaml` file as shown below. If you don't have a `sensors.yaml` file, do this in the `configuration.yaml` file under the `sensors` section.

```yaml
- platform: serial
  serial_port: /dev/ttyACM0
  name: WeatherXM
```

Then, add the following template sensors to extract the values from the debug output of the WeatherXM panel.

```yaml
- platform: template
  sensors:
    weatherxm_temperature:
      friendly_name: "WeatherXM Temperature"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"temperature":' in value %}
          {{ value.split('"temperature": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "°C"
    
    weatherxm_humidity:
      friendly_name: "WeatherXM Humidity"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"humidity":' in value %}
          {{ value.split('"humidity": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "%"
    
    weatherxm_wind_speed:
      friendly_name: "WeatherXM Wind Speed"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"wind_speed":' in value %}
          {{ value.split('"wind_speed": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "m/s"
    
    weatherxm_wind_gust:
      friendly_name: "WeatherXM Wind Gust"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"wind_gust":' in value %}
          {{ value.split('"wind_gust": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "m/s"
    
    weatherxm_wind_direction:
      friendly_name: "WeatherXM Wind Direction"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"wind_direction":' in value %}
          {{ value.split('"wind_direction": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "°"
    
    weatherxm_illuminance:
      friendly_name: "WeatherXM Illuminance"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"illuminance":' in value %}
          {{ value.split('"illuminance": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "lux"
    
    weatherxm_solar_irradiance:
      friendly_name: "WeatherXM Solar Irradiance"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if 'Solar rad:' in value %}
            {{ value.split('Solar rad: ')[1].split(' ')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "w/m²"
    
    weatherxm_uv_index:
      friendly_name: "WeatherXM UV Index"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"uv_index":' in value %}
            {{ value.split('"uv_index": ')[1].split(',')[0] }}
        {% else %}
          0
        {% endif %}
    
    weatherxm_precipitation_accumulated:
      friendly_name: "WeatherXM Precipitation Accumulated"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"precipitation_accumulated":' in value %}
          {{ value.split('"precipitation_accumulated": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "mm"
    
    weatherxm_battery_voltage:
      friendly_name: "WeatherXM Battery Voltage"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"ws_bat_mv":' in value %}
          {{ value.split('"ws_bat_mv": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "mV"
    
    weatherxm_pressure:
      friendly_name: "WeatherXM Pressure"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"pressure":' in value %}
          {{ value.split('"pressure": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "hPa"

    weatherxm_precipitation_rate:
      friendly_name: "WeatherXM Precipitation Rate"
      value_template: >-
        {% set value = states('sensor.weatherxm') %}
        {% if '"precipitation_rate":' in value %}
          {{ value.split('"precipitation_rate": ')[1].split(',')[0] }}
        {% else %}
          unknown
        {% endif %}
      unit_of_measurement: "mm/h"
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

  daily_rain_fall:
    name: Daily Rainfall
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
    {{ (states('sensor.weatherxm_precipitation_accumulated') | float) -
    (states('input_number.weatherxm_rain_accumulated_storage') | float) }}
target:
  entity_id: input_number.daily_rain_fall
```

Save the automation as `Update Daily Rainfall`. Next we need to reset the daily rainfall at 0:00.
Create a new Automation and then click on Add Trigger.
Select Time and Location -> Time.
Select fixed time, and change the time to 00:00:00
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
This time, click on Choose Entity and select `input_number.daily_rain_fall`.
In Value just enter a zero.

That's it. Now the accumulated rainfall number from the WeatherXM station will be saved to
input_number.weatherxm_rain_accumulated_storage, and the daily rainfall will be zero'd.
When the weatherxm_precipitation_accumulated changes, the stored value will be subtracted
from the reading received from the station, giving you an accurate value for your daily rainfall.

### Final Steps

Restart Home Assistant, and you should see all your new weather sensors. Note that it may take some time for the initial values to be populated, as they depend on the next polling interval of the weather station.
