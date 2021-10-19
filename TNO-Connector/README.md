# TNO Connector - Plugfest Q3 Hands-on

This repository contains information for the hands-on session during the IDSA Plugest Q3-2021.

## Requirements

For the TNO connector to run you'll need:
- Docker
- Docker Compose
- Postman to interact with the connector (if you don't use the included UI)
- A valid IDS certificate (a valid localhost certificate is configured by default)
- Clone this repository on your local machine

## Configuration

The configuration of the TNO Connector is located in the `application.yaml` file.

The default configuration is ready to go and contains the localhost certificate.

If you'd want to use you own IDS certificate, valid in the AISEC DAPS, you can configure this by modifying the following lines in `application.yaml`:
- Line 23: Base64 encoded PEM* string of your certificate. Should start with: `LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t`
- Line 24: Base64 encoded PEM* string of your private key. Should start with: `LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0t`
- Line 2: The IDS identifier corresponding to the configured key


> PEM Certificate in the form of:
>```
>-----BEGIN CERTIFICATE-----
>...
>-----END CERTIFICATE-----
>```
>And PEM Private Key in the form of:
>```
>-----BEGIN PRIVATE KEY-----
>...
>-----END PRIVATE KEY-----
>```
>

## Deployment

Deployment of the Connector can be done via Docker Compose, by executing in the current directory:

```
docker-compose up -d
```

## Usage

Interacting with the connector can be done either via the included web UI or via a Postman collection.

### User Interface

Navigate to http://localhost:8083 to open the user interface.

Follow the steps below to retrieve an artifact from the hosted TNO Connector:
1) **Retrieve remote selfdescription**: Go to [_Self Description -> Request_](http://localhost:8083/#/selfdescription/request). And make sure the _Connector ID_ is set to `http://tnoscsn.demo` and _Access URL_ is set to `https://scsn.ids.smart-connected.nl/selfdescription`. And click _Request description_.
2) In the parsed view of the self description, click on the first catalog. And after that, click on the lock icon under _Actions_. This will bring you to the _Artifact Handler -> Consumer_ page.
3) The form under _Request Contract_ should now be filled with the relevant information. Click _Request Contract_ to start the Contract Negotiation sequence.
3) After the successfull contract negotiation, the _Transfer Contract_ field should contain an identifier. If this is the case, click _Request artifact_ to download the artifact to your local machine.

After following the steps, take a look around in the UI to explore the possibilities of the connector.


### Postman Collection
Download & import the [Postman Collection](Plugfest%20Connector%20Localhost.postman_collection.json). The Postman Collection contains Tests to automatically extract relevant information from the responses and store them in Collection variables.

> _NOTE: When exposing port 80 to a different port on your host machine then 8083, please modify the `API_URL` Collection variable from the Postman Collection accordingly._

The Postman collection contains the following basic requests:
- **Retrieve remote selfdescription** (`/api/description`): To fetch the self-description of the hosted TNO Connector. Contains a test to get the first offered resource as variable.
- **Retrieve contract offer** (`/api/description`): To fetch a specific element (`requestedElement`) from the self-description, in this case of the first offered resource. Puts the first contract offer in a variable.
- **Request Contract** (`/api/artifacts/consumer/contractRequest`): Starts a negotiation process with the remote connector based on a given contract offer (that has been retrieved from the self-description). Stores the Contract Agreement ID as variable.
- **Retrieve artifact** (`/api/artifacts/consumer/artifact`): Retrieves the given artifact with an references to the agreed upon contract.

All of the requests have the following parameters:
- `connectorId`: The IDS ID of the remote connector (required)
- `accessUrl`: The access URL of the remote connector (required)
- `agentId`: The Agent ID of the organization behind the remote connector (optional).

## Note

This uses an early build of the new TNO Connector, which will be made open-source later this year. Full documentation and API descriptions will follow when it's made open-source. 

## Contact

Contact [Maarten Kollenstart](mailto:maarten.kollenstart@tno.nl?subject=IDSA%20Hands%20On%20Session%202021) if you've any questions or want more information.