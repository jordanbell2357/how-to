# Cloudflare

<https://developers.cloudflare.com/api/resources/zones/methods/list/>

```console
ubuntu@vps-9e6a8f0e:~$ BEARER_TOKEN=
```

```bash
curl -X GET https://api.cloudflare.com/client/v4/zones \
  -H "Authorization: Bearer $BEARER_TOKEN" \
  -H "Content-Type: application/json" |
  jq '.result[0].id'
```

```console
ubuntu@vps-9e6a8f0e:~$ ZONE_ID=
```

<https://developers.cloudflare.com/api/resources/dns/subresources/records/methods/list/>

```bash
curl -X GET https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records \
  -H "Authorization: Bearer $BEARER_TOKEN" \
  -H "Content-Type: application/json" |
  jq '.result[0].id'
```

```console
ubuntu@vps-9e6a8f0e:~$ DNS_RECORD_ID=
```

We can determine our public IP address thus:

```console
ubuntu@vps-9e6a8f0e:~$ hostname -I | cut -d ' ' -f 1
148.113.200.36
ubuntu@vps-9e6a8f0e:~$ hostname -I | cut -d ' ' -f 2
2607:5300:205:200::2dec
```

We make `cloudflare.sh` in the user home directory:


```
$ crontab -e
# Add the following line to run the script every 5 minutes:
*/5 * * * * /path/to/update_dns.sh
```
