# Comprehensive Setup & Implementation Guide

This guide provides end-to-end instructions for launching a Windows Server EC2 instance on Amazon Web Services (AWS) and establishing a secure graphical Remote Desktop Protocol (RDP) connection from an Ubuntu Linux environment using the Remmina client.

---

## 1. Core Technical Concepts

Before configuring resources, understanding the underlying building blocks is essential for cloud engineering:

### What is Amazon EC2?
**Amazon Elastic Compute Cloud (Amazon EC2)** is an Infrastructure-as-a-Service (IaaS) offering from AWS that provides resizable, on-demand virtual compute capacity in the cloud. It eliminates the need to invest in upfront hardware, allowing engineers to spin up virtual servers in minutes across global AWS data centers.

### What is an EC2 Instance?
An **EC2 Instance** is a virtual server running within the AWS compute environment. It is composed of virtual CPU cores (vCPU), random-access memory (RAM), storage volumes (Amazon EBS), and virtual network interfaces (ENI), configured based on an Amazon Machine Image (AMI).

### Why Windows Server?
**Microsoft Windows Server** provides an enterprise-grade operating system designed for centralized management, Active Directory domain services, enterprise applications (.NET, IIS, MS SQL), and graphical administrative tooling. In AWS environments, Windows Server AMIs come pre-packaged with cloud initialization agents (such as `EC2Launch v2`).

### Why Ubuntu Linux?
**Ubuntu Linux** is one of the most widely deployed open-source Linux distributions in DevOps and cloud engineering. Using an Ubuntu workstation to manage Windows servers models standard cross-platform workflows, demonstrating multi-OS systems engineering skills.

### What is RDP and Why Port 3389?
The **Remote Desktop Protocol (RDP)** is a proprietary network protocol developed by Microsoft that provides a user with a graphical interface to connect to another computer over a network. RDP operates over **TCP port 3389** (and optionally UDP port 3389) to transmit mouse inputs, keystrokes, and compressed display frames.

### What is an AWS Security Group?
An **AWS Security Group** acts as a virtual, stateful firewall that controls inbound and outbound traffic at the network interface level for an EC2 instance. Being *stateful* means that if an inbound request is permitted (such as an incoming connection from your IP on port 3389), the corresponding response traffic is automatically allowed back out, regardless of outbound rules.

---

## 2. Cryptographic Password Retrieval Explained

When launching a Linux EC2 instance, administrators typically authenticate using public-key SSH cryptography directly. However, Windows Server instances authenticate users via an interactive username/password model. 

AWS bridges this gap using an automated asymmetric encryption workflow:

```mermaid
sequenceDiagram
    autonumber
    actor User as Cloud Engineer
    participant AWS as AWS EC2 Engine
    participant Instance as Windows EC2 Instance (EC2Launch)
    
    User->>AWS: Create/Select RSA Key Pair (.pem)
    Note over User,AWS: AWS keeps Public Key;<br/>User saves Private Key locally
    User->>AWS: Launch EC2 Instance with Key Pair
    AWS->>Instance: Inject Public Key into Instance Metadata
    Instance->>Instance: First Boot: Generate 32-character random Admin password
    Instance->>Instance: Encrypt password using RSA Public Key
    Instance->>AWS: Store ciphertext in EC2 Console Metadata
    User->>AWS: Request "Get Windows Password" via Console
    User->>AWS: Provide Private Key (.pem) in browser
    Note over User,AWS: Decryption happens in browser memory.<br/>Private key is NOT sent over the wire.
    AWS-->>User: Decrypted plaintext Administrator password
```

1. **Key Pair Association**: During instance launch, an RSA key pair is selected. AWS stores the **public key** in the instance metadata.
2. **Password Generation**: Upon first boot, the Windows initialization service (`EC2Launch` or `EC2Config`) generates a complex, random 32-character password for the local `Administrator` account.
3. **Asymmetric Encryption**: The service encrypts the random password using the public key and writes the resulting ciphertext to the EC2 console output.
4. **Local Decryption**: When the engineer requests the password via the AWS Management Console, the engineer supplies their private key (`.pem`). The browser utilizes the private key to decrypt the ciphertext locally, revealing the plaintext `Administrator` password.

---

## 3. Step-by-Step Implementation

### Step 1: Launch the Windows EC2 Instance

1. Sign in to the **AWS Management Console** and navigate to the **EC2 Dashboard**.
2. Click **Launch Instance**.
3. **Name and tags**:
   * Name: `Windows-RDP-Lab-Server`
4. **Application and OS Images (Amazon Machine Image)**:
   * Select **Windows**.
   * Choose: **Microsoft Windows Server 2022 Base** (or **2025 Base**) &mdash; verify it displays the *Free tier eligible* badge if using a free tier account.
