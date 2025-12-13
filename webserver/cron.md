# cron

## curl

```bash
curl -s https://api.weather.gov/stations/1147W/observations/latest?require_qc=false | jq '.properties | {station: .stationId, stationName: .stationName, timestamp: .timestamp, temperature_C: .temperature.value, dewpoint_C: .dewpoint.value, windDirection_deg: .windDirection.value, windSpeed_km_h: .windSpeed.value, barometricPressure_Pa: .barometricPressure.value, precipitationLast3Hours_mm: .precipitationLast3Hours.value}'
```

```json
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

## cron

<https://opensource.com/article/17/11/how-use-cron-linux>

<https://cronitor.io/guides/cron-jobs>

<https://crontab-generator.org/>

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



Later:

```console
ubuntu@jordanbell:~/weather/2025/12$ TZ=EST date
Sat Dec 13 01:00:45 EST 2025
ubuntu@jordanbell:~$ find $(pwd) -name "1147W_*.json" | wc -l
885
```

```console
ubuntu@jordanbell:~$ find $(pwd) -name "1147W_*.json" | head
/home/ubuntu/weather/2025/12/13/1147W_20251213T024501.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T004501.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T010001.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T040001.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T054501.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T060001.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T034501.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T051501.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T000001.json
/home/ubuntu/weather/2025/12/13/1147W_20251213T014501.json
ubuntu@jordanbell:~$ find $(pwd) -name "1147W_*.json" | tail
/home/ubuntu/weather/2025/12/05/1147W_20251205T000001.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T024501.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T233001.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T081501.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T043001.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T221501.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T104501.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T094501.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T021502.json
/home/ubuntu/weather/2025/12/05/1147W_20251205T210001.json
```


We use xargs [^xargs] and cat, and then jq [^jq] to combine the JSON files.

[^xargs]: https://pubs.opengroup.org/onlinepubs/009604599/utilities/xargs.html

[^jq]:

```console
ubuntu@jordanbell:~$ find $(pwd) -name "1147W_*.json" | tail -n 2 | xargs cat | jq --slurp
[
  {
    "station": "1147W",
    "stationName": "The Weather Channel",
    "timestamp": "2025-12-05T02:00:00+00:00",
    "temperature_C": 7.28,
    "dewpoint_C": 3.11,
    "windDirection_deg": 0,
    "windSpeed_km_h": 0,
    "barometricPressure_Pa": 102065.7,
    "precipitationLast3Hours_mm": null
  },
  {
    "station": "1147W",
    "stationName": "The Weather Channel",
    "timestamp": "2025-12-05T20:40:00+00:00",
    "temperature_C": 7.94,
    "dewpoint_C": 6.68,
    "windDirection_deg": 0,
    "windSpeed_km_h": 0,
    "barometricPressure_Pa": 101490.02,
    "precipitationLast3Hours_mm": null
  }
]
```


```console
ubuntu@jordanbell:~$ find $(pwd) -name "1147W_*.json" | xargs cat | jq --slurp > weather_2025_12_13.json
```


```console
buntu@jordanbell:~$ jq ".[0]" weather_2025_12_13.json
{
  "station": "1147W",
  "stationName": "The Weather Channel",
  "timestamp": "2025-12-13T02:30:00+00:00",
  "temperature_C": 8.5,
  "dewpoint_C": 4.04,
  "windDirection_deg": 0,
  "windSpeed_km_h": 0,
  "barometricPressure_Pa": 101896.38,
  "precipitationLast3Hours_mm": null
}
```


Each entry in the JSON file is made of 11 lines, along with initial and terminal JSON list brackets.

```console
ubuntu@jordanbell:~$ echo "885 * 11" | bc
9735
ubuntu@jordanbell:~$ jq ". | length" weather_2025_12_13.json
885
ubuntu@jordanbell:~$ wc -l weather_2025_12_13.json
9737 weather_2025_12_13.json
```


We download the combined JSON file using WinSCP.

<img width="1491" height="645" alt="image" src="https://github.com/user-attachments/assets/dfd4d497-f059-4ae3-a191-6329c95a4f39" />



