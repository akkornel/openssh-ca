==================================
OpenSSH-ca Client/Key Request Flow
==================================

Introduction
------------

This document provides an overview of how a new client begins using OpenSSH-ca.

Client-Side, Part 1
-------------------

This document assumes that the OpenSSH-ca environment has already been
commissioned, and is ready for new enrollments, so all work must first take
place on the client.

A new client does not have any software or configuration specific to OpenSSH-ca.
The client will only have the OpenSSH software, plus its recently-generated
(and currently untrusted) host keys.  The bootstrap process involves six steps:

1. (*Optional*) Download and install an OpenSSH-ca enrollment client.

1. Download the bootstrap configuration file and public key.

1. Download and install the OpenSSH-ca KRL update agent.

1. Create and submit an enrollment request.

1. Wait for signatures to be issued.

1. Download and install the public keys.

Client Bootstrap
~~~~~~~~~~~~~~~~

The process begins by downloading the bootstrap key and configuration file onto
the client.  The bootstrap configuration file contains pointers to the other
configuration files, as well as the keys used to sign those files.  The
bootstrap configuration file includes the following information:

A. The name of the OpenSSH-ca environment.

A. The URL where the OpenSSH-ca's known_hosts file will be found.

A. The public key used to sign the known_hosts file.

A. The URL where the OpenSSH-ca enrollment client can find the list of
enrollment servers.

A. The public key used to sign the list of enrollment servers.

A. The URL where the OpenSSH-ca's KRL update agent can find its configuration
file.

A. The public key used to sign the KRL update agent's configuration file.

A. The public key used to sign enrollment server configuration files.

The bootstrap public key is used to sign the bootstrap configuration file.  The
bootstrap public key is one of the two most critical pieces of OpenSSH-ca (the
other is the OpenSSH public keys), because all of the OpenSSH-ca agents rely on
the bootstrap configuration file in order to work.

If an OpenSSH-ca enrollment client is being used, that client can handle
downloading the bootstrap configuration file, as long as it is given the
configuration file URL, and the bootstrap public key.

KRL Monitoring
~~~~~~~~~~~~~~

Once the bootstrap configuration file and key are installed, the OpenSSH-ca KRL
update agent should be started.  The agent will, on a regular schedule,
download the latest KRL (if a new one is available), verify its authenticity,
confirm that OpenSSH can understand it, and then put it into place on the
server.

The bootstrap configuration file contains a URL to the KRL agent's configuration
file, which contains all of the information necessary for the agent to work.
This includes the following:

A. The URL to download the KRL.

A. The public key used to sign the KRL.

A. A known-good public key, for KRL testing.

A. A known-revoked public key, for KRL testing.

The bootstrap configuration file contains the public key used to verify the KRL
agent's configuration file.

The OpenSSH-ca KRL update agent is the only OpenSSH-ca software that needs to
remain active on the server, so that it can keep the KRL updated.  However, if
you are using a system management service like Puppet, Chef, or Salt, you may
wish to just have the KRL update agent running on your management servers.

Client Enrollment Request
~~~~~~~~~~~~~~~~~~~~~~~~~

Once the bootstrap is complete and the KRL is being kept up-to-date, the client
may now submit its enrollment request.  The new request starts out with the
following fields:

I. The system's hostname and (if different) fully-qualified domain name.

I. The system's OpenSSH public keys.

I. A randomly-generated nonce.

I. Date and time the request was generated.

At this point, the request does not contain any authenticating information,
because there are multiple ways for authenticating a request, and each
environment will make their own choice as to how to authenticate.

Once the request is submitted, the client gets an enrollment token, and will
begin to regularly query the enrollment agent to see if the enrollment is
complete.

Enrollment Server
-----------------

The enrollment agent has the task of authenticating the host, adding additional
information to the request, and sending the updated request to the OpenSSH-ca
CA.

The OpenSSH-ca environment includes a enrollment server configuration file,
which includes

A. The Amazon SNS endpoint URL to use.

A. The SNS ARN where requests should be sent.

A. The SQS queue the enrollment server should monitor for completed enrollments.

The enrollment server configuration file is signed using the public key provided
in the bootstrap configuration file.

Each enrollment server also has their own key pair and 

Once the enrollment server has authenticated the requestor, and verified that
the request is authorized, the following information is added to the request:

I. The ID of the enrollment server that received the request.

I. The method used to authenticate the client.

I. The unique ID of the client (which will vary depending on the authentication
method used).

I. Date and time the request was authenticated.

For example, if the authentication method was GSSAPI over SSH, using the
client's host principal, the authentication method will be `GSSAPI+http` and
the unique ID will be `host/system_fqdn@realm_name`.

The updated request is then signed by the enrollment server's enrollment key,
and published to the appropriate SNS topic, using the configured endpoint and
ARN.  The enrollment request is small enough to fit directly within the
notification.

OpenSSH-ca CA
-------------

The enrollment notification arrives at the OpenSSH-ca environment, where it is
sent to two subscribers: A lambda function, and an SQS queue.  The queue
contains all pending enrollment requests to be processed by the CA.

The Lambda Function
~~~~~~~~~~~~~~~~~~~

The Lambda function does not actually read the message, it simply ensures that
the CA is running.  The CA normally shuts down when there is no work to be
done, in order to conserve resources and minimize attack surface availability.

Request Validation
~~~~~~~~~~~~~~~~~~

Once the CA is running, it receives the request from the queue and performs
validation.  The enrollment server's signature is verified, and the request is
checked to make sure it conforms to schema.  If the request appears authentic
and valid, the following information is added to the request:

I. The SQS message ID.

I. Date and time the request was received.

The updated request is placed into a local queue directory for processing by the
request preparer, and the preparer is signalled that a new request is
available. This is the last time the request will exist as a single file.

Request Preparation
~~~~~~~~~~~~~~~~~~~

The request preparer takes the 

CA Signer
~~~~~~~~~

The CA 

Once 



Client-Side, Part 2
-------------------




Once client enrollment is complete, and assuming that the host keys will never
change, the OpenSSH-ca enrollment client should be uninstalled.

OpenSSH Configuration
~~~~~~~~~~~~~~~~~~~~~

Once the OpenSSH-ca KRL update agent is running, and the client enrollment is
complete, OpenSSH's server configuration must be updated to incorporate the new
public key and KRL files.  The system-wide known_hosts file must also be
updated to trust the CA's public keys, or the OpenSSH-ca known_hosts update
agent can be used to keep the files up-to-date.