# Advanced usage of IoT Agents

This is a tutorial to demonstrate some advanced features available across IoT Agents. This repository contains a Docker Compose file to deploy the test environment to execute the tests.

[![Run in Postman](https://run.pstmn.io/button.svg)](https://god.gw.postman.com/run-collection/1382458-5e03cead-5f67-4956-8b18-d21001db33a9?action=collection%2Ffork&collection-url=entityId%3D1382458-5e03cead-5f67-4956-8b18-d21001db33a9%26entityType%3Dcollection%26workspaceId%3D2ee6f2c1-b67a-44af-9d62-83b77a28686e)

## Running the environment

```
git clone https://github.com/mapedraza/tutorial.iotagent-advanced
cd tutorial.iotagent-advanced
docker-compose up -d 
```

To stop the services

```
docker-compose down
```

## Healthcheck 

### Orion

```
curl --location --request GET 'http://localhost:1026/version'
```

### IoTA

```
curl --location --request GET 'http://localhost:4041/iot/about'
```

### Mosquitto

```
mosquitto_sub -h localhost -t '$SYS/#' -C 1
```

## Creating a service 

```
curl --location --request POST 'http://localhost:4041/iot/services' \
--header 'fiware-service: service' \
--header 'fiware-servicepath: /' \
--header 'Content-Type: application/json' \
--data-raw '{
 "services": [
   {
     "apikey":      "4jggokgpepnvsb2uv4s40d59ov",
     "cbroker":     "http://orion:1026",
     "entity_type": "Thing",
     "resource":    "/iot/d",
      "transport": "MQTT"
   }
 ]
}'
```

## Expressions

### Expressions usage in measures

Create device

```
curl --location --request POST 'http://localhost:4041/iot/devices' \
--header 'fiware-service: service' \
--header 'fiware-servicepath: /' \
--header 'Content-Type: application/json' \
--data-raw '{
    "devices": [
        {
            "apikey": "4jggokgpepnvsb2uv4s40d59ov",
            "device_id": "devbin10",
            "entity_name": "devbin0010",
            "entity_type": "dev",
            "protocol": "PDI-IoTA-UltraLight",
            "transport": "MQTT",
            "expressionLanguage": "jexl",
            "attributes": [
                {
                    "object_id":"lvl",
                    "name": "fillingLevel",
                    "type": "Text",
                    "expression": "lvl/100"
                }
            ]
        }
    ]
}'
```

Sending data

```
curl --location --request POST 'http://localhost:7896/iot/d?k=4jggokgpepnvsb2uv4s40d59ov&i=devbin10' \
--header 'Fiware-Service: {{service}}' \
--header 'Content-Type: application/json' \
--data-raw '{
	"lvl": "2395"
}
'
```

And retrieving the entities

```
curl --location --request GET 'http://localhost:1026/v2/entities' \
--header 'fiware-service: service' \
--header 'fiware-servicepath: /' \
| python -m json.tool
```

Another example, creating the device:

```
curl --location --request POST 'http://localhost:4041/iot/devices' \
--header 'fiware-service: service' \
--header 'fiware-servicepath: /' \
--header 'Content-Type: application/json' \
--data-raw '{
    "devices": [
        {
            "apikey": "4jggokgpepnvsb2uv4s40d59ov",
            "device_id": "devbin12",
            "entity_name": "devbin0012",
            "entity_type": "dev",
            "protocol": "PDI-IoTA-UltraLight",
            "transport": "MQTT",
            "expressionLanguage": "jexl",
            "attributes": [
                {
                    "object_id":"txt",
                    "name": "textMessage",
                    "type": "Text",
                    "expression": "txt|urlencode"
                }
            ]
        }
    ]
}'
```

Sending data:

```
curl --location --request POST 'http://localhost:7896/iot/d?k=4jggokgpepnvsb2uv4s40d59ov&i=devbin12' \
--header 'Fiware-Service: {{service}}' \
--header 'Content-Type: application/json' \
--data-raw '{
	"txt": "foobar{}"
}
'
```

### Expressions usage in commands

Provision

```JSON
curl --location --request POST 'http://localhost:4041/iot/devices' \
--header 'fiware-service: service' \
--header 'fiware-servicepath: /' \
--header 'Content-Type: application/json' \
--data-raw '{
    "devices": [
        {
            "apikey": "4jggokgpepnvsb2uv4s40d59ov",
            "device_id": "devbin11",
            "entity_name": "devbin0011",
            "entity_type": "dev",
            "protocol": "PDI-IoTA-UltraLight",
            "transport": "MQTT",
            "expressionLanguage": "jexl",
            "commands": [
                {
                    "name": "ring",
                    "type": "command",
                    "expression": "{setValue:0}"
                }
            ]
        }
    ]
}'
```

Trigger command

```
curl --location --request PUT 'http://localhost:1026/v2/entities/devbin0011/attrs/ring?type=dev' \
--header 'Content-Type: application/json' \
--header 'fiware-service: service' \
--header 'fiware-servicepath: /' \
--data-raw '{
      "type" : "command",
      "value" : "1"
}'
```


## Receiving Binary data

Hex mosquitto sub

```
mosquitto_sub -v -t '#' -F %X
```

Provision the device

```
{
    "devices": [
        {
            "apikey": "4jggokgpepnvsb2uv4s40d59ov",
            "device_id": "devbin5",
            "entity_name": "devbin005",
            "entity_type": "dev",
            "protocol": "PDI-IoTA-UltraLight",
            "transport": "MQTT",
            "expressionLanguage": "jexl",
            "attributes": [
                {
                    "object_id": "t",
                    "name": "temperature",
                    "type": "Text"
                },
                {
                    "name": "temperature_mod",
                    "type": "Text",
                    "expression": "t?('0x'+(t|slice(2,6)))|parseint|tostring:'NO DATA'"
                }
            ]
        }
    ]
}
```


Post hex data

```
echo -ne "\x45\x09\x1B\x50\x0D\x00" | mosquitto_pub -h localhost -t '/json/4jggokgpepnvsb2uv4s40d59ov/devbin5/attrs/t' -s
```

0x091b = 2331

```
echo -ne "\x45\x5F\x5F\x50\x0D\x00" | mosquitto_pub -h localhost -t '/json/4jggokgpepnvsb2uv4s40d59ov/devbin5/attrs/t' -s
```

0x5F5F = 24415

And retrieving the entities
```
curl --location --request GET 'http://localhost:1026/v2/entities' \
--header 'fiware-service: service' \
--header 'fiware-servicepath: /' \
| python -m json.tool
```
