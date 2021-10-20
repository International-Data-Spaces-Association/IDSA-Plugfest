# Local test scenario with 2 connectors

This folder contains an example deployment of two connectors with the same IDS IDs. This is only intended for test purposes, but allows you to experiment with the connector locally.

The information for the connectors:
- Connector 1
    - **UI URL**: http://localhost:8083
    - **Connector ID**: http://plugfest2021.08.localhost.demo
    - **Self Description Access URL**: http://core-container-connector-1:8080/selfdescription
- Connector 2
    - **UI URL**: http://localhost:8088
    - **Connector ID**: http://plugfest2021.08.localhost.demo
    - **Self Description Access URL**: http://core-container-connector-2:8080/selfdescription

> **Note**: The communication between the two core containers is routed internally within Docker. So other connectors won't be able to access the connector via the URL names. Also, the self description will contain these internal addresses only, so the access URL must be changed if an external connector wants to reach it (by replacing http://core-container-connector-1:8080 and http://core-container-connector-2:8080 in the access URL to respectively http://MACHINE_HOSTNAME:8080 and http://MACHINE_HOSTNAME:8085) 