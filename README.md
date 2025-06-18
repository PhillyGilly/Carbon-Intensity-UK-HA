# Carbon Intensity for your Postcode

This is a simple alternative to the HACs addin "Carbon Intensity UK" for making the regional carbon footprint available in Home Assistant and to other apps such as Predbat.

The basic premise is that you can get data from National Grid's website 
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

In configuration.yaml
```
rest:
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


This could be further improved by adding postcode as an input in apps.yaml as below
```
  # Carbon Intensity data from National grid
  carbon_intensity: 're:(sensor.carbon_intensity_postcode)'
  postcode: LE16 9xy
```
Although this is my concept, the rest sensor yaml was pretty much written for me by @Troon in the HA community forum. Thank you!
