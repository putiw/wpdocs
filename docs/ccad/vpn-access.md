# CCAD VPN and VDI Access

## Accounts

- CCAD account: `wenp@clevelandclinicabudhabi.ae`
- Use the CCAD account, not the NYU account, to set up Microsoft Authenticator for CCAD MFA.

## Local Secret Files

The VPN XML files and import passcode are local only:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/CCAD/
```

Do not commit this folder.

## macOS Setup

Install **FortiClient VPN-only**. The CCAD guide used version `7.4.3`.

Before importing the profile:

1. Open **System Settings**.
2. Go to **Privacy & Security**.
3. Open **Full Disk Access**.
4. Enable `fctservctl2`.

Import the VPN profile:

1. Open FortiClient VPN.
2. Add a new VPN connection.
3. Select XML import.
4. Import:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/CCAD/VPN-Profile-MacOS.sconf
```

5. Enter the VPN import passcode from:

```text
/Users/pw1246/Documents/GitHub/wpdocs/secrets/CCAD/CCAD VPN.rtf
```

6. Select the imported VPN profile:

```text
CCAD-ZTNA
```

7. Log in with the CCAD account and approve MFA in Microsoft Authenticator.

VPN server in the profile:

```text
ccadvpn.clevelandclinicabudhabi.ae
```

Windows note: import `VPN-Profile-Windows.conf` into FortiClient using the same passcode.

## Verify VPN

```bash
ssh imac1@10.157.29.87
```

`10.157.29.87` is Asif's iMac. It is the main machine used for CCAD data export and OsiriX access.

## Omnissa Horizon VDI

After VPN connects, open **Omnissa Horizon Client** and connect to the CCAD VDI.

```text
mydesktop.ccadi.local
```

Use the **Imaging Desktop** pool.

Expected access from the VDI:

- Synapse PACS
- SyngoVia
- BrainLab

BrainLab station:

```text
CCADBLP050.CCADI.LOCAL
```

If VDI works but PACS, SyngoVia, or BrainLab rejects the account, ask CCAD IT to add the missing app/group permission.
