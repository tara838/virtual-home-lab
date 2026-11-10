**Goal:** prove that a client can find the domain, authenticate against it and receive policy.
**What I did:** the client received a DHCP lease from DC01 (address, gateway, DNS), then joined `barab0lia.internal` directly into `OU=Workstations`, because GPOs link to OUs and not to the default Computers container.

| GPO                                          | Type            | Linked to         | Effect                                                                                 |
| -------------------------------------------- | --------------- | ----------------- | -------------------------------------------------------------------------------------- |
| HR - Restrict Control Panel                  | User policy     | `OU=HR`           | Blocks Control Panel for HR users only. Tested against an IT user as a control         |
| WKS - Logon Banner                           | Computer policy | `OU=Workstations` | Legal notice at logon for every user of the machine                                    |
| Default Domain Policy (password and lockout) | Account policy  | Domain            | Password complexity, length and age, account lockout. Lockout tested on a test account |
| USR - Drive Maps                             | User preference | `OU=Users`        | `S:` Company for everyone, `I:` IT only via item-level targeting on `GG_IT`            |
User vs computer policy - the Control Panel block follows the *person*, the banner follows the *machine*. Both are demonstrated.

---

Client received lease from DC01 DHCP: IP, gateway, DNS
![[phase4-client-dhcp-received.png]]


And we can see that the IP address was indeed given to the windows 10 client in DHCP Management Console
![[phase4-dhcp-lease-check-on-GUI.png]]

We can also extract the same information from the CLI
![[phase4-check-lease-cli.png]]


Let's test the connectivity with external and internal hosts. Also check if DNS service is working properly
![[phase4-client-connectivity-after-boot.png]]
![[ping-again?.png]]



==🟣Windows 10 joins the domain==
Now lets join this host to the domain barab0lia.internal
![[phase-4-join-domain.png]]

Check if the client successfully joined the domain 
![[phase-4-join-succes.png.png]]

Or check in DC host in Active Directory Users and Computers
![[phase-4-check-joining-Client01.png]]

![[phase-4-check-on-DC01-powershell.png.png]]




==🔴Creating GPO==
Create a GPO that restricts access to control panel
![[phase4-GPO-Restrict-Control-Panel-create-Powershell.png]]

Check the created GPO in Group Policy Management
![[phase4-GPO-check-gui-HR-restrict-Control-Panel.png]]



Log in as a user inside HR OU to see if the created GPO will take affect.
![[phase4-signin-as-HR.png]]![[phase4-nltest.png]]


`gpupdate /force` command to download and re-apply every single policy from Domain Controller that is assigned to this user
![[phase4-GPO-gpupdate-force.png]]


The access to control panel is successfully blocked for any user within HR Organizational Unit
![[phase4-deny-to-open-control-panel.png]]


We logged in as a `kono.it` user and control Panel successfully opened  because the user is not in HR Organizational Unit. So it works as expected
![[phase4-open-control-panel.png]]


The process of creating GPO in GUI using Group Policy Management to objects within Workstations OU. So any object inside that OU inherit it.
![[phase4-gpo-creating-in-gui.png]]

![[phase4-creating-GPO-logon-banner.png]]

![[phase4-verify-creating-GPO-logon-banner.png]]



Functionality test
![[phase4-computer_GPO-applied.png]]

Edit the Default Domain Password Policy that applies to any object in the domain. 
Because it is linked at the domain root (`barab0lia.internal`), Group Policy flows down from where it is linked, so it reaches every OU below, and that covers all of the lab.
In production I would add a fine-grained password policy for privileged accounts.”
![[phase-4-password-policy.png]]

To unlock account go to Active Directory Users and Computers and use search filter to find specific. Then tick the button that states `Unlock the accont`
![[phase4-unlock-account.png]]