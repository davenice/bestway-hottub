# bestway-hottub

I wanted to be able to access the Bestway REST API for my Lay-z-spa Milan hot tub.

I used various other github projects that had already reverse engineered the Bestway API and picked through what they'd done and turned it into curl commands.
Here were my references: https://github.com/NicolasBernaerts/debian-scripts/tree/master/heatzy / https://github.com/cdpuk/ha-bestway/tree/main

First I downloaded the Bestway app, signed up for an account (with username and password which we'll use in a minute) and connected my hot tub. I only have one device in the Bestway app. I'm in the UK so I am connected to the EU endpoint of Gizwits.

Here's the authentication:
```
curl -X POST https://euapi.gizwits.com/app/login -H 'Content-Type: application/json' -H 'X-Gizwits-Application-Id: 98754e684ec045528b073876c34c7348' -d '{"username": "xxx@xxx.xxx", "password": "xxx", "lang": "en"}'
```
This returns JSON including a token (we'll use this in a sec) and an expiration which I think is seconds since the epoch - it's about 6 months away.
```
{"token": "MY_TOKEN", "uid": "MY_UID", "expire_at": 1702059675}%     
```

Next step is to ask what devices I've got against my user:
```
curl -X GET https://euapi.gizwits.com/app/bindings -H 'Content-Type: application/json' -H 'X-Gizwits-Application-Id: 98754e684ec045528b073876c34c7348' -H 'X-Gizwits-User-token: MY_TOKEN'
```
JSON returned - the 'did' is the device ID which is the important bit:
```
{"devices": [{"protoc": 3, "ws_port": 8080, "port_s": 8883, "gw_did": null, "host": "eum2m.gizwits.com", "sleep_duration": 3600, "port": 1883, "mcu_soft_version": "P151D102", "product_key": "fbc861733103419b9c2a09c71584cfe5", "state_last_timestamp": 1687548996, "role": "owner", "is_sandbox": false, "type": "normal", "product_name": "Airjet", "is_disabled": false, "mcu_hard_version": "P150D100", "wifi_soft_version": "0402003A", "dev_alias": "MY_TUB_NAME", "mesh_id": null, "is_online": true, "dev_label": [], "wss_port": 8880, "remark": "21", "did": "MY_DID", "mac": "bcff4dc8xxxxx", "passcode": "XXXXXXX", "wifi_hard_version": "00ESP826", "is_low_power": false}]}%  
```

Next we use the DID to request the latest information from the tub:
```
curl -X GET https://euapi.gizwits.com/app/devdata/MY_DID/latest -H 'Content-Type: application/json' -H 'X-Gizwits-Application-Id: 98754e684ec045528b073876c34c7348' -H 'X-Gizwits-User-token: MY_TOKEN'
```
```
{"did":"MY_DID","updated_at":1687548978,"attr":{"system_err2":0,"wave_appm_min":59940,"heat_timer_min":0,"heat_power":1,"earth":0,"wave_timer_min":59940,"system_err6":0,"system_err7":0,"system_err4":0,"system_err5":0,"heat_temp_reach":0,"system_err3":0,"system_err1":0,"system_err8":0,"system_err9":0,"filter_timer_min":0,"heat_appm_min":0,"power":1,"temp_set_unit":"\u6444\u6c0f","filter_appm_min":0,"temp_now":34,"wave_power":0,"locked":1,"filter_power":1,"temp_set":39}}%
```

From this I can grab the current temperature, which is what I was after.
