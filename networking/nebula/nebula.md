## What is Nebula?
Nebula is a scalable overlay networking tool with a focus on performance, simplicity and security.
It lets you seamlessly connect computers anywhere in the world. Nebula is portable, and runs on Linux, OSX, Windows, iOS, and Android.
It can be used to connect a small number of computers, but is also able to connect tens of thousands of computers.

Nebula incorporates a number of existing concepts like encryption, security groups, certificates,
and tunneling, and each of those individual pieces existed before Nebula in various forms.
What makes Nebula different to existing offerings is that it brings all of these ideas together,
resulting in a sum that is greater than its individual parts.

Further documentation can be found [here](https://www.defined.net/nebula/).

You can read more about Nebula [here](https://medium.com/p/884110a5579).

You can also join the NebulaOSS Slack group [here](https://join.slack.com/t/nebulaoss/shared_invite/enQtOTA5MDI4NDg3MTg4LTkwY2EwNTI4NzQyMzc0M2ZlODBjNWI3NTY1MzhiOThiMmZlZjVkMTI0NGY4YTMyNjUwMWEyNzNkZTJmYzQxOGU).

## Supported Platforms

#### Desktop and Server

Check the [releases](https://github.com/slackhq/nebula/releases/latest) page for downloads or see the [Distribution Packages](https://github.com/slackhq/nebula#distribution-packages) section.

- Linux - 64 and 32 bit, arm, and others
- Windows
- MacOS
- Freebsd

#### Distribution Packages

- [Arch Linux](https://archlinux.org/packages/community/x86_64/nebula/)
    ```
    $ sudo pacman -S nebula
    ```
- [Fedora Linux](https://copr.fedorainfracloud.org/coprs/jdoss/nebula/)
    ```
    $ sudo dnf copr enable jdoss/nebula
    $ sudo dnf install nebula
    ```

#### Mobile

- [iOS](https://apps.apple.com/us/app/mobile-nebula/id1509587936?itsct=apps_box&amp;itscg=30200)
- [Android](https://play.google.com/store/apps/details?id=net.defined.mobile_nebula&pcampaignid=pcampaignidMKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1)

## Technical Overview

Nebula is a mutually authenticated peer-to-peer software defined network based on the [Noise Protocol Framework](https://noiseprotocol.org/).
Nebula uses certificates to assert a node's IP address, name, and membership within user-defined groups.
Nebula's user-defined groups allow for provider agnostic traffic filtering between nodes.
Discovery nodes allow individual peers to find each other and optionally use UDP hole punching to establish connections from behind most firewalls or NATs.
Users can move data between nodes in any number of cloud service providers, datacenters, and endpoints, without needing to maintain a particular addressing scheme.

Nebula uses Elliptic-curve Diffie-Hellman (`ECDH`) key exchange and `AES-256-GCM` in its default configuration.

Nebula was created to provide a mechanism for groups of hosts to communicate securely, even across the internet, while enabling expressive firewall definitions similar in style to cloud security groups.