#!/usr/bin/env python

"""
This app will extract service name and weather condition for current day data from two APIs.
Namely, openweathermap.org and mobile311-dev.sfgov.org.

This app requires the following parameters to execute:
  - auth_id: type string with no quotes, the appid is required by the openweathermap.org API
    Example --auth_id=12345dkdkk222222

To run this app:
  $ python get_info.py

An example output shown below:
Service Name                             | Address                                  | Weather Condition
Blocked Driveway or Illegal Parking      | 910 Corbett Ave Twin Peaks               | Clouds
Tree Maintenance                         | 500 NAPLES ST                            | Clouds
Street or Sidewalk Cleaning              | Intersection of Cumberland St & Guerrer  | Clouds
Homeless Concerns                        | 450 THE EMBARCADERO                      | Clouds
Pothole or Street Issues                 | 1819 MARKET ST                           | Clouds
...


To integrate this app with other apps, this app can be modified to be
importable source of data to another app by changing the main section of
the app and make it as a function. The functions in the app can be called
by any other function with the two required parameters fed into them.

"""


import requests
import json
import datetime
import time
import argparse

WEATHER_URL = 'http://api.openweathermap.org/data/2.5/weather'
SERVICE_ISSUE_URL = 'http://mobile311.sfgov.org/open311/v2/requests.json'

def get_data(url, parms):
    resp = requests.get(url, params=parms)
    j_str = {}
    if resp.status_code == 200:
        j_str = json.loads(resp.text)
    return j_str


if __name__ == '__main__':
    pr = argparse.ArgumentParser(description='Need parameters before making the API calls')
    pr.add_argument('--auth_id', help='Authentication ID to OpenWeather API', required=True)
    args = pr.parse_args()

    weather_condition = ''
    current_date = datetime.datetime.now().strftime('%Y-%m-%dT%H:%M:%SZ')
    parms = {'start_date': current_date, 'status': 'open'}
    url =  SERVICE_ISSUE_URL
    j_str = get_data(url, parms)
    if j_str:
        url = WEATHER_URL
        print "{:40s} | {:40s} | {:40s}".format("Service Name", "Address", "Weather Condition")
        for i in j_str:
            lon, lat = i['long'], i['lat']
            parms = {'lat': lat, 'lon': lon, 'appid': args.auth_id}
            j_str = get_data(url, parms)
            if j_str:
                weather_condition = j_str['weather'][0]['main']
            else:
                weather_condition = 'Weather data unavailable'
            print "{:40s} | {:40s} | {:40s}".format(i['service_name'][:39], i['address'][:39], weather_condition)
            time.sleep(2)
    else:
        print 'System is unavailable. Please try again later.'
