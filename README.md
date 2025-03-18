# Protocol details for KDC

## TCP Packet structure

| Name | Length (Bytes) | Type | Description |
| :--- | :---           | :--- | :---        |
| Type | 1 | Enum | Specifies message type |
| Length | 4 | Int32 | Specfies message body length in bytes (maybe this should be higher?) |
| Body | As specified by Length | Bytes | Main message body |


## Client -> Server

#### Message types

##### Connect
Connects to the server
Mainly to make sure the protocol version is compatible and make sure the server is online

| Field name | Description |
| :---       | :---        |
| Version    | Specifies protocol version according to Semver |

##### Authenticate
Sends User information to tell the server to set us as online

| Field name | Description |
| :---       | :---        |
| Username   | Specifies previously registered username |
| Authentication secret | How authentication is to be done is TBD |


##### Fetch connected 
Send request to retreive all connected users

| Field name | Description |
| :---       | :---        |
    
##### Search query
Send request to send list of files matching the search query

| Field name | Description |
| :---       | :---        |
| Query      | Search string to match with file list |

##### Fetch user file list
Send request to send list of files hosted by a particular user who is online

| Field name | Description |
| :---       | :---        |
| User       | Username of user whose file list needs to be fetched |

##### Verify-client
Verify whether client that we are connecting to is the real client

| Field name | Description |
| :---       | :---        |
| TBD | TBD |

##### Disconnect
Tell the server we are going offline

| Field name | Description |
| :---       | :---        |

##### Post files
Send the server the list of files we want public

| Field name | Description |
| :---       | :---        |
| Files      | JSON formatted file tree |

##### Send message
Send a generic message to the server (this could be used for things that don't pertain to the core functioning of the system)
Maybe a message board like in the old DC++?

| Field name | Description |
| :---       | :---        |
| Message | JSON formatted message |

## Server -> CLient

##### Auth success 
Indicate authentication success and issue a token (?)

| Field name | Description |
| :---       | :---        |
| TBD (token?) | TBD |

##### Auth fail
Indicate authentication failure with reason

| Field name | Description |
| :---       | :---        |
| Reason     | State reason for failure |

##### Send connected
Send list of currently connected users 

| Field name | Description |
| :---       | :---        |
| Connected  | JSON formatted list of connected clients with their usernames and IPs |

##### Send file list
Send list of files based on previous request

| Field name | Description |
| :---       | :---        |
| File list | JSON formatted list of files including names (path), size, and user who is hosting it |

##### Verify-success
Response to [`Verify-client`](#####Verify-client)
Indicate success

| Field name | Description |
| :---       | :---        |

##### Verify-fail
Response to [`Verify-client`](#####Verify-client)
Indicate failure with reason

| Field name | Description |
| :---       | :---        |
| Reason     | State reason for failure |

##### Send-message
Send a generic message to the client (this could be used for things that don't pertain to the core functioning of the system)
Maybe announcements and notifications

| Field name | Description |
| :---       | :---        |
| Message    | JSON formatted message |

##### Disconnect
Tell the client that the server is disconnecting them with a reason (example: Server going offline)

| Field name | Description |
| :---       | :---        |
| Reason     | State reason for disconnection |


## Client <-> Client (TCP)

##### Connect
Initiate connection with peer
This should also exchange keys for encryption (Diffie-Hellman? PGP?)
After this, the body of all packets will be encrypted with this key

| Field name | Description |
| :---       | :---        |
| TBD | Fields need to handle verification. This will probably be multiple different packets |

##### Request-file
Request to download a file that the peer hosts

| Field name | Description |
| :---       | :---        |
| File-descriptor | Maybe a UUID of some kind to describe each file? |

##### Request-acknowledge
Accept request to download file
After this, a UDP connection is opened and the files are sent

| Field name | Description |
| :---       | :---        |
| Connection ID | ID that will later be referenced in the UDP connection |
| Number of packets | Number of packets that will be sent over the UDP connection |

##### Request-deny
Deny request to download file with reason

| Field name | Description |
| :---       | :---        |
| Reason | State reason for denial |

##### Send message
Send a generic message to the client (this could be used for things that don't pertain to the core functioning of the system)
Maybe p2p messaging

| Field name | Description |
| :---       | :---        |
| Message | JSON formatted message |

##### Disconnect
Tell the peer that the host peer is disconnecting with reason (example: inactivity, shutting down, out of ports)

| Field name | Description |
| :---       | :---        |
| Reason | State reason for disconnection |

##### File-Nack
This is sent while the host is receiving a previously requested file and it did not receive a specific part indicated by its Packet num

| Field name | Description |
| :---       | :---        |
| Packet num | Packet num of missing packet |



## UDP Packet structure

| Name | Length (Bytes) | Type | Description |
| :--- | :---           | :--- | :---        |
| Type | 1 | Enum | Specifies message type |
| Conn ID | 8 | Bytes | Specfies connection ID sent in [`Request-acknowledge`](#####Request-acknowledge) |
| Packet num | 8 | Int32 | Specifies the which packet this is in order |
| Len | 4 | Int16 | Specifies length of this packet |
| Body | As specified by Len | Bytes | Main message body |

## Client <-> Client (TCP)

##### EOF
This specifies the last packet in the file

##### File
This packet is part of the file but not the last packet
