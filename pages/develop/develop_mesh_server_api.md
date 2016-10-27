---
title: MESH Server API
keywords: development
tags: [development]
sidebar: overview_sidebar
permalink: develop_mesh_server_api.html
summary: An overview of the HTTP interface available as part of the MESH Server API.
---

## Check user authentication ##

The Check User Authentication message attempts to connect to a specific mailbox. This allows the user to ensure that their authentication is correct and will update the details of the connection history held for the mailbox. It can be considered similar to a keep-alive or a ping message in that it allows monitoring on the Spine to be aware of the ongoing utilisation of the inbox despite a lack of traffic. 
This message should be the first message sent in a ‘polling’ period for a mailbox.  This allows the client to check that the MESH server can be contacted and the mailbox is active before attempting to send/receive messages.

| Part | Value |
|------|-------|
| URL | `/messageexchange/{mailboxId}` |
| HTTP Action | `POST` |
| Request Headers | Authorization: {Authentication Headers}<br/>Mex-ClientVersion: {Client Version Number}<br/>Mex-OSArchitecture: {Operating System Architecture}<br/>Mex-OSName: {Operating System Name}<br/>Mex-OSVersion: {Operating System Version}<br/>Mex-JavaVersion: {JVM Version Number} |
| Response Code | 200: Ok<br/>403: Authentication Failed |
| Results | If the authentication is successful then the Connection History on the Mailbox will be updated.|

### Example Request ###

> POST https://10.210.162.216/messageexchange/NONFUNC01

```http
Connection: keep-alive
Mex-JavaVersion: 1.7.0_60
Mex-ClientVersion: Alpha.0.0.1
Mex-OSVersion: 6.1
Mex-OSArchitecture: Windows 7
Mex-OSName: x86
Authorization: NONFUNC01:jt81ti68rlvta7379p3ng949rv:1:201511201038:291259e6a73cde3278f99cd5bd3c7ec9b3d1d5479077a8f711bddf58073d5555
Content-Type: application/x-www-form-urlencoded
```

## Send message ##

This command uploads a message to the Message Exchange Server for onward delivery. The message is POSTed to the Sender's Virtual Outbox on the Spine. Meta-information is supplied in the request headers and the content of the message is supplied in the request body. Messages will be kept for five days before being deleted at which point a ‘message non-collected message’ is sent to the sender for information. Implementation of the MESH Server API must be include downloading and processing of non-collected reports and performing appropriate business processes to acknowledgement the delivery failure.

If senders have not received an acknowledgement within three days they MAY assume a failed delivery and consider re-sending. However the Spine does not determine fixed contract properties for MESH exchanges and users are free to agree alternative, more stringent, reliable exchange algorithms to suit their scenarios.

| Part | Value |
|------|-------|
| URL | `/messageexchange/{senders mailbox ID}/outbox` |
| HTTP Action | `POST` |
| HTTP Request Headers | Authorization: {Authentication Headers}<br/>Content-Type: application/octet-stream<br/>Mex-From: {senders mailbox ID}<br/>Mex-To: {recipient mailbox ID}<sup>1</sup><br/>Mex-WorkflowID: {DTS Workflow ID}<br/>Mex-FileName: {Original File Name}<br/>Mex-LocalID: {Local unique identifier of the message} |
| Optional HTTP Request Headers | Mex-ProcessID: {DTS Process ID}<br/>Mex-Content-Encrypted: { Flag indicating that the original message is encrypted }<br/>Mex-Content-Compressed: { Flag indicating that the original message is compressed }<br/>Mex-Content-Encoded: {Used when Flag to indicate file is compressed, set to ‘gzip’}.Currently MESH only accepts this form of compression.<br/>Mex-Subject : {Subject line to be used for SMTP messages} |
| Request Body | The binary contents of the message which is being uploaded. |
| Response Code	| 202: Accepted<br/>403: Authentication Failure<br/>417: Invalid Recipient |
| Response Body | JSON which includes the Message ID of the newly created message record.<br/>{"messageID": allocatedMessageID}<br/>In the Case of a Validation Failure the JSON will contain additional elements which will provide information about the cause of the error:<br/>{"messageID": allocatedMessageID,<br/>"errorEvent": statusEvent,<br/>"errorCode" : statusCode,<br/>"errorDescription" : statusDescription}<br/>The errorEvent, errorCode & errorDescription entries will contain values as per the DTS Status Values in the DTS Client Interface Specification document. |
| Results | A new record is created in the messageExchangeRecord bucket which contains the meta-information about the message.<br/>The Trading Summary Information for the Sending Mailbox is updated on the Spine so that the counts of the number of messages and the total message size transferred are updated. The ID of the newly created message is added to the inbox of the intended recipient mailbox |

<sup>1</sup>The sender attribute is cross-checked with the authentication token.

{% include note.html content="The `WorkflowID` is semantically similar, though not identical to and `InteractionID`. Rather it represents a set of messages under a single heading. This field is also expected to support Interaction IDs and other message type identifiers in the future." %}

### Example Request ###

> POST https://10.210.162.216/messageexchange/NONFUNC01/outbox

POST data:

```code
This is a test file
I am typing this so I have a file to send via MESH
```

