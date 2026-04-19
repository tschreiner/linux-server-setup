# linux-server-setup

Ansible-Baseline fuer einen neuen Debian-13-Server mit Fokus auf sicheren Erstzugang, ueblichen Systempaketen, Updates, Firewall und Resolver-Client-Konfiguration.

Nicht im Scope:
- Direkte Installation von Tailscale auf dem Host
- Direkter Betrieb von Unbound auf dem Host

## Enthaltene Bausteine

- `robertdebock.bootstrap` fuer den Day-0-Bootstrap
- optional `robertdebock.users` fuer Admin-User und SSH-Keys
- optionale Docker-Engine-Installation mit Docker Compose Plugin und aktiviertem Docker-Daemon
- `devsec.hardening` fuer OS- und SSH-Hardening
- optional `oefenweb.ufw` fuer die Host-Firewall
- Eigene Rollen fuer Basis-Hosteinstellungen, Paketprofil, optionalen Resolver-Client, Zugriffspolitik und automatische Updates

## Repository-Aufbau

```text
ansible.cfg
requirements.yml
inventory/hosts.yml.example
inventory/hosts.yml            # lokal, gitignored
inventory/group_vars/all.yml
inventory/host_vars/debian.yml.example
inventory/host_vars/debian.yml # lokal, gitignored
playbooks/bootstrap.yml
playbooks/site.yml
playbooks/verify.yml
roles/
```

## Schnellstart

1. Abhaengigkeiten installieren:

```bash
ansible-galaxy install -r requirements.yml
```

2. Inventar und Host-Variablen anpassen:

- `cp inventory/hosts.yml.example inventory/hosts.yml`
- `cp inventory/host_vars/debian.yml.example inventory/host_vars/debian.yml`
- `inventory/hosts.yml`
- `inventory/host_vars/debian.yml`
- `inventory/group_vars/all.yml`

`inventory/hosts.yml` und `inventory/host_vars/*.yml` sind absichtlich lokal und werden nicht versioniert. Dadurch bleiben produktive IPs, Usernamen, Key-Pfade und Host-spezifische Daten automatisch aus spaeteren Commits heraus.

3. Day-0-Bootstrap ausfuehren:

```bash
ansible-playbook playbooks/bootstrap.yml -l debian-example
```

4. Baseline anwenden:

```bash
ansible-playbook playbooks/site.yml -l debian-example
```

Nach dem Bootstrap und dem ersten erfolgreichen `site.yml`-Lauf sollte das Inventar vom initialen `root`-Zugang auf den angelegten Admin-User umgestellt werden.

5. Verifikation ausfuehren:

```bash
ansible-playbook playbooks/verify.yml -l debian-example
```

## Wichtige Variablen

- `linux_server_setup_admin_users`: Admin-Accounts fuer `robertdebock.users`
- `linux_server_setup_hostname_manage`: Hostname-Verwaltung ein- oder ausschalten
- `linux_server_setup_manage_docker`: Docker Engine, Compose Plugin und Docker-Daemon ein- oder ausschalten
- `docker_engine_users`: optionale Benutzer, die zusaetzlich in die lokale Gruppe `docker` aufgenommen werden
- `access_policy_ssh_allowed_users`: SSH-Allowlist
- `access_policy_admin_allowed_cidrs`: erlaubte Quellnetze fuer SSH
- `linux_server_setup_manage_resolver`: Resolver-Konfiguration des Hosts ein- oder ausschalten
- `resolver_client_nameservers`: externe Resolver des Hosts
- `linux_server_setup_packages`: Standard-Paketprofil
- `linux_server_setup_enable_devsec_hardening`: aktiviert `devsec.hardening`
- `linux_server_setup_manage_firewall`: aktiviert `oefenweb.ufw`

## Hinweise

- Das Beispielinventar ist absichtlich nicht direkt produktionsreif. Der erste produktive Lauf soll fehlschlagen, wenn kein Admin-User, kein SSH-Key, kein erlaubtes Quellnetz oder kein Resolver gesetzt wurde.
- Fuer produktive Nutzung immer die `.example`-Dateien nach `inventory/hosts.yml` und `inventory/host_vars/<host>.yml` kopieren und nur die lokalen Kopien anpassen. Das Repository bleibt dadurch standardmaessig anonymisiert.
- Docker ist optional. Wenn `linux_server_setup_manage_docker` aktiviert wird, installiert die Rolle die Docker-APT-Quelle, `docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin` und `docker-compose-plugin` und aktiviert den Docker-Daemon.
- `hostname`, `ufw` und Resolver-Verwaltung sind opt-in. Standardmaessig bleibt der Hostname unberuehrt, die Firewall wird nicht verwaltet und es wird keine lokale Resolver-Konfiguration ausgerollt.
- Wenn `systemd-resolved` auf dem Zielhost nicht vorhanden ist, faellt die Resolver-Rolle automatisch auf eine direkte Verwaltung von `/etc/resolv.conf` zurueck, statt den Lauf abzubrechen.
- SSH-Hardening wird konservativ ueber die eigene Rolle `access_policy` ergaenzt, damit Root-Login und Passwort-Login auch dann explizit abgeschaltet werden, wenn zusaetzliche Hardening-Rollen spaeter ausgetauscht werden.
- Wenn `devsec.hardening` zu weit in die Systemdefaults eingreift, kann die Collection spaeter deaktiviert und durch eine kleinere Security-Kombination ersetzt werden.
