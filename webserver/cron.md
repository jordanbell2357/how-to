# cron

## curl

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

We make `weather.sh`:

```bash
#!/usr/bin/env bash
# weather.sh

STATIONID="1147W"

YEAR=$(date +"%Y")
MONTH=$(date +"%m")
DAY=$(date +"%d")
TIMESTAMP=$(date +"%Y%m%dT%H%M%S")

mkdir -p weather/$YEAR/$MONTH/$DAY

curl -s https://api.weather.gov/stations/$STATIONID/observations/latest?require_qc=false |
jq '.properties | {station: .stationId, stationName: .stationName, timestamp: .timestamp, temperature_C: .temperature.value, dewpoint_C: .dewpoint.value, windDirection_deg: .windDirection.value, windSpeed_km_h: .windSpeed.value, barometricPressure_Pa: .barometricPressure.value, precipitationLast3Hours_mm: .precipitationLast3Hours.value}' > weather.json

mv weather.json weather/$YEAR/$MONTH/$DAY/"$STATIONID"_"$TIMESTAMP".json
```

```bash
chmod +x weather.sh
```

We execute `weather.sh` twice.

```console
ubuntu@vps-9e6a8f0e:~$ ls weather/2025/11/12/*
weather/2025/11/12/1147W_20251112T001442.json  weather/2025/11/12/1147W_20251112T001757.json
```


```console
ubuntu@vps-9e6a8f0e:~$ cat weather/2025/11/12/*
{
  "station": "1147W",
  "stationName": "The Weather Channel",
  "timestamp": "2025-11-11T23:50:00+00:00",
  "temperature_C": 7.89,
  "dewpoint_C": -5.58,
  "windDirection_deg": 44,
  "windSpeed_km_h": 3.24,
  "barometricPressure_Pa": 102438.2,
  "precipitationLast3Hours_mm": null
}
{
  "station": "1147W",
  "stationName": "The Weather Channel",
  "timestamp": "2025-11-12T00:00:00+00:00",
  "temperature_C": 7.78,
  "dewpoint_C": -5.75,
  "windDirection_deg": 76,
  "windSpeed_km_h": 3.24,
  "barometricPressure_Pa": 102438.2,
  "precipitationLast3Hours_mm": null
}
```

## cron

<https://cronitor.io/guides/cron-jobs>

> At its most basic level, a cron job is an entry written into a table called the cron table, otherwise known as the crontab for short. This entry contains a schedule and a command to be executed. The cron daemon (crond) looks for entries in the crontab to determine what jobs it should run, and when it should run them according to the specified schedule.

We run

```console
ubuntu@vps-9e6a8f0e:~$ crontab -e
no crontab for ubuntu - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 2
crontab: installing new crontab
```

In this session using vim, we add the following line to the end of the file:

```
*/15 * * * * /home/ubuntu/weather.sh
```

Then we wait and check logs for cron's activity:

```console
ubuntu@vps-9e6a8f0e:~/weather/2025/11/12$ journalctl -n 10 -u cron --no-pager
Nov 12 00:24:59 vps-9e6a8f0e sudo[11402]:     root : PWD=/root ; USER=root ; COMMAND=/usr/bin/certbot renew -q
Nov 12 00:24:59 vps-9e6a8f0e sudo[11402]: pam_unix(sudo:session): session opened for user root(uid=0) by root(uid=0)
Nov 12 00:25:00 vps-9e6a8f0e sudo[11402]: pam_unix(sudo:session): session closed for user root
Nov 12 00:25:00 vps-9e6a8f0e CRON[10847]: pam_unix(cron:session): session closed for user root
Nov 12 00:25:00 vps-9e6a8f0e CRON[11420]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Nov 12 00:25:00 vps-9e6a8f0e CRON[11421]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Nov 12 00:25:00 vps-9e6a8f0e CRON[11420]: pam_unix(cron:session): session closed for user root
Nov 12 00:30:01 vps-9e6a8f0e CRON[11485]: pam_unix(cron:session): session opened for user ubuntu(uid=1000) by ubuntu(uid=0)
Nov 12 00:30:01 vps-9e6a8f0e CRON[11486]: (ubuntu) CMD (/home/ubuntu/weather.sh)
Nov 12 00:30:03 vps-9e6a8f0e CRON[11485]: pam_unix(cron:session): session closed for user ubuntu
```
