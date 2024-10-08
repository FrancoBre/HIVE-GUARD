![logo](./assets/cortina.gif)

## Contents
1. [Introduction](#introduction)
2. [How to follow this docs](#how-to-follow-this-docs)
3. [System Architecture](#system-architecture)
4. [Detailed flow](#detailed-flow)
   1. [Streamer - Master Server connection](#streamer-master-server-connection)
   2. [Master Server register and frontend setup](#master-server-register-and-frontend-setup)
   3. [Frontend Hive Data querying](#frontend-hive-data-querying)
   4. [Alerts Polling](#alerts-polling)
6. [How to Setup](#how-to-setup)
7. [Troubleshooting](#troubleshooting)
8. [Things Left to Do](#things-left-to-do)
9. [Thanks](#thanks)

## Introduction

Are you a beekeeper? Do you struggle to keep control of your beehives and would like insights on what's going on, and possible actions to take, from your smartphone or laptop? Then our system is designed for you!

HiveGuard aims to improve beekeeping by leveraging modern technology to give beekeepers an edge in managing and monitoring their hives. This includes real-time video streaming, environmental sensing, machine learning for bee behavior analysis, and instant alerts - all integrated into an easy-to-use platform.

## How to follow this docs

You will find a general overview of the system here, and linked you will find the software and documentation for every component. We recommend you read this docs first, and then read the docs for the [Master Server](https://github.com/pablotrrs/hive-guard-master-server/blob/main/README.md), then the docs for the [Streamer](https://github.com/FrancoBre/hive-guard-streamer/blob/main/README.md), and finally the docs for the [Frontend](https://github.com/EvolutionRX/hive-guard-client/blob/main/README.md).

## System Architecture

The system is composed of three major components: 
![logo](./assets/infrastructure.png)

1. *Streamer*: An ESP32CAM along with a DHT11 module attached to a hive for recording video, temperature, and humidity. Attach the streamer to your hive. It records video, temperature, and humidity, and streams this data to the master server over a local network.

  
![explicacion-arq](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/0ca559d7-9a9d-422a-a7fd-98a013980be1)

  ![image](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/c59538b6-df63-48e7-8906-e9ef734ab370)

2. *Master Server*: A central server that processes data from the streamer. The master server uses machine learning to detect and analyze bee behaviors such as entering/leaving the hive and identifying ill bees. This server is intended to be run in a computer on the field where the hives are in, or in a raspberry pi. Its results are sent to the frontend.

![ezgif-1-4c515cab20](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/28130051-8dc3-4bdf-b987-1b47bde92535)

3. *Frontend*: A user interface that displays processed data and insights. The frontend generates charts and summaries, helping you make informed decisions such as when to place a pollen trap or identify potential hive issues. Alerts are sent to your email and displayed on the frontend.

![image](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/e4c0e126-a4ec-473e-b31e-4c9815d33901)
![image](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/abf80eae-534e-4f19-9a32-d41a7557da89)

## Detailed Flow

### Streamer - Master Server connection
This sequence diagram describes the initial connection and configuration process of the system. It starts with the beekeeper initiating the master server and ends with the streamer transmitting data to the master server.

![streamer-master-connection](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/e34a7e6a-8deb-4baa-a534-f314910a78d2)

The streamer sends the following data from itself to the master server:

 - `id`: Unique identifier for the ESP32 camera.
 - `wsPort`: WebSocket port.
 - `appPort`: Application port.
 - `display`: Display name for the camera.
 - `ip`: IP address of the streamer.

The ws and app (HTTP server) ports are generated dynamically, checking via a HTTP request if they're occupied or not. The UDP port is `12345` always and that's why it isn't sent. 

### Master Server register and frontend setup
This sequence diagram details how the beekeeper enters the master server's public URL in the frontend and how the initial parameters are configured and verified.

![frontend-config](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/51e39e56-e37f-4570-b203-e6a87e64a239)

The frontend sends the following threshold and email parameters configuration:

```json
{
  "TEMP_MIN_THRESHOLD": 20,
  "TEMP_MAX_THRESHOLD": 60,
  "HUM_THRESHOLD": 80,
  "EMAIL_USER": "test",
  "EMAIL_PASS": "test",
  "EMAIL_RECIPIENT": "test"
}
```

 - `EMAIL_RECIPIENT`: The beekeeper's email entered in the frontend, where emails will be sent.
 - `EMAIL_USER`: The email address from which emails will be sent.
 - `EMAIL_PASS`: The API key (e.g., Gmail) for that email address.

You can get a Gmail API key by following [this documentation](https://cloud.google.com/docs/authentication/api-keys).

### Frontend Hive Data querying
This sequence diagram shows how the beekeeper queries updated hive data and how the frontend generates charts with temperature and humidity data.

![frontend-views](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/5e7e06f7-c75a-40bf-b4e6-0031d430e531)

### Alerts Polling
This sequence diagram details how the frontend polls for alerts and how the master server sends email alerts to the beekeeper.

![frontend-alerts](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/8ca7aff5-66d9-47f1-9067-e9f4a480ea7d)

The following alerts can be generated:

 - Humidity exceeds the threshold.
 - Temperature exceeds the threshold.
 - Temperature falls below the threshold.

## How to Set Up

### Prerequisites
- **Hardware**: ESP32-CAM modules for streamers, DHT11 sensors for temperature and humidity, a computer or Raspberry Pi for the master server.
- **Software**: All software and detailed setup instructions are available as open source. You can find the Git projects as submodules of this repository.

You can try the system using the [mocks available in the master server](https://github.com/pablotrrs/hive-guard-master-server#Mock-Versions) to showcase some functionalities before deploying it to your hives.

To try the system on your hives:
1. *Initialize Master Server*: Start the master server first.
2. *Streamer Initialization*: Power on the ESP32CAM. It will launch an AP for WiFi credentials, connect to the network, and send a UDP broadcast message. The master server listens and starts a WebSocket connection.
3. *Save master server public URL*: The server's webpage (localhost:8000/client) displays bee images, temperature, humidity data, and both local and public IP addresses. Save the master server public URL.
4. *Frontend Connection*: A live version is available at this [link](https://hive-guard-client-production.up.railway.app/). If unavailable, follow the local setup instructions in the documentation. Enter the master server's public URL in the frontend. The frontend performs a health check and establishes a connection to display hive data. Then it will ask you for config params such as thresholds for the alerts.

Here's a step by step video on how to set up the sytem:
[![Watch the video](https://github.com/user-attachments/assets/15169c97-10f2-49bd-9511-7b2b79fb03ce)](https://youtu.be/CMT_JoFqt6s)

## Troubleshooting

Having trouble? Here are some troubleshooting tips:

- *Firewall Issues*: If you encounter connectivity problems, ensure any firewalls on the master server's device are disabled.
- *Connectivity*: Make sure all devices are on the same network and properly configured.

## Things Left to Do

Here are our next steps:

### Further Testing

We are not beekepers, and the beekeper who is helping us is not a programmer, so this system needs further testing under real life scenarios. If you're a beekeper and a programmer, any feedback and contributions would be invaluable!

![OIG2](https://github.com/FrancoBre/HIVE-GUARD/assets/66085255/9e78d34d-51f0-4847-9b2c-79475ac54fd3)

### Battery
We developed a version that allows battery-powered streamers and battery level monitoring from the streamers. The battery level is displayed by the master server on its webpage and also in the frontend. We even created alerts for when the battery level exceeds a threshold, similar to how we handle humidity and temperature alerts, both via email and live alerts in the frontend!

The problem is that we didn't have enough pins on the ESP32-CAM. We had to read a general-purpose pin on the ESP32-CAM only during startup. When the device starts, it reads from the pin; if there is voltage, it means battery level monitoring is enabled. We save the battery level, stream it, work normally for 10 minutes, and then reboot the device to read the battery level again.

This is problematic because the re-connection takes time, during which we might lose data.

So, we thought of adding an extra ESP8266 device to the ESP32-CAM to use its additional pins. We could also attach an infrared movement sensor to record actual bee movement and save battery.

You can find more information about this in the "Things Left to Do" section in the [master server](https://github.com/pablotrrs/hive-guard-master-server#What-is-left-to-do) and [streamer projects](https://github.com/FrancoBre/hive-guard-streamer#Battery).
   
### Network Optimization
Batch data transmission from streamers to optimize bandwidth usage and avoid congestion.

## Thanks

We would like to express our deep gratitude to:

- *Jorge Seniw and his family*: For letting us try the system, and helping us out with beekeeping related questions and issues. You can find him in this instagram account: [La miel de papá](https://www.instagram.com/lamiel.depapa?igsh=MWZzeG85d3R0cXhpMQ==)
- *[Fabian Hickert](https://github.com/BeeAlarmed/BeeAlarmed)*: For his outstanding work on the machine learning neural network, which we used for detecting bees and identifying illnesses.
- *[Nomadic Geek](https://www.youtube.com/@nomadicgeek_369)*: For his foundational [surveillance system](https://www.youtube.com/watch?v=WsPFQx0p4Us&list=PLFRIWPt6uUoVsirzwwqrNtp5vG-4ByIoe) that inspired our project.
- *Prof. Alexis Tcach*: For his guidance and support throughout the development of HiveGuard.

---

Thank you for choosing HiveGuard! Whether you are a beekeeper, a programmer, or both, your contributions and feedback are invaluable. Feel free to fork, contribute, or contact us. Happy beekeeping!
