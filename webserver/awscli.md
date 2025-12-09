# awscli

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

https://support.us.ovhcloud.com/hc/en-us/articles/4603838122643-Getting-started-with-Object-Storage

```console
ubuntu@histfile:~$ aws configure
AWS Access Key ID [****************66c4]:
AWS Secret Access Key [****************c682]:
Default region name [ca-east-tor]:
Default output format [json]:
```

Then

```bash
echo "endpoint_url = https://s3.ca-east-tor.io.cloud.ovh.net/" >> .aws/config
```

```
ubuntu@histfile:~$ cat .aws/config
[default]
region = ca-east-tor
output = json
endpoint_url = https://s3.ca-east-tor.io.cloud.ovh.net/
```

Then

```console
ubuntu@histfile:~$ aws s3 ls
2025-12-09 04:13:34 rich-lorentz
```


```console
ubuntu@histfile:~$ aws s3 cp names.zip s3://rich-lorentz
upload: ./names.zip to s3://rich-lorentz/names.zip
```

```console
ubuntu@histfile:~/ssa$ aws s3 ls s3://rich-lorentz
2025-12-09 06:49:37    7726516 names.zip
```


We see the file in the web GUI interface

<img width="1505" height="542" alt="image" src="https://github.com/user-attachments/assets/89fa4e03-bc68-4095-b149-8e854a1e847d" />
