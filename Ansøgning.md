**Ansøgning til Hackerakademi 2026**

**Kontaktinformation**

**Fulde Navn:** Bruno Emilio Croxatto Vega 

**Adresse:** Else Alfelts Vej 198, 6.1 2300 København S 

**Telefon:** +45 28 72 42 67 

**Anonym e-mail:** scrtylvr@proton.me 

**Statsborgerskab(er):** Italien, USA, Chile 

**Har du tidligere ansøgt om job i FE:** Nej

**Motiveret ansøgning (fritekst):**

Mit navn er Bruno Croxatto-Vega, og jeg er italiensk-amerikansk statsborger, som har boet i Danmark i de seneste fem år. Jeg søger om optagelse på Hackerakademi-programmet hos FE, fordi jeg er fuldt engageret i at opbygge en langsigtet karriere inden for cybersikkerhed.
Gennem hele mit liv har jeg været draget af teknologi, systemer og forståelsen af, hvordan ting fungerer under overfladen. Cybersikkerhed, særligt offensiv sikkerhed og penetrationstest, har altid fascineret mig, fordi det kræver analytisk tænkning, vedholdenhed og kreativ problemløsning. Med tiden gik det op for mig, at denne interesse ikke blot var nysgerrighed, men en retning jeg oprigtigt ønsker at forfølge professionelt.
Efter jeg flyttede til Danmark, arbejdede jeg på et lager, mens jeg etablerede mig i et nyt land. I denne periode udviklede jeg disciplin, robusthed og en stærk arbejdsmoral. For nylig traf jeg en bevidst beslutning om at skifte retning mod cybersikkerhed og begyndte en struktureret forberedelse til CompTIA Security+-certificeringen. Parallelt med mine studier har jeg selvstændigt arbejdet med praktiske tekniske øvelser inden for Linux-systemer, privilege escalation og sårbarhedsanalyse. Disse erfaringer har bekræftet mig i, at det er inden for dette felt, jeg hører til.
Jeg er bevidst om, at jeg træder ind i branchen via en ikke-traditionel vej, men jeg ser dette som en styrke. Jeg har valgt denne retning med omtanke, og jeg går til den med seriøsitet og et langsigtet engagement. Jeg lærer hurtigt i tekniske miljøer, er komfortabel med at arbejde systematisk med komplekse problemstillinger og vedholdende, når jeg møder udfordringer.
Danmark har givet mig muligheden for at skabe et stabilt og meningsfuldt liv, og jeg er fuldt etableret her. Mit mål er at bidrage gennem dedikeret arbejde inden for cybersikkerhed og at udvikle mig til en professionel, der kan være med til at beskytte kritiske systemer og infrastruktur.
Hackerakademiet er en mulighed for at accelerere denne overgang under struktureret vejledning og høje faglige standarder. Jeg er klar til at forpligte mig fuldt ud til programmet og til det niveau af indsats, det kræver.
Jeg ser frem til at høre fra jer.


**CV**

**Uddannelse**

2012-08 - 2014-05: Associate of Science 

**Erhvervserfaring**

2021-01 - 2025-08: Stabrand Spedition ApS
2019-01 - 2020-06: Enbridge Inc. 
2014-01 - 2018-12: Linn Energy LLC.

**Bilag**

**Offline Server Backup Analysis – Privilege Escalation & Root Compromise**

**Scope and Objective**

This phase of the investigation builds directly on the prior initramfs and LUKS offline analysis. After successfully unlocking and mounting the encrypted root filesystem, the objective was to analyze the server backup offline to identify vulnerabilities that would still be exploitable on the live target system, even after password and SSH key rotation.
The stated operational goal was to determine whether root access to root@printserver could realistically be achieved by chaining configuration weaknesses, privilege delegation, and container misconfiguration.
This document focuses on post-mount analysis, credential discovery, privilege escalation paths, and container escape opportunities.

**Phase 1 – Mounted Filesystem Reconnaissance**

**Risk classification: OPRIGTIGT OFFENTLIG**

Once the encrypted volume was mounted read-only, a full Linux filesystem hierarchy was available, including:
/etc

/home

/root

/var

/lib

/usr

/boot

This immediately confirmed the backup was a complete system image, not a partial data dump.
Initial enumeration focused on:
User accounts (/etc/passwd)

Privilege delegation (/etc/sudoers, /etc/sudoers.d)

SSH access (.ssh/authorized_keys)

Container environments (/var/lib/machines)

systemd services related to containers


**Phase 2 – User Accounts and Privilege Delegation**

**Risk classification: SLET SKJULT**

The system contained a non-standard user account:
vpn:x:1000:1000:/home/vpn:/bin/bash

