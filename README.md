# ttn-tp-proxy-wemos

The http integration of [The Things Network](https://thethingsnetwork.com/) is limited in the sence that it does not allow to only forward the `payload_fieds` (  https://www.thethingsnetwork.org/docs/applications/http/#uplink), so you can pass it directly to [Thingspeak](https://thingspeak.com/). In order to fix this, I used a Wemos D1 to do this filtering/proxy between The Things Network and Thingspeak. I am sure this will work on other devices too, as long as they run the Arduino IDE and have some network connection.

This code can be compiled using http://platformio.org/

The `ArduinoJson` library ought to install automatically, but if not, install the `ArduinoJSon` libary:

```
pio lib install 64
```

Then put your wifi credentials in `lib/credentials/credentials.h`, compile and upload the code to a Wemos D1. You can test the functionality by navigating with your browser to http://$wemosip/

The code forwards only the `payload_fields` part of incomming json posts to `/ttn` to Thingspeak.

A simple test is to send some data to the Wemos. Make sure you have created a channel on Thingspeak first and have the *Write API Key* around.

```
read -p "Wemos D1 ip: " wemos_ip
read -p "Thingspeak API key: "  api_key
read -p "Value to post: " value

DATA=$(cat <<__EOT__
{ "foo" : "bar ",
  "payload_fields" : {
    "api_key" : "${api_key}",
    "field1" : "${value}"
  } }
__EOT__
)

curl \
  --header "Content-Type: application/json" \
  --request POST \
  --data "${DATA}" \
  http://${wemos_ip}/ttn

```

If this works, configure your home router to forward packages a port of your choice to port 80 of the Wemos. Last, configure the things network http integration to point to http://$yourip:$yourport/ttn

# OTA

The firmware also allows for over the air (OTA) updates. Adjust the `upload_port` in your `platformio.ini` file to match the ip of the Wemos.

# Troubleshooting

If you end up with "`400`" responses, the buffer for the json parsing might not be large enough. Check your JSON on http://arduinojson.org/assistant/ to get the size right for your settings.
