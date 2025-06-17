This is a simple alternative to the HACs addin "Carbon Intensity UK" for making the regional carbon footprint available in Home Assistant and to other apps such as Predbat.

In configuration.yaml
```
rest: !include rest.yaml
```

In rest.yaml
```
- resource: https://api.carbonintensity.org.uk/regional/postcode/LE16       
  scan_interval: 600
  sensor:
    - name: "Carbon Intensity PostCode"
      unique_id: carbonintensitypostcode
      unit_of_measurement: 'g/kWh'
      icon: 'mdi:molecule-co2'
      availability: "{{ value_json is defined }}"
      value_template: "{{ value_json['data'][0]['data'][0]['intensity']['forecast'] }}"
```

Note LE16 is the first part of my postcode. You will need to add your own
This could be further improved by finnessed by adding postcode as an input in apps.yaml as below
```
  # Carbon Intensity data from National grid
  carbon_intensity: 're:(sensor.carbon_intensity_postcode)'
  postcode: LE16 9xy
```
Although this is my concept, the rest sensor yaml was pretyy much written for me by @Troon in the HA community forum. Thak you!
