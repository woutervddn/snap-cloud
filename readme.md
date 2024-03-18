# SNAP - Sensor Network Along Perimiter

S.N.A.P. or SNAP is a hackathon project make during the [#inno4def](https://inno4def.be/) hackathon of 15-17th March 2024.

**Please note that this code is just a prototype made in 24hours, some things are hardcoded, some things are not working, most things are written poorly.**

SNAP uses novel sensor nodes that leverage an ESP32 microcontroller and one or more sensors like PIR, Seismic/Vibration,... 

After a detection, the camera on the same or another node gets triggered. The trigger info and/or the camera image get then send over LoRa to one of the gateways where it gets uploaded to the cloud.

This repository houses the cloud part of that code.

# Installation

Installing this code requires `docker` and `docker compose` or `docker-compose` to be installed.

Launching the entire thing is as simple as `docker compose up -d`. Please note that each of the folders need to have the right data in them. This might require credentials that are not listed here.

# Development

Development can be easily done by restarting one or more of the individual containers and rebuilding them in the process by using: `docker compose up SERVICE_NAME --build -d`

# Stopping

Stopping one service is done using `docker compose stop SERVICE_NAME` or running `docker compose stop` to stop all services.

# Debugging

Debugging can be done by viewing the logs for a specific service by using: `docker compose logs SERVICE_NAME` adding ` -f` will let you follow along live while these logs are generated.

# Configuration

You can configure ports, data directories and more my editing the `docker-compose.yaml` file or the relevant files in the sub folders.

# Extra

For more info, please check te relevant subfolders.

Access n8n by using:
- user: wouter@vierpuntnul.be
- pass: SnapSnap1