=========================================
OpenSSH-ca: A (hopefully)-safe OpenSSH CA
=========================================

What & Why
==========

OpenSSH-ca is here to help you use the certificate authority functionality that is built in to OpenSSH.

Why should you use this?  Simple...

**The *Here's a fingerprint.  Do you trust it?* question is something that you should never ever see.**  This software helps you to avoid that prompt, in a (hopefully) safe way.

For OpenSSH to work, both ends of the connection need to be authenticated.  Authenticating you is simple enough; that's what the password, public key, or Kerberos principal is for.  Authenticating the server, however, is harder.  Originally, the only method for server authentication was passing around pre-made `known_hosts` files.  Later, GSSAPI key-exchange was introduced as a way to use Kerberos for server authentication (not just client authentication), but that method has been kept out of upstream OpenSSH code.

OpenSSH now has a basic, single-level, certificate authority infrastructure.  A set of OpenSSH keys can be created and blessed as keys that trust other keys.  You give your SSH host keys to the OpenSSH CA, and the OpenSSH CA generates a set of signature files.  Those signatures are passed to the client during host authentication; if the client trusts the CA's keys, then host authentication is complete!

By having a single set of OpenSSH keys be so trusted, it makes your OpenSSH CA an especially juicy target.  Getting a copy of the CA's keys means you can pass your fake server as something else.  Normally OpenSSH would catch this, because your host's keys would not match the keys stored in the client's known_hosts file.  But, since an OpenSSH CA is being used, your fake server's SSH host keys are treated as valid.

This software forms just part of the solution.  It it meant to be deployed within an AWS environment designed to present a minimal external (and internal) attack surface.  AWS products are the only means of normal communication between the CA and the outside world.  Other documents in this repository will explain how the environment should be configured.

OK, What Do I Do?
=================

First, you should check out `docs/architecture.rst` to learn how everything is laid out.

That's all for now, though.  Sorry.  (Hopefully) More soon!