5. **Instance type**:
   * Select `t2.micro` (or `t3.micro` depending on your AWS region's free tier eligibility).
6. **Key pair (login)**:
   * Select an existing key pair or click **Create new key pair**.
   * Key pair type: **RSA**.
   * Private key file format: `.pem` (recommended for standard OpenSSL/console decryption).
   * Save the downloaded `.pem` file to a secure directory on your local machine.

---

### Step 2: Configure the Security Group

In the **Network settings** section of the launch wizard:

1. Select **Create security group**.
2. **Security group name**: `windows-rdp-restricted-sg`.
3. **Description**: `Allow inbound RDP traffic restricted strictly to administrator public IP`.
4. Configure the **Inbound Security Group Rules**:

| Type | Protocol | Port Range | Source Type | Source CIDR | Description |
|:---|:---|:---|:---|:---|:---|
| **RDP** | TCP | `3389` | **My IP** | `<YOUR_PUBLIC_IP>/32` | Administrative RDP access from home/office network |

> [!WARNING]
> Never select **Anywhere-IPv4 (`0.0.0.0/0`)** for port 3389. Exposing RDP to the global Internet makes your server a target for brute-force attacks and automated credential stuffing.

5. **Configure Storage**:
   * Keep default volume size (typically 30 GiB General Purpose SSD `gp3` or `gp2`, which is within the AWS Free Tier threshold).
6. Click **Launch Instance**.

---

### Step 3: Retrieve the Decrypted Administrator Password

1. In the EC2 Console, select **Instances** and monitor your newly created instance.
2. Wait until **Instance State** is **Running** and **Status check** displays **2/2 checks passed** (this typically takes 3 to 5 minutes for Windows initialization).
3. Select the instance checkbox, click the **Connect** button at the top right.
4. Navigate to the **RDP client** tab.
5. Take note of the following parameters:
   * **Public DNS** / **Public IPv4 address** (e.g., `ec2-xx-xx-xx-xx.compute-1.amazonaws.com` or `54.x.x.x`).
   * **User name**: `Administrator`.
6. Click **Get password**.
7. Click **Browse** (or upload) and select the `.pem` private key file downloaded in Step 1.
8. Click **Decrypt Password**.
9. Copy the generated plaintext password to a secure temporary clipboard or password manager.

---

### Step 4: Install and Configure Remmina on Ubuntu Linux

On your local Ubuntu Linux virtual machine or workstation, install Remmina and its RDP plugin.

#### 1. Update Package Index & Install Remmina
Open your Linux terminal (`Ctrl + Alt + T`) and execute:

```bash
# Update repository package indexes
sudo apt update

# Install Remmina along with the FreeRDP plugin
sudo apt install -y remmina remmina-plugin-rdp
```

* **`sudo apt update`**: Refreshes the local database of available packages from Ubuntu mirrors.
* **`sudo apt install -y remmina remmina-plugin-rdp`**: Installs the core Remmina GUI application and the `FreeRDP` underlying protocol engine necessary for Microsoft RDP communication.

#### 2. Launch Remmina
Launch the application from the command line or desktop launcher:

```bash
remmina &
```

---

### Step 5: Establish the RDP Connection

1. In the Remmina main toolbar:
   * Set the protocol dropdown selector to **RDP**.
   * Enter the **Public IPv4 address** of your Windows EC2 instance into the quick-connect bar:
     ```text
     <WINDOWS_EC2_PUBLIC_IP>
     ```
   * Press **Enter** (or click the **+** icon in the top left to create a saved connection profile).

2. **Connection Profile Parameters** (if creating a saved profile):
   * **Name**: `AWS-Windows-Server-EC2`
   * **Protocol**: `RDP - Remote Desktop Protocol`
   * **Server**: `<WINDOWS_EC2_PUBLIC_IP>`
   * **Username**: `Administrator`
   * **Password**: `<DECRYPTED_WINDOWS_PASSWORD>`
   * **Color depth**: `True color (24 bpp)` or `High color (16 bpp)` (for optimal network performance)

3. Click **Connect** (or **Save and Connect**).

4. **Certificate Validation**:
   * A security dialog titled **Certificate verification** will appear, displaying the self-signed TLS certificate generated by Windows Server.
   * Verify the fingerprint and click **Accept Certificate** / **Trust**.

5. **Desktop Session Initialization**:
   * The Windows desktop interface will load inside Remmina.
   * The **Server Manager Dashboard** will appear automatically, confirming an active, authenticated administrative session.

---

## 4. Teardown & Resource Cleanup

To prevent ongoing AWS compute and storage charges after completing your lab:

1. Navigate to the **EC2 Dashboard** $\rightarrow$ **Instances**.
2. Select `Windows-RDP-Lab-Server`.
3. Click **Instance state** $\rightarrow$ **Terminate instance**.
4. Confirm termination when prompted.
5. Once terminated, navigate to **Security Groups** and delete `windows-rdp-restricted-sg` if no longer required.

