# What I would do differently in production

## Identity and directory

| Lab                                                                                                           | Production                                                                                                                         |
| ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| One domain controller                                                                                         | At least two DCs in separate failure domains. Replication, FSMO placement, and no snapshot rollback of a DC (USN rollback risk)    |
| Samba file server is standalone, with its own user list and Linux groups maintained by hand next to AD groups | Join the file server to AD (winbind or SSSD with Kerberos) so users sign in with domain accounts and the AD groups control access. |
| One shared lab password policy                                                                                | Fine-grained password policies for privileged accounts, and MFA                                                                    |
| Built-in admin avoided, but `adm.taras` is a permanent Domain Admin                                           | Tiered administration, just-in-time elevation, LAPS for local admin passwords                                                      |


## Network and firewall

| Lab                                          | Production                                                                  |
| -------------------------------------------- | --------------------------------------------------------------------------- |
| LAN has a broad "allow LAN to any" rule      | Replace with explicit rules per service, as done on the SERVERS interface   |
| `SERVERS net` may reach any web port         | Restrict updates to the vendor mirrors or an internal package proxy         |
| ICMP test rule from `deb01`                  | Remove after diagnostics                                                    |
| Single firewall, self-signed GUI certificate | HA pair, internal CA certificate, admin GUI limited to a management network |

## Time

| Lab                                | Production                                                                                      |
| ---------------------------------- | ----------------------------------------------------------------------------------------------- |
| DC01 follows two public pools      | Several upstream sources, ideally a local reference. Monitor offset and stratum, alert on drift |

## Server hardening

- Key-only SSH is in place. Add fail2ban or equivalent, and central log forwarding.
- nftables restricts inbound traffic. Add outbound filtering where appropriate.
- Apply a hardening baseline (CIS benchmark) to Windows and Linux.

## Operations

| Area                   | Production                                                                                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Backups                | **Snapshots are not backups.** Use real backup with restore tests, and a documented recovery procedure for AD                                     |
| Patching               | WSUS or equivalent for Windows, unattended security updates and a maintenance window for Linux                                                    |
| Monitoring and logging | Central syslog and event collection, metrics, alerting                                                                                            |
| Configuration as code  | The scripts here are a start. Production would keep GPOs, firewall config and server config in version control, with Ansible or similar for Linux |
| DHCP                   | Failover pair, reservations for infrastructure                                                                                                    |
| Documentation          | Keep the IP plan, runbooks and this troubleshooting log updated as a living document                                                              |

