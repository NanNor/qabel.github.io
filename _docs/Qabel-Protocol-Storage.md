---
title: Protocol Storage
---
# Storage Specification

## Abstract

A protocol for read, write and delete access to bulk (cloud) storage.

## Protocol

### URLs

The URL to access a Qabel Storage Volume can be split into four parts:
* the transport protocol: https and http are supported. https is the recommended protocol.
* the service address: Host and port specification of the service. In the following SERVER.
* the service prefix: This is a prefix for all Qabel related URLs on this server. This way to multiplex services sharing the same service address. In the following PREFIX.
* identifier of a Storage Volume (in the following PUBLIC) or "_new" for a creation request.

<a name="url"></a>
In [BNF](http://www.w3.org/Addressing/URL/5_BNF.html) [notation](http://www.w3.org/Notation.html) of the W3C:

`volumeurl ::= protocol "://" serviceaddress servicepath "/" public`

1. protocol ::= "https" | "http"
2. serviceaddress ::= serveraddress ( ":" serverport ) ?
   1. serveraddress ::= IPv4 | IPv6 | DNSName
   2. serverport ::= "1" - "65535"
3. servicepath ::= "/" [ URLChars, "/" ] *
4. public ::= *friendlybase64char
   1. friendlybase64char ::= [ "A" - "Z", "a" - "z", "0" - "9", "-", "_" ]

Example:
`https://qabelserver:8000/qabel-storage/8043810841`

### Create a new Qabel Storage Volume

* HTTP-Method: POST
* URL Example: https://example.com/qabel-storage/_new
* URL Scheme: http[s]?://[:SERVER:][:PREFIX:]/_new

This creates a new Qabel Storage Volume with it's own write- and revoke-tokens. These values are returned to the user by a JSON document sent back to this request:

```json
{
  "public": String,       // public token for read access
  "revoke_token": String, // private token for deleting the storage
  "token": String         // secret token to allow write access
}
```
The fields are described as follows:

* ```public```: the public token which is sent as part of the URL. If the public token is ```123456790```, the URL to access this Qabel Storage Volume will be ```http://server/prefix/1234567890```
* ```revoke_token```: this token is needed to delete a Qabel Storage Volume with all it's contents. This token should never be public or given to any untrustworthy authority. See section deletion.
* ```token```: this token is needed to do any updates on the Qabel Storage Volume. See section upload.

#### Return values

|HTTP status code|reason|
|:----------------:|------|
| 201 | Storage Volume successfully created (*Created*) |
| 503 | Failed to create volume, because of overloading. |


### Probe an existing Qabel Storage Volume

* HTTP-Method: GET
* URL Example: https://example.com/qabel-storage/1223456789/
* URL Scheme: http[s]?://[:SERVER:][:PREFIX:]/[:PUBLIC:]/

This method checks if the Qabel Storage Volume identified by the given PUBLIC
exists and is ready for use.

|HTTP status code|reason|
|:----------------:|------|
| 400 | Storage Volume ID is missing or syntactically invalid |
| 404 | Storage Volume ID does not exist |
| 200 | Storage Volume ID does exist |


### Uploading new Blobs

* HTTP-Method: POST or PUT
* URL Example: https://example.com/qabel-storage/1223456789/foobar
* URL Scheme: http[s]?://[:SERVER:][:PREFIX:]/[:PUBLIC:]/[:BLOBNAME:]
* Required HTTP header field: `X-Qabel-Token` containing the secret token

This method requires authorization through the token which is returned by the request described in "Create a new Qabel Storage Volume". The token has to be submitted within the HTTP header.
https is strongly encouraged (most servers should not accept http here anyway.)

The body of the http request will be saved to the blobname and can later be accessed via a get request under this URL (see below).

#### Return values

|HTTP status code|reason|
|:----------------:|------|
| 400 | Storage Volume ID is missing or syntactically invalid |
| 401 | No token has been submitted |
| 403 | Token is invalid |
| 404 | Storage Volume ID does not exist |
| 200 | Blob successfully uploaded |


### Retrieving a Blob

* HTTP-Method: GET
* URL Example: https://example.com/qabel-storage/1223456789/foobar
* URL Scheme: http[s]?://[:SERVER:][:PREFIX:]/[:PUBLIC:]/[:BLOBNAME:]

This method retrieves the blob BLOBNAME from the Qabel Storage Volume identified by PUBLIC.

#### Return values

|HTTP status code|reason|
|:----------------:|------|
| 400 | Storage Volume ID is missing or syntactically invalid |
| 404 | Storage Volume ID or blob does not exist |
| 200 | Blob successfully retrieved |


### Deleting a Qabel Storage Volume

* HTTP-Method: DELETE
* URL Example: https://example.com/qabel-storage/1223456789
* URL Scheme: http[s]?://[:SERVER:][:PREFIX:]/[:PUBLIC:]/
* Required HTTP header field: `X-Qabel-Token` containing the secret revoke_token

This method requires authorization through the revoke_token which is returned by the request described in "Create a new Qabel Storage Volume". The token has to be submitted within the HTTP header.
https is strongly encouraged (most servers should not accept http here anyway.)

#### Return values

|HTTP status code|reason|
|:----------------:|------|
| 400 | Storage Volume ID is missing or syntactically invalid |
| 401 | No REVOKE_TOKEN has been submitted |
| 403 | REVOKE_TOKEN is invalid |
| 404 | Storage Volume ID does not exist |
| 204 | Storage Volume successfully deleted (*No content*) |


### Deleting a Qabel Storage Blob

* HTTP-Method: DELETE
* URL Example: https://example.com/qabel-storage/1223456789/foo
* URL Scheme: http[s]?://[:SERVER:][:PREFIX:]/[:PUBLIC:]/[:BLOBNAME:]
* Required HTTP header field: `X-Qabel-Token` containing the secret token

This method requires authorization through the token which is returned by the request described in "Create a new Qabel Storage Volume". The token has to be submitted within the HTTP header.
https is strongly encouraged (most servers should not accept http here anyway.)

#### Return values

|HTTP status code|reason|
|:----------------:|------|
| 400 | Storage Volume ID is missing or syntactically invalid |
| 401 | No TOKEN has been submitted |
| 403 | TOKEN is invalid |
| 404 | Storage Volume ID or blob name does not exist |
| 204 | Storage Volume blob successfully deleted (*No content*) |
