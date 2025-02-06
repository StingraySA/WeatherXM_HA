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
          unknown
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
```

### Excluding from Recorder

The serial sensor generates a lot of entries in your logbook. To prevent this, edit your `configuration.yaml` file under `recorder`:

```yaml
recorder:
  exclude:
    entities: sensor.weatherxm
```

### Final Steps

Restart Home Assistant, and you should see all your new weather sensors. Note that it may take some time for the initial values to be populated, as they depend on the next polling interval of the weather station.
