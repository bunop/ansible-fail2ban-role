fail2ban
========

Installs and configures [fail2ban](https://www.fail2ban.org/), including a
`recidive` jail for repeat offenders and a hardened `sshd` jail. Optional
task files add jails for Proxmox VE, NGINX and vsftpd on top of the base
configuration.

Requirements
------------

None beyond a Debian/Ubuntu-based target with `apt`.

Role Variables
---------------

Defined in `defaults/main.yml`:

| Variable | Default | Description |
| --- | --- | --- |
| `fail2ban_ignoreip` | `127.0.0.1/8 ::1` | Space-separated list of IPs/subnets fail2ban never bans |
| `fail2ban_destemail` | `root@localhost` | Address used for fail2ban's ban/whois notification emails |
| `fail2ban_dbpurgeage` | `14d` | How long fail2ban keeps ban history in its sqlite database |
| `fail2ban_sshd_findtime` | `12h` | Window in which `sshd` failed attempts are counted |
| `fail2ban_sshd_maxretry` | `3` | Failed attempts allowed in `fail2ban_sshd_findtime` before a ban |
| `fail2ban_sshd_bantime` | `6h` | Ban duration for the `sshd` jail |
| `fail2ban_recidive_findtime` | `1d` | Window in which repeat bans are counted by the `recidive` jail |
| `fail2ban_recidive_maxretry` | `3` | Number of bans in `fail2ban_recidive_findtime` before the `recidive` jail kicks in |
| `fail2ban_recidive_bantime` | `1w` | Ban duration imposed by the `recidive` jail |

The `sshd`/`recidive` values must stay consistent with `fail2ban_dbpurgeage`
(the database must retain entries for at least as long as the longest
`bantime` in use).

Additional jails
-----------------

The base `tasks/main.yml` only installs fail2ban and the `sshd`/`recidive`
jails. Extra jails are opt-in via `tasks_from`, and each appends its own
block to `/etc/fail2ban/jail.local`:

```yaml
- role: fail2ban
  tags: fail2ban_role

- import_role:
    name: fail2ban
    tasks_from: proxmox
  tags: [configure, fail2ban_role]

- import_role:
    name: fail2ban
    tasks_from: nginx
  tags: [configure, fail2ban_role]

- import_role:
    name: fail2ban
    tasks_from: vsftpd
  tags: [configure, vsftp_role]
```

Dependencies
------------

None.

Example Playbook
-----------------

```yaml
- hosts: servers
  roles:
    - role: fail2ban
      tags: fail2ban_role
      vars:
        fail2ban_ignoreip: "127.0.0.1/8 ::1 10.0.0.0/24"
```

License
-------

MIT

Author Information
-------------------

Paolo Cozzi ([@bunop](https://github.com/bunop))