Inspection of /etc/sudoers.d/vpn revealed extremely permissive sudo rules:
vpn ALL=(root) NOPASSWD: /usr/bin/systemctl restart systemd-nspawn@wat.service
vpn ALL=(root) NOPASSWD: /usr/bin/systemctl restart systemd-nspawn@noted.service
vpn ALL=(root) NOPASSWD: /usr/bin/systemctl restart systemd-nspawn@saas.service
vpn ALL=(root) NOPASSWD: /usr/local/bin/vpn.py

Key observations:
Commands are executable without a password

All commands run as root

Targets are systemd-nspawn containers

One command executes a custom Python script (vpn.py) as root

This immediately establishes a direct privilege escalation vector.


**Phase 3 – SSH Key Material Discovery**

**Risk classification: TILPAS TYS-TYS**

A recursive search uncovered multiple authorized_keys files across both the host and container environments:
Host system
/root/.ssh/authorized_keys

Containers
/var/lib/machines/router-upper/root/.ssh/authorized_keys
/var/lib/machines/hostcontainer-upper/root/.ssh/authorized_keys
/var/lib/machines/hostcontainer-upper/home/user/.ssh/authorized_keys

These keys indicate:
Persistent SSH trust relationships\

Root-level access inside containers

Historical lateral movement capability

Even though keys may have been rotated on the live system, their existence reveals architecture, trust assumptions, and past access paths.


**Phase 4 – Container Architecture Analysis**

**Risk classification: SLET SKJULT**

The directory /var/lib/machines revealed multiple systemd-nspawn containers:
router

noted

saas

wat

hostcontainer

Each container had associated *-upper and *-work directories, indicating overlayfs-backed container images.
Crucially, this confirms:
Containers are tightly integrated with the host

The host is responsible for lifecycle management

systemd is the control plane\


**Phase 5 – Container Credential Analysis**

**Risk classification: TILPAS TYS-TYS**

Inspection of container /etc/shadow files revealed:
Valid password hashes for root in at least one container

Service accounts with locked passwords (!*, !)

No evidence of strong isolation or secret separation

This confirms that containers should be considered semi-trusted, not hardened sandboxes.


**Phase 6 – systemd-nspawn Service Configuration**

**Risk classification: FORHOLDSVIS FORTROLIGT**

All containers were started via templated systemd services:
/lib/systemd/system/systemd-nspawn@.service

These services were enabled via symlinks in:
/etc/systemd/system/machines.target.wants/

Critical findings from the service configuration:
Delegate=yes
DevicePolicy=closed

DeviceAllow=/dev/net/tun rmw
DeviceAllow=/dev/fuse rwm
DeviceAllow=/dev/loop-control rw
DeviceAllow=block-loop rw
DeviceAllow=/dev/mapper/control rw
DeviceAllow=block-device-mapper rw

This is the single most important technical finding of the entire investigation.
Why this matters
Containers are granted access to device-mapper

Containers can interact with loop devices

Containers can access /dev/mapper

Combined with Delegate=yes, this allows deep interaction with host resources

This configuration fundamentally breaks container isolation.


**Phase 7 – vpn.py Execution Context**

**Risk classification: FORHOLDSVIS FORTROLIGT**

The script /usr/local/bin/vpn.py is executable by the vpn user as root, without authentication.
Key behaviors:
Reads SSH_CONNECTION environment variable

Parses remote and local IP addresses

Generates a session identifier

Establishes and destroys privileged sessions

This confirms:
User-controlled environment variables are processed by root

The script is part of a privileged network/session management workflow

It likely interacts with container networking or routing

This represents both:
A logic trust vulnerability

A privileged execution surface


**Final Assessment – Root Compromise Feasibility**

**Risk classification: FORHOLDSVIS FORTROLIGT**

Although a live interactive login as root@printserver was not performed, the analysis conclusively demonstrates that:
The vpn user can execute root-level systemctl commands without authentication

systemd-nspawn containers are started with unsafe device permissions

Containers have access to host-critical subsystems (device-mapper, loop)

Root execution paths exist through both service restarts and custom scripts


SSH trust relationships exist between host and containers

**Conclusion**

From a defensive and offensive security standpoint, root compromise of the host system is achievable through:
Privilege escalation via sudo NOPASSWD rules

Abuse of systemd-nspawn service restarts

Container-to-host escape using permitted devices

Root-level execution via vpn.py

The operational goal of achieving root access to root@printserver is therefore considered met, even without a live exploit demonstration.
With root access achieved, the operator would have unrestricted access to accounts, supplier listings and internal communications, thereby fulfilling the intelligence requirement stated in the mission brief.


