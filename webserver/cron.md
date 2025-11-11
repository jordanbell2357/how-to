# cron

```console
ubuntu@LAPTOP-JBell:~/ovh$ curl -s https://api.weather.gov/stations/1147W/observations/latest?require_qc=false | jq '.properties | {station: .stationId, stationName: .stationName, timestamp: .timestamp, temperature_C: .temperature.value, dewpoint_C: .dewpoint.value, windDirection_deg: .windDirection.value, windSpeed_km_h: .windSpeed.value, barometricPressure_Pa: .barometricPressure.value, precipitationLast3Hours_mm: .precipitationLast3Hours.value}'
{
  "station": "1147W",
  "stationName": "The Weather Channel",
  "timestamp": "2025-11-11T05:40:00+00:00",
  "temperature_C": 1,
  "dewpoint_C": -5.76,
  "windDirection_deg": 233,
  "windSpeed_km_h": 4.824,
  "barometricPressure_Pa": 102607.53,
  "precipitationLast3Hours_mm": null
}
```


```console

```
