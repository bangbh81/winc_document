For Device providers
# Device 동작
디바이스 초기
1. 인증서가 있는지 검사
2. 없으면 Auth 서버에 접속하여 인증서 다운로드 시도
3. 인증서를 올바르게 다운로드 했다면 리부팅
4. IoT 허브로 접속하여 IsActivate 업데이트(!)
5. 인증서가 있다면 IsActivate 확인 후 정상 동작.

# MQTT Topic table
MQTT Topic should be used from the device side.
|Type | Topic Name|PUB/SUB|Description|
|:---:|:---|:---:|:---|
|Data|devices/{device_id}/messages/events/|PUB|Event Data|
|Data|devices/{device_id}/messages/devicebound/#|SUB|Command|
|Twin|$iothub/twin/PATCH/properties/reported/?rid={requeste id}|PUB|Device Twin D2C
|Twin|$iothub/twin/PATCH/properties/desired/#|SUB|Device Twin C2D

# Device twin
Device twin은 두가지 종류가 있다. Desired property는 Cloud to Device의 방향으로 Firmware Update와 관련된 내용들을 업데이트할 수 있으며 Reported Property는 Device to Cloud의 방향으로 Command에 대한 Response와 디바이스의 등록 여부 등의 기본적인 내용들을 업데이트 할 수 있다.

## Desired property update (Cloud to Device)
winc.ai로부터 트윈이 업데이트되면 디바이스는 업데이트 된 desired property를 json 형식으로 받는다.
디바이스는 desired property를 받기 위해 아래와 같은 토픽을 구독해야 한다.

**Subscribe topic:** `$iothub/twin/PATCH/properties/desired/#`

###  Desired property
Desired property는 총 2 종류로 이루어 진다.
1. Firmware
2. HostFirmware

* JSON Structure
```json
"Firmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"},
        "autoUpdate":{},
},
"HostFirmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"}
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

**Publish topic:** `$iothub/twin/PATCH/properties/reported/?rid={requeste id}` //request id can be any integer

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
#### Tags structure

```json
"device_hw_info": {
	"deviceManufacturer": {"type": "string"},
	"deviceCategory": {"type": "string"},
	"deviceModelNo":{"type": "string"},
	"hwVersion":{"type": "string"}
},
"deviceInfo":{
	"owner":{"type": "email"},
	"deviceNickName":{"type": "string"},
}
```
#### Desired property structure
```json
"Firmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"},
        "autoUpdate":{"type":"boolean"}
},
"HostFirmware" : {
        "fwVersionName": {"type": "string"},
        "fwVersion": {"type": "integer"},
        "fwBinURL": {"type": "URL"},
        "fwToken": {"type": "string"},
        "autoUpdate":{"type":"boolean"}
}
```
#### Reported property structure
```json
"DeviceStatus"  : {
	"FWVer": {"type": "integer", "initial_state":0},
	"hostFWVer": {"type": "integer", "initial_state":0},
	"isActivated": {"type": "true", "initial_state":"false"}
},
"UserProperty": {
        "UpMessage":{"type": "string","maxNo":50}
}
```

#### Event data
```json
"EventDataFormat":{
	"type":"string",
}
```

##### Example for the event data format
```json
"EventDataFormat":{
	"type":"json",
	"data":{
		"StrMessage": {"type":"string","maxlen":100},
		"Temperature": {"type":"integer", "max":50, "min":50},
		"Humidity": {"type":"integer", "max":50, "min":50}
	}
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NzkzOTg4ODgsMTAwNjg4NTU2NCwtMT
M0OTYyNDAwNiwtMTM5ODYzNDkxNSwtMTQwMTQ5NjUyMSwtMTAw
ODM4NDI3NSwxNTg1OTI0NjU5LDIxMTc0MjY4MzRdfQ==
-->