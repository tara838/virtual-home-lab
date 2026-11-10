# IP plan, names and accounts

## Networks

| Network    | libvirt name | Subnet           | Gateway               | Type                              |
| ---------- | ------------ | ---------------- | --------------------- | --------------------------------- |
| Upstream   | `default`    | 192.168.122.0/24 | host NAT              | libvirt NAT, used as OPNsense WAN |
| Client LAN | `lab-lan`    | 10.10.10.0/24    | 10.10.10.1 (OPNsense) | Isolated, no libvirt DHCP         |
| Servers    | `lab-srv`    | 10.10.20.0/24    | 10.10.20.1 (OPNsense) | Isolated, no libvirt DHCP         |


## Hosts

| Host     | Interface        | Network | Address                 | Role                             |
| -------- | ---------------- | ------- | ----------------------- | -------------------------------- |
| OPNsense | WAN (vtnet0)     | default | DHCP (192.168.122.x)    | Uplink                           |
| OPNsense | LAN (vtnet1)     | lab-lan | 10.10.10.1/24           | LAN gateway                      |
| OPNsense | SERVERS (vtnet2) | lab-srv | 10.10.20.1/24           | Servers gateway                  |
| DC01     | Ethernet         | lab-lan | 10.10.10.10/24 (static) | AD DS, DNS, DHCP, time authority |
| CLIENT01 | Ethernet         | lab-lan | DHCP, 10.10.10.100-200  | Windows 10 Pro domain member     |
| deb01    | enp1s0           | lab-srv | 10.10.20.10/24 (static) | nginx, Samba, SSH, chrony        |

## Domain and DNS

| Item                     | Value                                                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| Domain                   | `barab0lia.internal`                                                                                              |
| NetBIOS name             | `BARAB0LIA`                                                                                                       |
| DNS server for all hosts | `10.10.10.10` (DC01)                                                                                              |
| DNS forwarder (on DC01)  | `10.10.10.1` (OPNsense)                                                                                           |
| Static record            | `deb01.barab0lia.internal` to `10.10.20.10`                                                                       |
| DHCP scope               | `LAN_Clients`, 10.10.10.100-200, 8-day lease, router `10.10.10.1`, DNS `10.10.10.10`, domain `barab0lia.internal` |

## Organizational units and groups

```
barab0lia.internal
└─ OU=Lab
   ├─ OU=Users
   │   ├─ OU=IT
   │   ├─ OU=HR
   │   ├─ OU=Sales
   │   └─ OU=Accountant
   ├─ OU=Groups
   ├─ OU=Workstations   (CLIENT01 is here)
   └─ OU=Servers
```

| Group | Purpose |
|---|---|
| `GG_IT`, `GG_HR`, `GG_Sales`, `GG_Accountant` | Department membership. `GG_IT` drives the item-level targeting of the `I:` drive |

## Accounts

| Account          | Type                      | Notes                                                    |
| ---------------- | ------------------------- | -------------------------------------------------------- |
| `kono.it`        | Standard user, IT         | Member of `GG_IT`                                        |
| `sheva.hr`       | Standard user, HR         | Member of `GG_HR`                                        |
| `carol.sales`    | Standard user, Sales      | Member of `GG_Sales`                                     |
| `bob.accountant` | Standard user, Accountant | Member of `GG_Accountant                                 |
| `adm.taras`      | Privileged, Domain Admins | Separate from everyday accounts, used for administration |

## Samba permission matrix (deb01)

| Share | Linux group | `kono.it` | `sheva.hr` |
|---|---|---|---|
| `Company` (`/srv/samba/company`) | `smb-all` | Read/write | Read/write |
| `IT` (`/srv/samba/it`) | `smb-it` | Read/write | **Denied** |

The drive map controls what is displayed (item-level targeting on `GG_IT`). Samba permissions control what is allowed.

## Firewall policy summary

**OPNsense, SERVERS interface** (default deny, specific passes):

| Source      | Destination | Protocol / port | Purpose                 |
| ----------- | ----------- | --------------- | ----------------------- |
| 10.10.20.10 | 10.10.10.10 | UDP 123         | Time sync               |
| 10.10.20.10 | 10.10.10.10 | TCP/UDP 53      | DNS                     |
| SERVERS net | any         | TCP web ports   | Package updates         |
| 10.10.20.10 | any         | ICMP            | Diagnostics (test rule) |

deb01 host firewall (nftables): input policy drop. Allowed from `10.10.10.0/24`: TCP 22, 80, 445, plus ICMP echo and replies to its own connections.
