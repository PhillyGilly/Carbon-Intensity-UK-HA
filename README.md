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

## Footnote
Borrowing unashamedly from @Olivershingler and with a bit of encouragement from @Troon, I added a few more enities relating to generation mix and a couple of cards as below.

Here are the cards.

Here is the code in rest.yaml:
```
    - name: "Carbon Intensity genmix coal"
      unique_id: carbonintensitygenmixcoal
      unit_of_measurement: '%'
      icon: mdi:molecule-co2
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','coal')|first)['perc']|round(1) }}"

    - name: "Carbon Intensity genmix imports"
      unique_id: carbonintensitygenmiximports
      unit_of_measurement: '%'
      icon: mdi:transmission-tower-import
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','imports')|first)['perc']|round(1) }}"

    - name: "Carbon Intensity genmix gas"
      unique_id: carbonintensitygenmixgas
      unit_of_measurement: '%'
      icon: mdi:fire-circle
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','gas')|first)['perc']|round(1) }}"

    - name: "Carbon Intensity genmix nuclear"
      unique_id: carbonintensitygenmixnuclear
      unit_of_measurement: '%'
      icon: mdi:atom
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','nuclear')|first)['perc']|round(1) }}"

    - name: "Carbon Intensity genmix other"
      unique_id: carbonintensitygenmixother
      unit_of_measurement: '%'
      icon: mdi:molecule-co2
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','other')|first)['perc']|round(1) }}"

    - name: "Carbon Intensity genmix hydro"
      unique_id: carbonintensitygenmixhydro
      unit_of_measurement: '%'
      icon: mdi:hydro-power
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','hydro')|first)['perc']|round(1) }}"

    - name: "Carbon Intensity genmix solar"
      unique_id: carbonintensitygenmixsolar
      unit_of_measurement: '%'
      icon: mdi:solar-panel-large
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','solar')|first)['perc']|round(1) }}"

    - name: "Carbon Intensity genmix wind"
      unique_id: carbonintensitygenmixwind
      icon: mdi:wind-turbine
      unit_of_measurement: '%'
      availability: "{{ value_json is defined }}"
      value_template: "{{ (value_json['data'][0]['data'][0]['generationmix']|selectattr('fuel','==','wind')|first)['perc']|round(1) }}"
```
Here is the code for the auto entities cards (you my need to get this custom card from HACs:
```
type: custom:auto-entities
card:
  type: entities
  title: Carbon Intensity Generation Mix
card_param: null
filter:
  exclude:
    - state: < 0.1
entities:
  - entity: sensor.carbon_intensity_genmix_biomass
  - entity: sensor.carbon_intensity_genmix_coal
  - entity: sensor.carbon_intensity_genmix_gas
  - entity: sensor.carbon_intensity_genmix_hydro
  - entity: sensor.carbon_intensity_genmix_imports
  - entity: sensor.carbon_intensity_genmix_nuclear
  - entity: sensor.carbon_intensity_genmix_other
  - entity: sensor.carbon_intensity_genmix_solar
  - entity: sensor.carbon_intensity_genmix_wind
state_color: true
show_header_toggle: false
title: Carbon Intensity
sort:
  numeric: true
  reverse: true
  method: state
```
Here is the code for the guage:
```
type: gauge
name: Grid Carbon Intensity
needle: true
segments:
  - from: 0
    color: green
  - from: 40
    color: lightgreen
  - from: 120
    color: yellow
  - from: 200
    color: orange
  - from: 290
    color: red
  - from: 450
    color: darkred
max: 600
unit: gCO2/kWh
entity: sensor.carbon_intensity_postcode
```
