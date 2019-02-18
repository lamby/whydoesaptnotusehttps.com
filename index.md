---
layout: default
title: Why does APT not use HTTPS?
subtitle: "\"Isn't it more secure...?\""
description: Why does APT in Debian/Ubuntu, etc. not use SSL for package downloads?
---
{: #disclaimer}
<small>
(This site represents the status quo as it was some time ago,
particularly before [CVE-2019-3462](https://lists.debian.org/debian-security-announce/2019/msg00010.html). It does not represent my personal opinion nor that of Debian/Ubuntu.)
</small>

## tl;dr

https is used to prevent intruders from being able to listen to
communications between you and websites you visit, as well as to avoid
data being modified without your knowledge.

However, files obtained by APT are accompanied by their own signature
which allows your system to check they originated from your
distribution.

These signatures are checked against a small set of trusted keys
already stored on your computer. Downloaded files are rejected by APT
if they are signed by an unknown key[^apt-unknown-key] or are missing
valid signatures. This ensures that the packages you are installing
were authorised by your distribution and have not been modified or
replaced since.

HTTPS can not detect if malicious tampering has occurred on the disks
of the server you are downloading from. There is little point
"securely" transfering a compromised package.

## But what about privacy?

HTTPS does not provide meaningful privacy for obtaining packages.  As
an eavesdropper can usually see which hosts you are contacting, if you
connect to your distribution's mirror network it would be fairly
obvious that you are downloading updates.

Furthermore, even over an encrypted connection it is not difficult to
figure out which files you are downloading based on the size of the
transfer[^tor]. HTTPS would therefore only be useful for downloading
from a server that also offers other packages of similar or identical
size.

What's more important is _not_ that your connection is
encrypted but that the files you are installing haven't been modified.

## Overly trusting CAs

There are over 400 "Certificate Authorities" who may issue
certificates for any domain. Many have poor security records and some
are even explicitly controlled by governments[^ca].

This means that HTTPS provides little-to-no protection against a
targeted attack on your distribution's mirror network. You could limit
the set of valid certificates that APT would accept, but that would be
be error-prone and unlikely worth the additional hassle over the
existing public-key scheme.

## Why not provide HTTPS anyway?

Your distribution could cryptographically sign the files using the
existing scheme and additionally serve the files over HTTPS to
provide "defense in depth."

However, providing a huge worldwide mirror network available over
SSL is not only a complicated engineering task (requiring the
secure exchange and storage of private keys), it implies a
misleading level of security and privacy to end-users as described
above.

A switch to HTTPS would also mean you could not take advantage of
local proxy servers for speeding up access and would additionally
prohibit many kinds of peer-to-peer mirroring where files are
stored on servers not controlled directly by your distribution.
This would disproportionately affect users in remote locales.

## Ah, what about replay attacks?

One issue with a naïve signing mechanism is that it does not guarantee
that you are seeing the most up-to-date version of the archive.

This can lead to a _replay attack_ where an attacker substitutes an
archive with an earlier---unmodified---version of the archive. This
would prevent APT from noticing new security updates which they could
then exploit.

To mitigate this problem, APT archives includes a timestamp after
which all the files are considered stale[^valid-until].

## Where can I find out more?

More technical details may be found on the
_[SecureAPT](https://wiki.debian.org/SecureApt)_ wiki page.

### Footnotes

[^apt-unknown-key]: This appears as <tt>Release: The following
    signatures couldn't be verified because the public key is not
    available</tt>.

[^tor]: This may even be the case if using Tor via (for example)
    [apt-transport-tor](https://retout.co.uk/blog/2014/07/21/apt-transport-tor).

[^ca]: See, for example, _[What trusted root certification authorities
    should I trust?](https://security.stackexchange.com/questions/53117/what-trusted-root-certification-authorities-should-i-trust)_
    on StackOverflow.

[^valid-until]: See the _[Date,
    Valid-Until](https://wiki.debian.org/DebianRepository/Format#Date.2C_Valid-Until)_
    section of the <tt>DebianRepository</tt> page on the Debian Wiki.
