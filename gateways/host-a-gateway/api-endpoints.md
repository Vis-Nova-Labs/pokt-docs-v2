# API Endpoints

The Gateway Server currently exposes all its API endpoints in form of HTTP endpoints.`x-api-key` is an api key set by the gateway operator to transmit internal private data

|                     |        |                                                                                            |             |                                          |
| ------------------- | ------ | ------------------------------------------------------------------------------------------ | ----------- | ---------------------------------------- |
| `/relay/{chain_id}` | ANY    | The main endpoint for your reverse proxy to send requests too                              | ANY         | `{chain_id}` - Network identifier        |
| `/metrics`          | GET    | Metadata on the gateway server performance for observability purposes                      | N/A         | N/A                                      |
| `/poktapp`          | POST   | Adds an existing app stake to the appstake database (not recommended due to security)      | `x-api-key` | `private_key` - private key of app stake |
| `/poktapp`          | DELETE | Removes an existing app stake from the appstake database (not recommended due to security) | `x-api-key` | `app_id` - id of the appstake            |
| `/poktapp`          | GET    | A list of all the available app stakes                                                     | `x-api-key` | N/A                                      |

\
