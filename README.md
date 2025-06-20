# Carbon Intensity for your Postcode

This is a simple way of making the regional carbon footprint available in Home Assistant.
For Predbat users it is alternative to the HACs addin [Carbon Intensity UK](https://github.com/jfparis/sensor.carbon_intensity_uk) as described here in the [Predbat documentation](https://springfall2008.github.io/batpred/energy-rates/#uk-grid-carbon-intensity).

The basic premise is that you can get data directly from [National Grid's carbon intensity website](https://carbonintensity.org.uk/)
```
$ curl -X GET https://api.carbonintensity.org.uk/regional/postcode/LE16 -H 'Accept: application/js'
```
Note LE16 is the first part of my postcode. You will need to add your own postcode to make this work.

This returns a json string with a data format that looks like:
```
{'data':
  [ {'regionid': 9,
     'dnoregion': 'WPD East Midlands',
     'shortname': 'East Midlands',
     'postcode': 'LE16',
     'data': [ {'from': '2025-06-16T07:00Z',
                'to': '2025-06-16T07:30Z',
                'intensity': {'forecast': 206, 'index': 'high'},
                'generationmix': [ {'fuel': 'biomass', 'perc': 27.9},
                                   {'fuel': 'coal', 'perc': 0},
                                   {'fuel': 'imports', 'perc': 3.5},
                                   {'fuel': 'gas', 'perc': 43.8},
                                   {'fuel': 'nuclear', 'perc': 0.6},
                                   {'fuel': 'other', 'perc': 0},
                                   {'fuel': 'hydro', 'perc': 0},
                                   {'fuel': 'solar', 'perc': 13.4},
                                   {'fuel': 'wind', 'perc': 10.7} ]
                  } ]
      } ]
}
```
This GET can be used to create a rest sensor in Home Assistant with the following yaml.    

UK post codes are made up of two parts the outward, which is the post town plus a number, and the inward which defines the local street number.
The Carbon intensity values returned are calculated for each Distribution Network Operator (DNO) which can be found from the first, outward part of a postcode.

## Steps:
1. Create an text input helper called input_text.postcode_outward as below:

![image](https://github.com/user-attachments/assets/c7807173-b2a4-423b-afef-2675157f9ebc)


2. Add this line in configuration.yaml
```
rest: !include rest.yaml
```
3. Create a file in ithe home Assistant config directory rest.yaml
```
- resource_template: 'https://api.carbonintensity.org.uk/regional/postcode/{{states("input_text.postcode_outward")}}'
  scan_interval: 600
  headers:
    Accept: "application/json"
    Content-Type: "application/json"
  sensor:
    - name: "Carbon Intensity PostCode"
      unique_id: carbonintensitypostcode
      unit_of_measurement: 'g/kWh'
      icon: 'mdi:molecule-co2'
      availability: "{{ value_json is defined }}"
      value_template: "{{ value_json['data'][0]['data'][0]['intensity']['forecast'] }}"
```
4. Add an entities card
```
type: entities
entities:
  - entity: input_text.postcode_outward
  - entity: sensor.carbon_intensity_postcode
state_color: true
show_header_toggle: false
title: Carbon Intensity
```
Which produces this:

![image](https://github.com/user-attachments/assets/79052169-577a-4b71-b09c-53b4b14cd5bc)

## Improvement
This could be further improved by doing away with the input_text sensor and reading the outward postcode as the left part of a postcode input in apps.yaml as below:

[ <img src="https://github.com/user-attachments/assets/2b40bac2-1000-4db0-bf1f-5e05d25e5f50" width=70%  alt="Demo on YouTube"/>](https://springfall2008.github.io/batpred/energy-rates/#uk-grid-carbon-intensity)

## Thanks
Although this is my concept, the rest sensor yaml was pretty much written for me by @Troon in the HA community forum. Thank you!

I also noticed that @olivershingler covered this topic in his 2023 [video](https://youtu.be/w5fcff63agY?si=CBhvuYhpmoFMVCqe)
