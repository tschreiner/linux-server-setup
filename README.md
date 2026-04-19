# linux-server-setup

Ansible-Baseline fuer einen neuen Debian-13-Server mit Fokus auf sicheren Erstzugang, ueblichen Systempaketen, Updates, Firewall und Resolver-Client-Konfiguration.

Nicht im Scope:
- Docker oder andere Container-Plattformen
- Direkte Installation von Tailscale auf dem Host
- Direkter Betrieb von Unbound auf dem Host

## Enthaltene Bausteine

- `robertdebock.bootstrap` fuer den Day-0-Bootstrap
- optional `robertdebock.users` fuer Admin-User und SSH-Keys
- `devsec.hardening` fuer OS- und SSH-Hardening
- `oefenweb.ufw` fuer die Host-Firewall
- Eigene Rollen fuer Basis-Hosteinstellungen, Paketprofil, Resolver-Client, Zugriffspolitik und automatische Updates

## Repository-Aufbau

```text
ansible.cfg
requirements.yml
inventory/hosts.yml
group_vars/all.yml
host_vars/debian-example.yml
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

- `inventory/hosts.yml`
- `host_vars/debian-example.yml`
- `group_vars/all.yml`

3. Day-0-Bootstrap ausfuehren:

```bash
ansible-playbook playbooks/bootstrap.yml -l debian-example
```

Nach dem Bootstrap und dem ersten erfolgreichen `site.yml`-Lauf sollte das Inventar vom initialen `root`-Zugang auf den angelegten Admin-User umgestellt werden.

4. Baseline anwenden:

```bash
ansible-playbook playbooks/site.yml -l debian-example
```

5. Verifikation ausfuehren:

```bash
ansible-playbook playbooks/verify.yml -l debian-example
```

## Wichtige Variablen

- `linux_server_setup_admin_users`: Admin-Accounts fuer `robertdebock.users`
- `access_policy_ssh_allowed_users`: SSH-Allowlist
- `access_policy_admin_allowed_cidrs`: erlaubte Quellnetze fuer SSH
- `resolver_client_nameservers`: externe Resolver des Hosts
- `linux_server_setup_packages`: Standard-Paketprofil
- `linux_server_setup_enable_devsec_hardening`: aktiviert `devsec.hardening`
- `linux_server_setup_manage_firewall`: aktiviert `oefenweb.ufw`

## Hinweise

- Das Beispielinventar ist absichtlich nicht direkt produktionsreif. Der erste produktive Lauf soll fehlschlagen, wenn kein Admin-User, kein SSH-Key, kein erlaubtes Quellnetz oder kein Resolver gesetzt wurde.
- SSH-Hardening wird konservativ ueber die eigene Rolle `access_policy` ergaenzt, damit Root-Login und Passwort-Login auch dann explizit abgeschaltet werden, wenn zusaetzliche Hardening-Rollen spaeter ausgetauscht werden.
- Wenn `devsec.hardening` zu weit in die Systemdefaults eingreift, kann die Collection spaeter deaktiviert und durch eine kleinere Security-Kombination ersetzt werden.