Request Headers:

```http
Connection: keep-alive
Authorization: NONFUNC01:jt81ti68rlvta7379p3ng949rv:8:201511201038:e1672428d9f02e6d14b686bb7953e065eaf7bb146817e9016cf5974612953f5f
Content-Type: application/octet-stream
Mex-From: NONFUNC01
Mex-To: NONFUNC02
Mex-WorkflowID: 2015_11_12_1543
Mex-Content-Compress: N
Mex-Content-Encrypted: N
Mex-Content-Compressed: Y
Mex-Content-Encoded: ‘gzip’
Mex-MessageType: Data
Mex-LocalID: TESTING01
Mex-Subject: TEST TRANSFER 01
```

## Check inbox ##

This function checks if there are any messages that have been sent to a specific recipient mailbox and are ready to be downloaded. Client systems MUST poll their assigned inbox a minimum or once a day and a maximum of once every five minutes for messages. 
Any messages that are identified SHOULD be downloaded immediately. However, clients may choose to throttle messages or may be required to distribute processing time across a number of registered MESH mailboxes.
Only a list of the first 500 messages is returned. Therefore recipients SHOULD process the first 500 messages in their inbox prior to attempting to view the next awaiting messages.  This process should be repeated until all messages have been downloaded and acknowledged.

| Part | Value |
|------|-------|
| URL | `/messageexchange/{recipient}/inbox` |
| HTTP Action | `GET` |
| HTTP Request Headers | Authorization: {Authentication Headers} |
| Response Code | 200: Ok<b/>403: Authentication Failure |
| Response Body | JSON containing the list of Message IDs in the recipient's inbox. (List is limited to the 1st 500 messages) |
| Results | No changes are made to the database by this action. |

### Example Request ###

> GET https://10.210.162.216/messageexchange/NONFUNC01/inbox

Request Headers:

```http
Connection: keep-alive
Authorization: NONFUNC01:jt81ti68rlvta7379p3ng949rv:2:201511201038:2ba994afb852c7e0d49593f454ad1f8705f5729454958955fe188f191442ea79
```

## Download message ##

Download a message which has been sent to a mailbox.

{% include note.html content="Messages available for download may include non-collected reports.  These “non-collected reports” MUST be presented to system users to highlight where messages have not been collected by the intended recipient.  Such reports will assist users in identifying an issue with message receipt and prompt investigation into why the specific messages have not been collected. Failure to present these messages to system users represents a clinical risk." %}

| Part | Value |
|------|-------|
| URL | `/messageexchange/{recipient}/inbox/{messageID}` |
| HTTP Action | `GET` |
| HTTP Request Headers | {Authentication Headers}<br/>Accept-Encoding: (optional) 'gzip' if client can accept & decompress messages in GZip format. |
| HTTP Response Code | 200: Ok<br/>403: Authentication Failure<br/>404: Message does not exist<br/>410: Gone, message has already been downloaded. This error can be generated when requesting a message that has already been downloaded or has expired and deleted before the download is requested. In either case, the message is no longer available. |
| HTTP Response Headers | Content-Type: application/octet-stream<br/>Content-Encoding: gzip if message was compressed on upload and can be accepted by the recipient.<br/>Mex-From: {senders mailbox ID}<br/>Mex-To: {recipient identifier}<br/>Mex-WorkflowID: {DTS Workflow ID}<br/>Mex-MessageID: {DTS Message ID}<br/>Mex-FileName: {Original File Name}<br/> |	
| Response Body	| The binary contents of the message which is being downloaded. This will be empty if the message is a Report rather than Data Results	No changes are made to the database by this action. |

### Example Request ###

> GET https://10.210.162.216/messageexchange/NONFUNC01/inbox/20151120103837238795_386EAF_1429036893

Request Headers:

```http
Connection: keep-alive
Authorization: NONFUNC01:jt81ti68rlvta7379p3ng949rv:6:201511201038:a764f7b9d0f9aab1ddfb4d9fe258ef63bc547094044408ae4a4dc0f036f06bae
```

## Acknowledge message download ##

This message MUST be sent by the recipient to indicate that a message has been downloaded and saved correctly. This changes the status of the message and removes it from the recipient's inbox. Note that this acknowledgement closes the transaction on the Spine but does not result in an associated acknowledgement message to the sending system.

Senders will receive notification of undelivered messages after five days. Senders MAY choose to check on the delivery status of a message in the meantime using the tracing API .

| Part | Value |
|------|-------|
| URL | `/messageexchange/{recipient}/inbox/{messageID}/status/acknowledged` |
| HTTP Action | `PUT` |
| Request Headers | {Authentication Headers} |
| Response Code	| 200: Ok<br/>403: Authentication Failure |

### Example Request ###

> PUT https://10.210.162.216/messageexchange/NONFUNC01/inbox/20151120103837238795_386EAF_1429036893/status/acknowledged

PUT data:

Request Headers:

```http
Connection: keep-alive
Authorization: NONFUNC01:jt81ti68rlvta7379p3ng949rv:7:201511201038:e0cc8fb33674c75da29f1cace36294974e3b748eda1b75561c5cd4bd89a37d09
```