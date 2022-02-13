# Orisec2MQTT
My first Node-RED flow, which has been running reliably for a few months. The primary purpose of this flow was to provide real-time zone state updates (PIR movement detected; door open) from an [Orisec](https://www.orisec.co.uk/) alarm panel to Home Assistant via [MQTT discovery](https://www.home-assistant.io/docs/mqtt/discovery/). Orisec alarm panels are becoming increasingly popular in the UK, and the inspiration for this flow was to mimic the functionality of the popular [Texecom2MQTT](https://github.com/dchesterton/texecom2mqtt-hassio) integration for Home Assistant by emulating the Orisec Android app.

**Note** - I have not attempted to establish *how* the Android app constructs the UDP payload requests to the alarm panel. Instead I am simply sending the same UDP payloads via Node-RED and interpreting the response messages (see Config below).

## To-Do
- Panel States (armed/disarmed)
- Full armed/disarmed/night-armed commands
- Improved documentation

## Prerequisites
- [Home Assistant](https://www.home-assistant.io/)
- [MQTT integration](https://www.home-assistant.io/integrations/mqtt)
- [Node-RED addon](https://github.com/hassio-addons/addon-node-red) for Home Assistant
- Orisec controlPlus Android App (iOS untested)
- Packet analyser software (eg [Wireshark](https://www.wireshark.org/))
- [MQTT explorer](https://mqtt-explorer.com/) - useful rather than essential

## Config/UDP requests
The following requests occur in sequence:
- Initiate Connection (PIN sent to panel)
- Fetch panel serial
- Fetch supporting info (model no, etc.)
- Fetch Zone names (see below)
- Fetch Zone Config (MQTT [binary sensor](https://www.home-assistant.io/docs/mqtt/discovery/#motion-detection-binary-sensor))
- Fetch Zones States (ON/OFF - repeat every 4 secs)

Each request has a unique payload which is defined as an environment variable against the flow (for ease of admin). These can be obtained by analysing the UDP packets sent between from a device running the Android app to the Alarm panel and are sent as a hex string.

## Zones explained
Zones names are fetched in batches of 10. My alarm system contains 13 zones (combination of PIRs, PAs, Door/window sensors), therefore I am sending two requests (i) Zones 1-10, (ii) Zones 11-20, and adding these to a zone array. In theory this could be extended for 20+ zones but I have no way to test this. A single request is subsequently made to fetch states for *all* zones.

