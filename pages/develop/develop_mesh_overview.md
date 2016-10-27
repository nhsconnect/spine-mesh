---
title: MESH Overview
keywords: development
tags: [development]
sidebar: overview_sidebar
permalink: develop_mesh_overview.html
summary: An overview of the MESH component of Spine.
---

## Overview ##

The MESH component of the Spine allows messages and files to be delivered to registered recipients via a java client. 
Users register for a mailbox and install the client. The MESH client manages the sending of messages which users (typically user systems) have placed in an outbox directory on their machine. It similarly manages the downloading of messages and files which other users have placed in the user’s virtual inbox on the Spine.

The MESH server has been designed with an underlying HTTP based protocol. This protocol is capable of being used directly where user systems wish to bypass the MESH client or where they want to construct their own clients.  Therefore, the MESH service will allow direct ‘store and retrieve’ messaging between endpoints as a new asynchronous pattern for Spine consumers.
MESH uses a RESTful paradigm and thus messages which are sent via mesh are POSTed to a MESH virtual outbox and recipients retrieve messages through a GET to their virtual inbox. The following document summarises the MESH HTTP API which may be used by clients directly or over which other SOAP or other APIs may be overlaid. 

## Security ##

To use the MESH Server API, the calling system will require a client certificate to connect to the MESH server using Secure Socket Layer (SSL) Mutual Authentication.   

Two types of client certificate can be used to connect to the MESH Server:

- If services currently connect to the Spine Messaging interfaces using an End Point Registration (EPR) certificate, this certificate can also be used for connection by the MESH client.   
- For users that currently do not use an EPR certificate, a MESH-specific certificate will be required.  These will be issued by NHS Digital’s Deployment Issue and Resolution (DIR) team.

For the MESH certificate, the Fully Qualified Domain Name (FQDN) will be based on the Organisation Data Service (ODS) code of the end system. It will follow the optionalLocalIdentifier.ODScode.api.mesh-client.nhs.uk naming convention.
Further details on the certificate request process can be found on the NHS Digital website.

### MESH mailboxes ###

All DTS user mailboxes and passwords were transferred to MESH at the end of January 2016.   If additional or new mailboxes are required, a Service Request should be raised via the National Service Desk.
