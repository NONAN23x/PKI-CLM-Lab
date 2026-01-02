# Microsoft Active Directory Certificate Services

A complete walkthrough for setting up a two-machine Active Directory and Certificate Services lab environment using Windows Server 2025 Evaluation Edition and VirtualBox.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [VirtualBox Setup](#virtualbox-setup)
3. [Setting up Primary Domain Controller](#primary-domain-controller-setup)
4. [Certificate Services Setup](#certificate-services-setup)
5. [Setting up Secondary CA Machine](#second-machine-setup)
6. [Post-Configuration Tasks](#post-configuration-tasks)


## Prerequisites

### Hardware Requirements
- **Host Machine**: Minimum 8 GB RAM (16 GB recommended)
- **CPU**: Multi-core processor with virtualization enabled
- **Storage**: 100+ GB available disk space

### Software Requirements
- **VirtualBox**: Version 7.0 or later
- **Windows Server 2025 Evaluation ISO**: Download from [Microsoft evaluation center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)

### Network Planning
- **Network Range**: 192.168.100.0/24 (can be modified)
- **Primary DC IP**: 192.168.100.10
- **Secondary Server IP**: 192.168.100.20
- **Domain Name**: `node.home` (can be modified)
- **NetBIOS Name**: `node` (can be modified)


## VirtualBox Setup

### Step 1: Create the Virtual NAT Network

1. Open VirtualBox and go to **File → Settings → Network**
2. Click the **+** icon to create a new NAT network adapter
3. Configure as follows:
   - **Name**: `LAB-Network`
   - **IPv4 Prefix**: `192.168.100.0/24`


### Step 2: Create the Primary DC Virtual Machine

1. Click **New** to create a new VM
2. Enter these settings:
   - **Name**: `DC01` (or `NODE-DC01`)
   - **Type**: Microsoft Windows
   - **Version**: Windows 2025 (Windows Server)
   - **Memory**: 4 GB (minimum)
   - **CPU Cores**: 2 (minimum)
   - **Storage**: 50 GB (dynamic allocation)

3. **Finish** and create the VM

4. Configure the VM network:
   - Right-click the VM and select **Settings**
   - Go to **Network**
   - **Adapter 1**: 
     - Attached to: `LAB-Network`
     - Promiscuous Mode: `Allow All`
   - Click **OK**

### Step 3: Create the Secondary Server Virtual Machine

Repeat Step 2 with these changes:
- **Name**: `SRV01` (or `Node-SRV01`)
- **Memory**: 2 GB (minimum)
- **CPU**: 2 cores (minimum)
- Keep the same **LAB-Network** attachment


## Primary Domain Controller Setup

### Step 1: Install Windows Server 2025

1. Start the **DC01** VM
2. Press any key to boot from ISO
3. Follow the Windows Setup wizard:
   - **Language, time, currency format**: Leave defaults
   - **Install now**
   - Select **Windows Server 2025 Standard Evaluation (Desktop Experience)** for GUI
   - Accept license terms
   - Select **Custom** installation
   - Choose the unallocated disk and proceed
4. Installation will take 10-15 minutes
5. Set the Administrator password when prompted (use a strong password)

### Step 2: Configure Network Settings

1. After login, **Server Manager** opens automatically
2. Go to **Local Server** in the left panel
3. Click the network adapter name (e.g., "Ethernet")
4. Right-click the adapter and select **Properties**
5. Double-click **Internet Protocol Version 4 (TCP/IPv4)**
6. Configure:
   - **IP Address**: `192.168.100.10`
   - **Subnet Mask**: `255.255.255.0`
   - **Default Gateway**: `192.168.100.1`
   - **Preferred DNS Server**: `192.168.100.10` (itself)
7. Click **OK** and close network settings
8. Rename the computer:
   - Right-click **This PC** → **Rename this PC**
   - Set to `DC01`
   - Restart the VM

### Step 3: Install Active Directory Domain Services

1. Open **Server Manager** (should be pinned to taskbar)
2. Click **Add roles and features**
3. Click **Next** until **Server Selection**
4. Ensure `DC01` is selected
5. On **Server Roles**, check **Active Directory Domain Services**
6. Click **Add Features** when prompted
7. Continue clicking **Next**
8. On **Confirmation**, click **Install**
9. Wait for installation to complete (2-3 minutes)
10. Click **Close**

### Step 4: Promote to Domain Controller

1. In **Server Manager**, look for the notification flag
2. Click **Promote this server to a domain controller**
3. On the **Deployment Configuration** screen:
   - Select **Add a new forest**
   - **Root domain name**: `NODE.home`
   - Click **Next**
4. On **Domain Controller Options**:
   - **Forest functional level**: `Windows Server 2025`
   - **Domain functional level**: `Windows Server 2025`
   - Check **DNS server**
   - **DSRM Password**: Set a strong password (different from Admin password)
   - Click **Next**
5. On **DNS Options**, click **Next**
6. On **Additional Options**:
   - **NetBIOS domain name**: `NODE`
   - Click **Next**
7. On **Paths**, leave defaults and click **Next**
8. On **Review Options**, click **Next**
9. On **Prerequisites Check**, click **Install**
   - This will take 5-10 minutes
10. The VM will automatically restart

### Step 5: Verify Active Directory Installation

1. After restart, login with `NODE\Administrator`
2. Open **Server Manager** → **Tools** → **Active Directory Users and Computers**
3. You should see:
   - `node.home` domain
   - Built-in containers (Users, Computers, Domain Controllers, etc.)
4. Open **DNS Manager** (**Tools** → **DNS**):
   - Expand the server
   - Expand **Forward Lookup Zones**
   - You should see `node.home` zone

## Certificate Services Setup

### Step 1: Install Active Directory Certificate Services

1. Open **Server Manager**
2. Click **Add roles and features**
3. Click **Next** until **Server Roles**
4. Check **Active Directory Certificate Services**
5. Click **Add Features** when prompted
6. Continue clicking **Next** until **Confirmation**
7. Click **Install**
8. Wait for completion and click **Close**

### Step 2: Configure ADCS

1. In **Server Manager**, click the notification flag
2. Select **Configure Active Directory Certificate Services on the destination server**
3. On **Credentials**, leave defaults and click **Next**
4. On **Role Services**, check:
   - **Certification Authority** (required)
   - **Certification Authority Web Enrollment** (optional but useful)
   - Click **Next**
5. On **Setup Type**, select **Enterprise CA**
6. Click **Next**
7. On **CA Type**, select **Root CA**
8. Click **Next**
9. On **Private Key**:
   - Select **Create a new private key**
   - Click **Next**
10. On **Cryptography**:
    - **Provider**: `RSA#Microsoft Software Key Storage Provider`
    - **Hash algorithm**: `SHA256`
    - **Key length**: `4096`
    - Click **Next**
11. On **CA Name**:
    - **Common name for the CA**: `NODE-ROOT-CA`
    - **Validity period**: `5` years
    - Click **Next**
12. On **Database**, leave defaults and click **Next**
13. On **Confirmation**, click **Configure**
14. Wait for configuration to complete (2-3 minutes)
15. Click **Close**

### Step 3: Create a Certificate Template for User Enrollment

1. Open **Server Manager** → **Tools** → **Certification Authority**
2. Expand the CA and right-click **Certificate Templates**
3. Click **Manage**
4. Right-click **User** template → **Duplicate Template**
5. On the **Compatibility** tab:
   - **Certification Authority**: Windows Server 2025
   - **Certificate recipient**: Windows 10/11 or higher
6. On the **General** tab:
   - **Template name**: `NODE-User`
   - **Validity period**: `2` years
   - **Renewal period**: `6` weeks
7. On the **Request Handling** tab:
   - **Allow private key to be exported**: Check this
8. On the **Subject Name** tab:
   - **Include e-mail name in subject name**: Check this
9. On the **Security** tab:
   - Select **Domain Users** and grant **Enroll** and **Autoenroll** permissions
10. Click **Apply** → **OK**
11. Close the Templates MMC
12. Back in **Certification Authority**:
    - Right-click **Certificate Templates**
    - Click **New** → **Certificate Template to Issue**
    - Select **NODE-User** and click **OK**

### Step 4: Verify ADCS Installation

1. Open **Certification Authority** in Server Manager Tools
2. You should see your CA listed with green checkmarks
3. Right-click the CA and select **Properties**
4. Verify the certificate is installed with correct validity period


## Second Machine Setup

### Step 1: Install Windows Server 2025 on SRV01

1. Start the **SRV01** VM
2. Follow the same installation steps as DC01 (see [Install Windows Server 2025](#step-1-install-windows-server-2025))
3. After reaching the desktop:
   - Set Administrator password
   - **Disable** Internet Explorer Enhanced Security (optional):
     - Open **Server Manager** → **Local Server**
     - Click **On** next to **IE Enhanced Security Configuration**
     - Disable for Administrators and Users

### Step 2: Configure Network Settings on SRV01

1. Configure the static IP:
   - **IP Address**: `192.168.100.20`
   - **Subnet Mask**: `255.255.255.0`
   - **Default Gateway**: `192.168.100.1`
   - **Preferred DNS Server**: `192.168.100.10` (DC01)
   - **Alternate DNS Server**: `192.168.100.10`

2. Rename the computer:
   - Right-click **This PC** → **Rename this PC**
   - Set to `SRV01`
   - Restart

### Step 3: Join SRV01 to the Domain

1. After restart, right-click **This PC** → **Properties**
2. Click **Advanced system settings** (right panel)
3. Click the **Computer Name** tab
4. Click **Change**
5. In the **Computer name** field, enter `SRV01`
6. Under **Member of**, select **Domain**
7. Type `node.home`
8. Click **OK**
9. When prompted, provide credentials:
   - **Username**: `NODE\Administrator`
   - **Password**: (your DC01 admin password)
10. Click **OK**
11. You'll see "Welcome to the node.home domain"
12. Click **OK** and restart the VM

### Step 4: Verify Domain Membership

After restart:
1. Login with `NODE\Administrator` (note the domain prefix)
2. Open **Server Manager** on DC01
3. Go to **Tools** → **Active Directory Users and Computers**
4. Navigate to **node.home** → **Computers**
5. You should see both `DC01` and `SRV01` listed

## Post-Configuration Tasks

### Task 1: Configure User Accounts

On **DC01**:

1. Open **Active Directory Users and Computers**
2. Right-click **nodehome** → **New** → **User**
3. Create test users:
   - **John Smith** (username: `jsmith`)
   - **Jane Doe** (username: `jdoe`)
4. For each user:
   - Set a temporary password
   - Check **User must change password at next logon**
   - Click **Finish**

### Task 2: Create Organizational Units (OUs)

On **DC01**:

1. In **Active Directory Users and Computers**
2. Right-click **node.home** → **New** → **Organizational Unit**
3. Create these OUs:
   - `IT`
   - `Sales`
   - `Finance`
   - `Workstations`
   - `Servers`

4. Move `SRV01` from Computers to **Servers** OU

### Task 3: Configure Group Policy

On **DC01**:

1. Open **Group Policy Management** (search in Server Manager → Tools)
2. Navigate to **Forest: node.home** → **Domains** → **node.home**
3. Create a new GPO:
   - Right-click **node.home** → **Create a GPO in this domain, and Link it here**
   - Name: `Password Policy`
4. Edit the GPO:
   - Right-click **Password Policy** → **Edit**
   - Navigate to: **Computer Configuration** → **Policies** → **Windows Settings** → **Security Settings** → **Account Policies** → **Password Policy**
   - Set:
     - **Maximum password age**: 90 days
     - **Minimum password length**: 12 characters
     - **Password must meet complexity requirements**: Enabled
   - Close the editor

### Task 4: Request a Certificate on SRV01

On **SRV01**:

1. Open **Certification Authority** (search in Start menu)
2. Click **Request a certificate**
3. Click **Request**
4. Select **User** or **NODE-User** template
5. Click **Enroll**
6. You should see "Certificate Issued" message
7. Click **View the properties of the issued certificate**

### Task 5: Configure SSL for Web Services (Optional)

If you installed **Certification Authority Web Enrollment**:

1. On **DC01**, go to **Server Manager** → **Local Server** → **Network connections**
2. Note the DC's IP address (192.168.100.10)
3. From **SRV01**, open a web browser
4. Navigate to `http://192.168.100.10/certsrv`
5. You can request and retrieve certificates via the web interface

### Task 6: Enable File Server Role on SRV01 (Optional)

For a more realistic environment:

1. On **SRV01**, open **Server Manager**
2. Click **Add roles and features**
3. Add **File and Storage Services** role
4. On **Role Services**, check:
   - **File Server**
   - **File Server Resource Manager**
5. Complete the installation
6. Create shared folders for testing group policy and file permissions


## Useful Commands and Tools

### PowerShell Commands

```powershell
# Check domain information
$env:USERDOMAIN
$env:USERDNSDOMAIN
whoami /groups

# Force group policy update
gpupdate /force

# Test DNS resolution
nslookup node.home 192.168.100.10
nslookup DC01 192.168.100.10

# View certificate store
certmgr.msc

# Check DC replication
repadmin /syncall /force

# View system event logs
eventvwr.msc
```

### MMC Tools to Add

Press `Windows+R`, type `mmc`, and add these snap-ins:
- **Active Directory Users and Computers** (`dsa.msc`)
- **Active Directory Sites and Services** (`dssite.msc`)
- **Certification Authority** (`certsrv.msc`)
- **Group Policy Management** (`gpmc.msc`)
- **Event Viewer** (`eventvwr.msc`)
- **Services** (`services.msc`)
- **Device Manager** (`devmgmt.msc`)


## Troubleshooting

### Issue: SRV01 Cannot Join Domain

**Solution**:
1. Verify DNS is pointing to DC01's IP (192.168.100.10)
2. On DC01, check DNS is running: `Services.msc` → look for "DNS Server"
3. Ping DC01 from SRV01: `ping 192.168.100.10`
4. Ensure both VMs are on the same network (`LAB-Network`)
5. Check firewall isn't blocking ports 53 (DNS), 389 (LDAP)

### Issue: Certificate Enrollment Fails

**Solution**:
1. Verify ADCS is running on DC01
2. Check certificate template permissions (Domain Users need Enroll permission)
3. Restart the DC and wait 5 minutes for template replication
4. Try manual enrollment: Run `certmgr.msc` and use Request New Certificate wizard

### Issue: VMs Lose Connectivity After Restart

**Solution**:
1. Verify both VMs are attached to `LAB-Network` in VirtualBox
2. Restart the VM's network adapter or restart the VM
3. Check that VirtualBox network configuration is still present
4. Re-run `ipconfig /all` and reconfigure if needed

### Issue: DC Boot Hangs After Promotion

**Solution**:
1. Wait 10-15 minutes (replication can take time)
2. If still hung, check the log: `C:\Windows\Logs\Dcpromo.log`
3. Try rebooting the VM
4. Disable IPv6 if issues persist


## Next Steps

### Advance Your Lab

1. **Add Subordinate CA**: Create a subordinate CA on SRV01 for better testing scenarios
2. **Configure AutoEnrollment**: Set up automatic certificate renewal for computers
3. **Implement PKI Hierarchy**: Practice multi-level CA structures
4. **Add Workstation**: Add a Windows 10/11 VM as a domain-joined workstation
5. **Configure Web Enrollment**: Test certificate requests via web portal
6. **Implement SCEP**: Set up Simple Certificate Enrollment Protocol
7. **Add Web Server Role**: Install IIS and configure HTTPS certificates

### Lab Maintenance

- **Regular Backups**: Snapshot VMs regularly in VirtualBox
- **Monitor Logs**: Check Event Viewer for errors and warnings
- **Certificate Renewal**: Plan for CA certificate renewal (yearly check)
- **Updates**: Apply Windows updates periodically
- **Documentation**: Keep detailed notes of your configuration


## Security Best Practices

1. **Use complex passwords** for all accounts (minimum 12 characters)
2. **Enable Windows Firewall** on all servers
3. **Regularly patch** Windows and applications
4. **Limit RDP access** to specific IP ranges
5. **Audit certificate requests** and issuance
6. **Backup CA database** regularly
7. **Test disaster recovery** procedures
8. **Monitor CA logs** for unauthorized activity


## Reference URLs

- [Windows Server 2025 Documentation](https://learn.microsoft.com/en-us/windows-server/)
- [Active Directory Certificate Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/active-directory-certificate-services-overview)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)
- [Microsoft PKI Deployment Guide](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/plan-and-design-your-pki-infrastructure)


## Lab Configuration Summary

| Component | Details |
|-----------|---------|
| **Host VM Memory** | 8 GB per VM (16 GB total) |
| **Host VM Storage** | 100 GB per VM |
| **Network** | 192.168.100.0/24 (LAB-Network) |
| **Primary DC** | DC01 @ 192.168.100.10 |
| **Secondary Server** | SRV01 @ 192.168.100.20 |
| **Domain** | node.home (NODE) |
| **OS** | Windows Server 2025 Evaluation |
| **CA Type** | Enterprise Root CA |
| **CA Name** | NODE-ROOT-CA |
| **CA Validity** | 5 years |
