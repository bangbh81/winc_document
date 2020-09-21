For Device providers
# Device 초기 동작
디바이스 초기
1. 인증서가 있는지 검사
2. 없으면 Auth 서버에 접속하여 인증서 다운로드 시도
3. 인증서를 올바르게 다운로드 했다면 리부팅
4. IoT 허브로 접속하여 IsActivate 업데이트(!)
5. 인증서가 있다면 IsActivate 확인 후 정상 동작.

# Device twin update
## Desired property update (Cloud to Device)
winc.ai로부터 트윈이 업데이트되면 디바이스는 업데이트 된 desired property를 json 형식으로 받는다.
디바이스는 desired property를 받기 위해 아래와 같은 토픽을 구독해야 한다.

**Subscribe topic:** `$iothub/twin/PATCH/properties/desired/#`

###  Desired property
Desired property는 총 3 종류로 이루어 진다.
1. Firmware
2. HostFirmware
3. UserProperty

* JSON Structure
```json
"Firmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"}
},
"HostFirmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"}
},
"UserProperty": {
        "GPIO0":{"type":"boolean", "initial_state": "true"},
        "GPIO1":{"type":"boolean", "initial_state": "true"},
        "DownMessage":{"type": "string","maxNo":50}
}
```
* Device Twin Example
```json
"Firmware" : {
        "fwVersionName": "v1.0.15",
        "fwVersion": "15",
        "fwBinURL": "fota.winc.ai/binary/winc485",
        "fwToken": "lkj2134j3k453k4"
},
"UserProperty": {
        "GPIO0":true,
        "GPIO1":false,
        "DownMessage":"Hello Device!"
}
```


## Reported property update (Device to Cloud)
디바이스는 자신의 상태를 Device Twin의 Reported property를 업데이트하여 클라우드에 전달한다. Reported Property update는 아래와 같은 토픽에 JSON형식의 payload를 발행하면 수행된다.

**Publish topic:** `iothub/twin/PATCH/properties/reported/?rid={requeste id}` //request id can be any integer

### Getting device twin from the device
디바이스에서 트윈 정보를 읽기 위해서는 다음과 같은 순서를 따른다.
1. Subscribe topic `$iothub/twin/res/#`
2. Publish empty message to `$iothub/twin/GET/?$rid={request id}` //request id can be any integer
3. Json format of device properties will come via topic `$iothub/twin/res/{status}/?$rid={request id}`


## Event data update
디바이스로부터 발생된 데이터는 JSON 형식으로 아래 토픽을 통해 전달되어야 한다.

**Publish topic:** `devices/{devices name}/message/events/`

## Example device
WizFi360을 예제로써 설명함
#### Desired property
```json
"Firmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"}
},
"HostFirmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"}
},
"UserProperty": {
        "DownMessage":{"type": "string","maxNo":50}
}
```

```json
"Firmware" : {
        "fwVersion": {"type": "integer"},
},
"HostFirmware" : {
        "fwVersion": {"type": "integer"},
},
"UserProperty": {
        "UpMessage":{"type": "string","maxNo":50}
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU4NTkyNDY1OSwyMTE3NDI2ODM0XX0=
-->