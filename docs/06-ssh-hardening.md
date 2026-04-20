# 🔑 SSH Hardening — pfSense

This document covers how SSH access to pfSense was secured by replacing password-based authentication with SSH public key authentication.

The goal was to reduce administrative attack surface, improve access security, and align remote management with enterprise best practices.

---

## 📋 Table of Contents

- [Why SSH Hardening Matters](#why-ssh-hardening-matters)
- [Initial SSH State](#initial-ssh-state)
- [Security Goals](#security-goals)
- [Key Generation Process](#key-generation-process)
- [Deploying the Public Key to pfSense](#deploying-the-public-key-to-pfsense)
- [pfSense SSH Configuration](#pfsense-ssh-configuration)
- [Firewall Rule — Restrict SSH to Admin VLAN Only](#firewall-rule--restrict-ssh-to-admin-vlan-only)
- [Testing & Verification](#testing--verification)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## ❓ Why SSH Hardening Matters

By default, password-based SSH access creates unnecessary risk:

| Risk | Description |
|---|---|
| Brute-force attacks | Automated tools can repeatedly attempt password logins |
| Credential reuse | Compromised passwords from other services can be replayed |
| Weak password exposure | Human-chosen passwords are often predictable |
| Broad attack surface | Password auth is accessible to any network with SSH access |

Using **SSH public key authentication** removes password-based login attempts entirely and significantly improves remote administrative security.

This is standard practice in production environments and an important part of infrastructure hardening.

---

## 🔓 Initial SSH State

Before hardening, the configuration allowed:

- ✅ SSH enabled
- ✅ Password-based authentication permitted
- ✅ GUI and SSH accessible from multiple VLANs

This created convenience, but also unnecessary exposure. The objective was to restrict management access to **trusted administrative networks only** and require **cryptographic authentication**.

---

## 🎯 Security Goals

| Objective | Status |
|---|---|
| Restrict SSH access to Admin VLAN only | ✅ Completed |
| Disable password authentication | ✅ Completed |
| Enforce public key authentication only | ✅ Completed |
| Reduce management-plane exposure | ✅ Completed |
| Maintain secure recovery access if lockout occurred | ✅ Completed |

---

## 🔐 Key Generation Process

SSH keys were generated from the **Windows administrative workstation** using **PuTTYgen**.

### Key Specification

| Setting | Value |
|---|---|
| Key type | RSA |
| Key length | 4096-bit |
| Private key format | `.ppk` (PuTTY format) |
| Public key format | OpenSSH (pasted into pfSense) |

### Steps

1. Open **PuTTYgen**
2. Set type to **RSA**, key length to **4096**
3. Click **Generate** and move the mouse to add entropy
4. Save the private key as:

```text
pfsense-admin.ppk
```

5. Copy the **public key** from the PuTTYgen window (OpenSSH format)
6. Store the private key securely — never share or upload it

> ⚠️ **Important:** The `.ppk` private key file should never be stored in this repository or any cloud location. Keep it on your administrative workstation only.

---

## 📤 Deploying the Public Key to pfSense

1. Log in to the **pfSense web GUI**
2. Navigate to **System → User Manager → Admin user**
3. Scroll down to **Authorised SSH Keys**
4. Paste the **OpenSSH public key** copied from PuTTYgen
5. Click **Save**

pfSense stores the public key and uses it to authenticate incoming SSH connections.

---

## ⚙️ pfSense SSH Configuration

Navigate to **System → Advanced → Admin Access** and apply the following:

| Setting | Value |
|---|---|
| Secure Shell Server | Enabled |
| SSHd Key Only | **Public Key Only** |
| SSH Port | 22 (or custom port for added obscurity) |
| Authentication method | Public key — password disabled |

Setting **SSHd Key Only** to *Public Key Only* disables password-based authentication entirely at the pfSense level.

---

## 🧱 Firewall Rule — Restrict SSH to Admin VLAN Only

To limit SSH exposure, a firewall rule was created to allow SSH **only from VLAN 10 (Admin)**:

| Field | Value |
|---|---|
| Interface | VLAN 10 (Admin) |
| Protocol | TCP |
| Source | `192.168.10.0/24` |
| Destination | pfSense LAN interface IP |
| Destination Port | 22 |
| Action | Pass |
| Description | Allow SSH from Admin VLAN only |

All other VLANs rely on the **default deny rule** and cannot reach the pfSense SSH service.

---

## ✅ Testing & Verification

### Confirm key-based login works

1. Open **PuTTY**
2. Enter pfSense IP under **Host Name**
3. Navigate to **Connection → SSH → Auth → Credentials**
4. Load `pfsense-admin.ppk` as the private key
5. Connect — login should succeed **without prompting for a password**

### Confirm password authentication is disabled

1. Open a new PuTTY session to pfSense
2. Remove the private key from the Auth settings
3. Attempt to connect
4. Expected result: **connection refused or authentication failure — no password prompt**

### Confirm SSH is blocked from non-admin VLANs

1. Connect a device on VLAN 20, 30, or 40
2. Attempt SSH to pfSense management IP
3. Expected result: **connection times out — firewall rule blocks the attempt**

---

## 📊 Result

| Area | Before Hardening | After Hardening |
|---|---|---|
| Authentication method | Password | Public key only |
| SSH access scope | All VLANs | Admin VLAN only |
| Brute-force exposure | High | Eliminated |
| Credential reuse risk | Present | Eliminated |
| Alignment with enterprise practice | Partial | ✅ Fully aligned |

---

## 💡 Lessons Learned

- **Password auth feels safe until it isn't.** Default SSH with passwords is one of the most commonly exploited entry points in real environments. Removing it entirely is the right call.
- **Key management is its own discipline.** Generating, storing, and protecting private keys correctly is as important as the hardening itself.
- **Firewall rules and SSH hardening work together.** Key-based auth protects the authentication layer, but restricting which networks can even *reach* SSH is an equally important control.
- **Test failure, not just success.** Verifying that password auth is *actually* disabled — not just configured to be — is a critical step that is easy to skip.

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
