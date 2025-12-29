# Randonmess sources

# NIST Randomness Beacon (Version 2.0 Beta) -- work in progress

https://csrc.nist.gov/Projects/interoperable-randomness-beacons/beacon-20

localRandomValue is "A random value represented as a 64-byte (512 bits) hex string."

```bash
ubuntu@LAPTOP-JBell:~$ curl -s https://beacon.nist.gov/beacon/2.0/pulse/last | jq ".pulse.localRandomValue"
"BB766DDA60D30293E1355CE902F9D91F5BEA362E368EB4578FCB4934008ED4CCEB3A710D55E319BEFB553A91A2B6BD3A9CC1C0D564103A608245D281FCF09BDC"
```


## CURBy - CU Randomness Beacon

https://random.colorado.edu/


## Australian National University Quantum Numbers (AQN)

https://quantumnumbers.anu.edu.au/

```console
ubuntu@LAPTOP-JBell:~$ API_KEY="<API Key>"
ubuntu@LAPTOP-JBell:~$ API_URL="https://api.quantumnumbers.anu.edu.au"
```

```bash
curl -X GET -H "x-api-key:${API_KEY}" "${API_URL}?length=5&type=hex8&size=2"
```

```json
{"success": true, "type": "hex8", "length": "5", "data": ["51f7", "5879", "ee84", "37f0", "0593"]}
```

## drand project

https://docs.drand.love/dev-guide/developer/http-api


```conole
ubuntu@LAPTOP-JBell:~$ curl -s api.drand.sh/public/latest | jq ".randomness"
"47fb67550165e1da717421c995ea11f8e51e38d0e33459ee3a79cb97a64f0703"
```

## Cisco Outshift Quantum Random Number Generator (QRNG)

https://outshift.cisco.com/quantum-random-number-generator


## LfD Quantum Random Number Generator

https://www.lfdr.de/QRNG/

> On traditional computing systems, random numbers are often generated with pseudo random number generators (PRNGs). Quantum random numbers generators use properties of quantum physics to generate a random source of high entropy. Our laboratory offers access to a QRNG source via a simple HTTP-API. The QRNG service is backed by a ID Quantique QRNG PCIe.

```bash
curl "https://lfdr.de/qrng_api/qrng?length=10&format=HEX"
```

```json
{"length":10,"qrn":"7900ee5cae1446aa08ae"}
```
