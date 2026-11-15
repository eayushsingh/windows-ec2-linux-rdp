# Project Screenshots & Evidence Guide

This directory is designated for visual verification artifacts captured during the hands-on lab: **Windows EC2 Deployment & Linux-to-Windows RDP Connection**.

To maintain a clean and professional portfolio repository, adhere to the naming guidelines and recommended captures below.

---

## Recommended Screenshot Checklist

| File Name | Description | Verification Purpose |
|:---|:---|:---|
| `01-ec2-instance-running.png` | AWS Management Console showing the Windows EC2 instance in **Running** state with **2/2 status checks passed**. | Proves successful provisioning and initial boot. |
| `02-security-group-rules.png` | Inbound rules of the associated Security Group showing port `3389` restricted to your IP (`<YOUR_IP>/32`). | Proves adherence to the Principle of Least Privilege. |
| `03-get-windows-password.png` | The AWS Console **Get Windows Password** decryption window with the RSA key uploaded. | Documents the key pair decryption workflow (mask sensitive strings). |
| `04-ubuntu-remmina-config.png` | Remmina connection profile configuration screen on Ubuntu Linux (Server IP, Username: `Administrator`). | Shows proper client-side parameter setup. |
| `05-rdp-cert-verification.png` | Certificate authenticity prompt in Remmina showing the Windows Server certificate footprint. | Documents TLS/SSL trust handshake. |
| `06-windows-desktop-connected.png` | Active Windows Server Desktop rendered within Remmina on Ubuntu, ideally showing Server Manager or system info. | Final proof of end-to-end connectivity. |

---

## Screenshot Standards

1. **Redact Sensitive Information**:
   - Mask or blur your actual public IP address (replace with placeholder notation like `203.0.113.25` or `REDACTED`).
   - Mask AWS Account Numbers in the top navigation bar.
   - Never show unencrypted private key strings or plaintext passwords.
2. **Format**: Save captures in `.png` format for crisp text clarity.
3. **Dimensions**: Standardize display resolutions where practical (e.g., 1080p) to avoid distorted scaling.

---

## Referencing Screenshots in Documentation

Once you add your screenshots to this directory, link them in `README.md` or `docs/setup-guide.md` using standard relative Markdown syntax:

```markdown
![Windows Desktop via Remmina](screenshots/06-windows-desktop-connected.png)
```

