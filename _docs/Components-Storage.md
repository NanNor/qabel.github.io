---
title: "Components: Storage"
---
# The storage component

## Abstract
The storage component is a core component of Qabel. It provides the functionalities for:

* writing to Qabel Storage Volume
* reading from Qabel Storage Volume
* deleting from Qabel Storage Volume

## Server and Protocol Design

### Paradigm

* Servers are universal and lightweight.
* Simple server via HTTP/REST.

## Encryption

The messages and direct meta data are encrypted by the client. The client has full control over the selection of ciphers. It can decide whether it should use symmetrical or asymmetrical encryption depending on the use case.

The content of the communication should not reach the outside. Blobs that are uploaded and stored are completely encrypted. The server cannot make assumptions over the structure or even the content.

In order to share uploaded data with other people, the uploading user has to provide the necessary
decryption information (e.g. a shared secret) for the recipients. However, this is expected to happen
outside of the storage protocol and is therefore not part of this documentation.

## Authentication

After [creating a new Storage Volume](../Qabel-Protocol-Storage/), the server provides three different tokens to regulate access:

* ```public```: A token to identify the Storage Volume on the Storage Server. This data can be safely published as it only allows read access to the encrypted information.
* ```token```: This token is stored by the server and all clients which are allowed to write to the Storage Volume. The server checks whether the token provided by the client matches the token of a Storage Volume. If both token match, the client is allowed to upload data to the Storage Volume.
* ```revoke_token```: This token is only stored by clients which are allowed to delete the Storage Volume. These clients are usually owners of the storage volume.

## More Things which have this module to do

This is only a list which have defined exactly

* The size of a blob has to define. All uploads have to have this size. This component fill it with dummy data.
* Storage module has to create the name of the blob and give it to the module which upload a blob
* Not for the first beta: Storage has to handle the webspace. It have to handle the order of new.
* Not for the first beta: Component has to handle different spaces if the user want it.
* Writeable only on its own storage. If other user allowed to edit data it has to upload on its own space.

## Sharing

### Example 1

Alice wants to share a file with Bob. Alice provides the public information and the cipher_key of this file to Bob through a third party (mostly, this will be the drop server). Bob can use this information to access the file read-only. If Bob wants to make changes to the file, he modifies his local copy and re-uploads the result to a new storage volume. Bob then grants access to Alice by sending her the public and the cipher_key of the newly created storage volume.
