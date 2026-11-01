# Phase 1 — Network & Domain Controller

## Goal
Create an isolated virtual network (`labnet`) in VirtualBox and build the first
machine on it: **DC01**, a Windows Server 2022 domain controller running Active
Directory and DNS for the `watchkeeper.local` domain.

## Prerequisites
- VirtualBox installed on the Linux Mint host
- A Windows Server 2022 VM already created (not yet configured)
- Host RAM: 32 GB (DC01 uses 4 GB)

---

## Background: the three networking terms you need

VirtualBox gives each VM one or more virtual network adapters, and each adapter
has a *mode*. Only three matter here:

| Mode | What it connects to | Use in this lab |
|------|--------------------|-----------------|
| **Internal Network** | Only other VMs on the same named network. No host, no internet. | This is `labnet` — our isolated lab |
| **NAT** | The internet, through the host. Isolated from other VMs. | Temporary, only for Windows updates/activation |
| **Bridged** | Your real home network | **Never use this for the lab** |

The plan: DC01 gets one **Internal Network** adapter (always on) and one
**NAT** adapter (on only when it needs updates, off during all attacks).

---

## Stage 1 — Create the isolated network

In VirtualBox you don't create the internal network separately; you create it by
*naming* it on a VM's adapter. The first VM to use the name `labnet` brings it
into being.

1. Select the Windows Server VM (DC01) in VirtualBox. Make sure it's powered off.
2. Click **Settings → Network**.
3. **Adapter 1:**
   - Check **Enable Network Adapter**
   - *Attached to:* **Internal Network**
   - *Name:* type `labnet` (exactly, lowercase)
   - Expand **Advanced**, set **Promiscuous Mode: Allow All**
4. **Adapter 2** (temporary internet for setup):
   - Check **Enable Network Adapter**
   - *Attached to:* **NAT**
5. Click **OK**.

> 📸 Screenshot: the Adapter 1 settings showing Internal Network / labnet.

---

## Stage 2 — Install Windows Server 2022

If your VM is freshly created with the ISO attached, start it and install:

1. Start DC01. Boot from the Windows Server 2022 ISO.
2. Choose **Windows Server 2022 Standard (Desktop Experience)** — the version
   with a GUI. The "Core" option is command-line only; avoid it for now.
3. Accept the license, choose **Custom install**, select the disk, let it install.
4. When prompted, set a strong **Administrator password**. Write it in your
   private `notes/` file (which is gitignored), never in the repo.
5. Let it reboot to the desktop.

> 📸 Screenshot: the Server Manager dashboard on first boot.

---

## Stage 3 — Set a static IP

A domain controller must have a fixed address, because every other machine will
look to it for DNS.

1. On DC01, open **Settings → Network & Internet → Ethernet**, or run `ncpa.cpl`
   from the Start menu to open network adapters.
2. You'll see two adapters (from the two VirtualBox adapters). Identify the
   **Internal Network** one — it's the one with no internet. Rename it `LAB` if
   you like, to keep them straight.
3. Right-click it → **Properties → Internet Protocol Version 4 (TCP/IPv4) →
   Properties**.
4. Choose **Use the following IP address** and enter:
   - IP address: `10.10.10.10`
   - Subnet mask: `255.255.255.0`
   - Default gateway: *leave blank* (a pure internal network has no router)
5. **Use the following DNS server addresses:**
   - Preferred DNS: `127.0.0.1` (the server points to itself; it becomes the DNS
     server once promoted)
6. Click OK.

> 📸 Screenshot: the IPv4 settings filled in.

**Verify:** open PowerShell and run `ipconfig`. The LAB adapter should show
`10.10.10.10`.

---

## Stage 4 — Rename the computer

Do this *before* promoting, because the name is baked in afterward.

```powershell
Rename-Computer -NewName "DC01" -Restart
```

The VM reboots. After it comes back, confirm with `hostname` — it should say
`DC01`.

---

## Stage 5 — Promote to a domain controller

This installs Active Directory and turns DC01 into the heart of the domain.

1. In **Server Manager**, click **Manage → Add Roles and Features**.
2. Click Next until **Server Roles**. Check **Active Directory Domain Services**.
   Accept the extra features it asks to add. Finish the wizard and let it install.
3. Back in Server Manager, click the **yellow warning flag** at the top → **Promote
   this server to a domain controller**.
4. Select **Add a new forest**. Root domain name: `watchkeeper.local`
5. On the next page, set a **Directory Services Restore Mode (DSRM) password**
   (save it in `notes/`). Leave other options at their defaults.
6. Click through. It will warn about DNS delegation — that's expected in a lab,
   ignore it. Let the prerequisite check run, then click **Install**.
7. The server reboots automatically.

> 📸 Screenshot: the "Add a new forest / watchkeeper.local" page.

---

## Verification

After the reboot, confirm the domain is alive. Log in as
`WATCHKEEPER\Administrator`, open PowerShell, and run:

```powershell
# Confirms AD is running and names the domain
Get-ADDomain | Select-Object Name, DNSRoot, DomainMode

# Confirms this machine is a DC
Get-ADDomainController | Select-Object Name, Domain

# Confirms DNS is answering for the domain
Resolve-DnsName watchkeeper.local
```

If `Get-ADDomain` returns `watchkeeper.local` and no errors, Phase 1 is done.

> 📸 Screenshot: the output of `Get-ADDomain`.

**Last step:** power off and take a VirtualBox **snapshot** named
`DC01-clean-promoted`. If anything breaks later, you roll back to here instead of
rebuilding.

---

## Problems and fixes
*(Fill this in as you go — every error and how you solved it.)*

---

## AI collaboration
*(What you asked, what was right or wrong, what you decided. Example: did the AI's
static-IP values match what your lab actually needed? Did you verify the domain
name before committing to it?)*

---

## Lessons learned
*(Two or three bullets once the phase is done.)*
