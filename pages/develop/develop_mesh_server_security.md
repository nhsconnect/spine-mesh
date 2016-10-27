---
title: MESH Server Security 
keywords: development
tags: [development]
sidebar: overview_sidebar
permalink: develop_mesh_server_security.html
summary: An overview of the security and authentication model used by the MESH Server API.
---

## Authorization Header Usage ##

MESH increases the security of DTS exchanges. As with DTS each ‘client’ has a unique User ID and Password that is maintained via the MESH administration application. The User ID will be used as the Sender/ Recipient ID and the password will be encrypted using HMAC-SHA256 and placed in the authentication header. The authentication header also includes a single use nonce (or combination of nonce and nonce count) so that the value of the Authentication Code can be unique on each request to avoid the possibility of man-in-the-middle/replay attacks.

The following Request Header MUST be present in every HTTP Request:

```http
Authorization : NHSMESH {UserID}:{Nonce}:{NonceCount}:{Timestamp}:{HMAC-Sha256(UserID:Nonce:NonceCount: Password:Timestamp)}
```

{% include note.html content="NHSMESH is the name of the Custom Authentication Schema. The Authentication Header should prefix the generated authentication token with the name of this schema followed by a space character." %}

### Request Authentication Checks ###

The Server will perform the following checks to authenticate the request:

- Get the User ID from the Recipient/Sender ID
- Get the expected password from the MESH database
- Use HMAC-SHA256 on the User ID, Password and the Nonce and Nonce Count from the Request Header and check that the hashed values match. The Secret Key for the HMAC generation will be supplied by NHS Digital on request.
- That we have not already received this combination of User ID, Nonce and Nonce Count on a previous HTTP request.
- That we have not already received this combination of User ID, Nonce and Nonce Count on a previous HTTP request.
- That the Timestamp is within allowable bounds (e.g. ± 2 hour of current time to allow for BST/GMT & client/server clock differences).

### MESH Header Attributes ###

| Attribute | Purpose | Mandatory | Legacy | ITK Attribute |
|-----------|---------|-----------|--------|---------------|
| Authorization: | To authenticate sender | Yes | &nbsp; | &nbsp; |
| Mex-ClientVersion: | For audit purposes | No | &nbsp; | &nbsp; |
| Mex-OSArchitecture: | For audit purposes | No	| &nbsp; | &nbsp; |
| Mex-OSName: | For audit purposes | No | &nbsp; | &nbsp; |
| Mex-OSVersion: | For audit purposes | No | &nbsp; | &nbsp; |
| Mex-JavaVersion: | For audit purposes | No | &nbsp; | &nbsp; |
| Mex-From: | Sender of the message, a DTS address | Yes | &nbsp; | `itk:senderAddress` (assumes MESH address type) |
| Mex-To: | Recipient of the message, a DTS address | Yes | &nbsp; | `itk:addressList/itk:address[1]` (assumes MESH address type) |
| Mex-WorkflowID: | Identifies the type of message being sent e.g. Pathology, GP Capitation | Yes | &nbsp; | `itk:header/@service` |
| Mex-FileName: | The name of file | No | &nbsp; | &nbsp; |
| Mex-ProcessID: | For future use. Identifier to specify the type of processing that might be required before forwarding to the recipient. | No |  &nbsp; | &nbsp; |
| Mex-Content-Compress: | Flag to indicate that the client should compress the file, if the resultant size will be smaller than the original | No | Yes	| &nbsp; |
| Mex-Content-Encrypted: | Flag indicating that the original message is encrypted | No | Yes | `manifest/manifestitem/@encrypted` |
| Mex-Content-Compressed: | Flag indicating that the original message is compressed | No | Yes | `manifest/manifestitem/@compressed` |
| Mex-Content-Encoded: | Used when Mex-Content-Compressed is set to Yes	| No | No | `gzip` |
| Mex-Checksum: | Checksum of the original message contents	| No | Yes | &nbsp; |
| Mex-MessageType: | ‘Data’ or ‘Report’ | No | Yes | &nbsp; |
| Mex-LocalID: | Local unique identifier of the message | Yes | &nbsp; | &nbsp; |
| Mex-Subject: | Subject line to be used for SMTP messages | No | &nbsp; | &nbsp; |
| Mex-PartnerID: | Obsolete | No | Yes | &nbsp; |
| Mex-MessageID: | Identifies a message once it has first been uploaded to the MESH server | Yes |  &nbsp; |  &nbsp; |