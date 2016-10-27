---
title: MESH Server Usage
keywords: development
tags: [development]
sidebar: overview_sidebar
permalink: develop_mesh_server_usage.html
summary: An overview of the operation and usage of the MESH Server API.
---

The following requirements must be followed in order to send and receive messages using MESH:

- MESH messages MUST be less than 100MB. Messages over 100MB will be rejected with an error. 
- Although MESH is a highly reliable service, message exchange clients are responsible for end to end reliability of exchanges.<sup>1</sup>

<sup>1</sup>This is similar to the old ‘Forward Express’ (Multi-hop end party reliability) messaging which similarly replaces responsibility for notifying the outcome of a transaction on the sender and recipient.

## Mailbox polling  ##

The basic pseudo code for a mailbox’s polling cycle is as follows:

```shell
Authenticate Mailbox
    For each file waiting to be sent:
        Send file to recipient
Fetch the list of messages in the Mailbox’s Inbox
    For each available message:
        Download the message
        Save the message to the file system.
        Acknowledge the successful receipt of the message
```

The sending and receiving of messages can be split into separate tasks which would make the pseudo code as follows:

### Sending messages ###

```shell
Authenticate Mailbox
For each file waiting to be sent:
    Send file to recipient
```

### Downloading messages ###

```shell
Authenticate Mailbox
Fetch the list of messages in the Mailbox’s Inbox
    For each available message:
        Download the message
        Save the message to the file system.
        Acknowledge the successful receipt of the message
```

## Typical MESH Message Exchange ##

The following sequence diagram shows a typical MESH message exchange:

![Mesh API Exchange Sequence Diagram](images/develop/mesh_exchange_sequence_diagram.jpg)
 
{% include important.html content="Every time a connection is made to the MESH service to upload or download messages, an Authenticate message MUST be called." %}
 
