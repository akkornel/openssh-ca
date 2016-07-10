=======================
OpenSSH-ca Architecture
=======================

OpenSSH-ca has four components: The *CA itself*, one or more *authentication proxies*, the *bastion host*, and the *orchestration server*.

As was mentioned in the readme, OpenSSH-ca is more than just the software whose documentation you are now reading.  OpenSSH-ca is a combination of the software *and the AWS environment to keep it (and your keys) safe*.

This architecture document has the following flow:

* First, we're going to talk about some of the higher-level aspects of the AWS environment that OpenSSH-ca needs.

* Next, we're going to actually talk about the four components of OpenSSH-ca, introducing additional parts of the environment as we go.

* Finally, we are going to talk about additional aspects of the data that OpenSSH-ca works with.  That is, the CA keys (public and private), the signed host keys (with accompanying metadata), the KRL file (again, with accompanying metadata), and ancillary keys (and, again, metadata!).

The AWS Environment
===================

First, a confirmation: By "the AWS environment", I am referring to an Amazon Web Services account that *is dedicated only to this service*.

The AWS environment starts with a single VPC with a /27 supernet.  
