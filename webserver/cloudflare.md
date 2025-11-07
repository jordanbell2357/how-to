# Cloudflare

```console
ubuntu@vps-9e6a8f0e:~$ curl "https://api.cloudflare.com/client/v4/accounts/22bb7f3ef7dac79d81ff569a8f4851e6/tokens/verify" \
-H "Authorization: Bearer Nf0uIot7yqlcP8HIxNfRcyUVdb-FxkcLOEMvpAqe"
{"result":{"id":"85cee884d0efea69bb2a568a06cb51c1","status":"active"},"success":true,"errors":[],"messages":[{"code":10000,"message":"This API Token is valid and active","type":null}]}
```

https://linuxconfig.org/automate-dynamic-ip-updates-for-your-domain-with-cloudflare-and-bash-script

https://developers.cloudflare.com/api/resources/dns/subresources/records/methods/get/


We want to track the public IP address of our server. We will make a cron job that does this and logs it.


```console
ubuntu@vps-9e6a8f0e:~$ hostname -I
148.113.200.36 2607:5300:205:200::2dec
```

