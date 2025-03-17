# IT-Infra
Tools and Common Issues

# Contents

- [OpenZiti](#openziti)
- [SmartDeploy](#smartdeploy)
- [NinjaOne](#ninjaone)
- [Proxmox Virtual Environment (PVE)](#proxmox-virtual-environment-pve)
- [Proxmox Backup Server (PBS)](#proxmox-backup-server-pbs)
- [Synology Backup Server](#synology-backup-server)
- [Active Backup for Office 365](#active-backup-for-office-365)
- [Docker](#docker)
- [Kubernetes](#kubernetes)
- [S3](#s3)
- [Zabbix](#zabbix)
- [Chocolatey](#chocolatey)
- [pfSense](#pfsense)
- [FreePBX](#freepbx)
- [CephFS](#cephfs)
- [LAPS](#laps)
- [Active Directory](#active-directory)

---

# OpenZiti

**OpenZiti** is an open-source, zero-trust networking platform that provides secure, private, and scalable connectivity for applications and services. While it is a powerful solution, users may encounter issues. 

  - **Controller**: The OpenZiti Controller is the central management component that defines policies, manages identities, and controls access to the network.

  - **Edge Routers**: These are the components that handle the actual routing of traffic within the OpenZiti network. They establish secure connections between devices and services.

  - **Clients**: OpenZiti clients (also known as "Ziti Edge Clients") are software components that run on user devices (e.g., laptops, smartphones) or servers. These clients authenticate with the OpenZiti Controller and establish secure connections to the Edge Routers.

  - **Services**: These are the applications or resources that you want to secure and make accessible over the OpenZiti network. Services can be anything from web applications to databases.

**OpenZiti Documentation:** https://openziti.io/docs/reference/

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with OpenZiti

### 1. Ziti Edge Router (ZER) Not Starting

- **Symptoms:**
  - Ziti Edge Router fails to start or crashes.
  - Error messages in logs (`/var/log/ziti/`).

- **Resolution:**
  - Verify the configuration file (`ziti-router.yml`) for errors.
  - Check for sufficient resources (CPU, memory, disk).
  - Ensure the Ziti Controller is running and accessible:

    ```bash
    curl http://<controller_ip>:<port>/edge/client/v1/version
    ```

### 2. Ziti Controller Issues

- **Symptoms:**
  - Ziti Controller fails to start or becomes unresponsive.
  - Error messages like `database connection failed` or `service unavailable`.

- **Resolution:**
  - Verify database connectivity and credentials:

    ```bash
    psql -h <db_host> -U <db_user> -d <db_name>
    ```
  - Check controller logs for errors (`/var/log/ziti/`).
  - Restart the Ziti Controller service:

    ```bash
    systemctl restart ziti-controller
    ```

### 3. Client Connection Issues

- **Symptoms:**
  - Clients cannot connect to the Ziti network.
  - Error messages like `connection refused` or `authentication failed`.

- **Resolution:**
  - Verify client configuration (`ziti.json`):
    - Check `ztAPI` URL and credentials.
  - Ensure the Ziti Edge Router is running and accessible.
  - Check firewall rules to allow traffic on required ports (default **3022** for ZER).

### 4. Service Unreachable

- **Symptoms:**
  - Services hosted on the Ziti network are unreachable.
  - Error messages like `service not found` or `timeout`.

- **Resolution:**
  - Verify service configuration in the Ziti Controller:
    - Check `intercept` and `host` settings.
  - Ensure the service is running and accessible locally.
  - Use `ziti` CLI to test service connectivity:

    ```bash
    ziti service list
    ziti service test <service_name>
    ```

### 5. High Latency or Poor Performance

- **Symptoms:**
  - High latency or slow performance on the Ziti network.
  - Error messages like `timeout` or `connection reset`.

- **Resolution:**
  - Check network performance between clients and Ziti Edge Routers:

    ```bash
    ping <zer_ip>
    traceroute <zer_ip>
    ```
  - Optimize Ziti Edge Router placement for better latency.
  - Use multiple Ziti Edge Routers for load balancing.

### 6. Authentication Issues

- **Symptoms:**
  - Clients or services fail to authenticate.
  - Error messages like `authentication failed` or `invalid credentials`.

- **Resolution:**
  - Verify client and service credentials in the Ziti Controller.
  - Check for expired or revoked certificates.
  - Reissue certificates if necessary:

    ```bash
    ziti edge create identity <identity_name> --role-attributes <roles>
    ```

### 7. Configuration Sync Issues

- **Symptoms:**
  - Configuration changes are not propagated to clients or routers.
  - Error messages like `configuration out of sync`.

- **Resolution:**
  - Verify Ziti Controller logs for sync issues.
  - Restart Ziti Edge Routers to force configuration updates:

    ```bash
    systemctl restart ziti-router
    ```
  - Use `ziti` CLI to manually sync configurations:

    ```bash
    ziti edge sync
    ```

### 8. Logging and Monitoring Issues

- **Symptoms:**
  - Logs are not populated or show errors.
  - Monitoring data is missing or incorrect.

- **Resolution:**
  - Verify logging settings in Ziti Controller and Edge Router configurations.
  - Check disk space and log rotation settings.
  - Enable and configure monitoring tools (e.g., Prometheus, Grafana).

### 9. Upgrade Issues

- **Symptoms:**
  - Problems after upgrading OpenZiti components.
  - Error messages like `incompatible version` or `feature not supported`.

- **Resolution:**
  - Follow the official OpenZiti upgrade guide for your version.
  - Check component compatibility and health:

    ```bash
    ziti version
    ```
  - Roll back to the previous version if necessary.

### 10. Network Policy Issues

- **Symptoms:**
  - Traffic is blocked or allowed unexpectedly.
  - Network policies do not behave as configured.

- **Resolution:**
  - Verify network policy configurations in the Ziti Controller:
    - Check `service policies`, `edge router policies`, and `identity policies`.
  - Use `ziti` CLI to test policy enforcement:

    ```bash
    ziti policy list
    ziti policy test <policy_name>
    ```

## General Troubleshooting Tips

- **Check Logs:** Review Ziti Controller and Edge Router logs (`/var/log/ziti/`) for errors.
- **Test Connectivity:** Use tools like `ping`, `traceroute`, and `curl` to diagnose network issues.
- **Update OpenZiti:** Ensure OpenZiti components are updated to the latest version:

  ```bash
  ziti update
  ```
- **Community Support:** https://openziti.discourse.group/
  
[go to Contents](#contents)

---

# SmartDeploy

**SmartDeploy**, powered by PDQ.com, is a tool for automating OS deployment and imaging.

**Features**
 - sysprep can be handle by SmartDeploy
 - Can choose to deploy Standard App or Custom App
https://smartdeploy.pdq.com/hc/en-us/articles/12982091499163-Create-a-Custom-Application-Pack

**SmartDeploy Documentation:** https://smartdeploy.pdq.com/hc/en-us/articles/21577398522267-Getting-Started-with-SmartDeploy

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with SmartDeploy

### 1. Deployment Failures

- **Symptoms:**
  - Deployment process halts or fails midway.
  - Error messages during deployment (e.g., "Failed to apply image").
  - Target machine does not boot into the deployed OS.

- **Resolution:**
  - Verify network connectivity between the SmartDeploy server and target machines.
  - Ensure the target machine meets hardware requirements for the OS being deployed.
  - Check the deployment logs (`C:\\Windows\\Panther\\UnattendGC\\setupact.log`) for errors.
  - Update SmartDeploy to the latest version.

### 2. Driver Issues

- **Symptoms:**
  - Target machine has missing or incorrect drivers after deployment.
  - Devices (e.g., network cards, storage controllers) not functioning properly.

- **Resolution:**
  - Use SmartDeploy's **Platform Pack** for the specific hardware model.
  - Manually inject missing drivers into the image using the **Driver Injection Wizard**.
  - Verify that the correct drivers are included in the deployment package.

### 3. Network Boot (PXE) Issues

- **Symptoms:**
  - Target machine fails to boot via PXE.
  - PXE boot process hangs or displays errors (e.g., "PXE-E53: No boot filename received").

- **Resolution:**
  - Ensure the DHCP server is configured correctly (e.g., options 66 and 67 for PXE boot).
  - Verify that the SmartDeploy PXE service is running.
  - Check network settings and firewall rules to allow PXE traffic.

### 4. Image Creation Problems

- **Symptoms:**
  - Unable to capture an image from a reference machine.
  - Captured image is corrupted or incomplete.

- **Resolution:**
  - Ensure the reference machine is properly prepared (e.g., sysprepped for Windows).
  - Use the **Capture Wizard** in SmartDeploy to create the image.
  - Verify that the reference machine has sufficient disk space for the image.

### 5. Slow Deployment Performance

- **Symptoms:**
  - Deployment process takes longer than expected.
  - High network or disk usage during deployment.

- **Resolution:**
  - Optimize network bandwidth (e.g., use a dedicated deployment network).
  - Use faster storage (e.g., SSDs) for the SmartDeploy server.
  - Enable multicast deployment for multiple machines.

### 6. Licensing and Activation Issues

- **Symptoms:**
  - Deployed OS fails to activate.
  - Licensing errors during or after deployment.

- **Resolution:**
  - Ensure the correct product key is included in the deployment package.
  - Verify that the target machine is connected to the internet for activation.
  - Use Volume Licensing (KMS/MAK) for enterprise deployments.

### 7. Unattended Installation Failures

- **Symptoms:**
  - Deployment requires manual intervention (e.g., prompts for user input).
  - Unattended installation settings are ignored.

- **Resolution:**
  - Verify the **answer file** (`unattend.xml`) is correctly configured.
  - Use the **Answer File Generator** in SmartDeploy to create the file.
  - Test the unattended installation on a single machine before deploying to multiple machines.

### 8. Software Installation Issues

- **Symptoms:**
  - Applications fail to install during deployment.
  - Missing or misconfigured software packages.

- **Resolution:**
  - Verify that the software packages are correctly added to the deployment package.
  - Test software installation on a reference machine before deployment.
  - Check for compatibility issues between the software and the deployed OS.

### 9. Disk Partitioning Problems

- **Symptoms:**
  - Incorrect disk partitioning during deployment.
  - Target machine fails to boot due to partition errors.

- **Resolution:**
  - Verify the disk partitioning settings in the deployment package.
  - Use the **Disk Wiping** feature to ensure a clean partition layout.
  - Test deployment on a single machine before deploying to multiple machines.

### 10. Post-Deployment Configuration Issues

- **Symptoms:**
  - Deployed machines have incorrect settings (e.g., hostname, domain join).
  - Custom scripts or configurations are not applied.

- **Resolution:**
  - Verify post-deployment scripts and configurations in the deployment package.
  - Use the **Post-Deployment Task** feature in SmartDeploy.
  - Test post-deployment configurations on a reference machine.

## General Troubleshooting Tips

- **Check Logs:** Review deployment logs (`C:\\Windows\\Panther\\UnattendGC\\setupact.log`) for errors.
- **Test on a Single Machine:** Always test deployments on a single machine before deploying to multiple machines.
- **Update SmartDeploy:** Ensure you are using the latest version of SmartDeploy.
- **Community Support:** https://discord.com/invite/pdq

[go to Contents](#contents)

---

# NinjaOne

**NinjaOne** (formerly NinjaRMM) is a remote monitoring and management (RMM) platform used for IT management, automation, and monitoring. While it is a powerful tool, users may encounter issues. 

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with NinjaOne

### 1. Agent Installation Failures

- **Symptoms:**
  - NinjaOne agent fails to install on endpoints.
  - Error messages during installation (e.g., "Installation failed").

- **Resolution:**
  - Ensure the endpoint meets the minimum system requirements.
  - Check for conflicting software (e.g., antivirus or other RMM tools).
  - Verify network connectivity and firewall rules (ports **443** and **9991** for NinjaOne).
  - Use the **Manual Installer** for troubleshooting.

### 2. Agent Communication Issues

- **Symptoms:**
  - Endpoints show as offline in the NinjaOne dashboard.
  - Alerts or monitoring data are not updated.

- **Resolution:**
  - Verify network connectivity between the endpoint and NinjaOne servers.
  - Check firewall rules to allow outbound traffic on ports **443** and **9991**.
  - Restart the NinjaOne agent service on the endpoint:

    ```bash
    net stop NinjaRMMAgent
    net start NinjaRMMAgent
    ```

### 3. Performance Monitoring Issues

- **Symptoms:**
  - Performance data (e.g., CPU, memory, disk usage) is missing or inaccurate.
  - Alerts are not triggered for performance thresholds.

- **Resolution:**
  - Verify that the NinjaOne agent is running and communicating.
  - Check for resource bottlenecks on the endpoint (e.g., high CPU or memory usage).
  - Update the NinjaOne agent to the latest version.

### 4. Patch Management Failures

- **Symptoms:**
  - Patches fail to install on endpoints.
  - Patch status is not updated in the NinjaOne dashboard.

- **Resolution:**
  - Verify that the endpoint has internet access to download patches.
  - Check for conflicting software (e.g., WSUS or third-party patch management tools).
  - Review patch policies in the NinjaOne dashboard for misconfigurations.

### 5. Script Execution Failures

- **Symptoms:**
  - Scripts fail to run on endpoints.
  - Script output is missing or incorrect.

- **Resolution:**
  - Verify that the script is compatible with the endpoint's OS.
  - Check script permissions and execution policies (e.g., PowerShell execution policy).
  - Test the script manually on the endpoint to identify issues.

### 6. Alerting and Notification Issues

- **Symptoms:**
  - Alerts are not triggered for configured thresholds.
  - Notifications are not sent to users.

- **Resolution:**
  - Verify alert thresholds and notification settings in the NinjaOne dashboard.
  - Check for email or SMS delivery issues (e.g., spam filters or incorrect contact information).
  - Test alerts by manually triggering them.

### 7. Remote Control Issues

- **Symptoms:**
  - Unable to establish a remote control session.
  - Remote control session is slow or unresponsive.

- **Resolution:**
  - Verify that the NinjaOne agent is running and communicating.
  - Check network connectivity and bandwidth between the technician and endpoint.
  - Use the **Web Remote** or **Splashtop** options for troubleshooting.

### 8. Backup Monitoring Issues

- **Symptoms:**
  - Backup status is not updated in the NinjaOne dashboard.
  - Backup failures are not reported.

- **Resolution:**
  - Verify that the backup software is configured correctly.
  - Check for backup logs or errors on the endpoint.
  - Ensure the NinjaOne agent has permissions to monitor the backup software.

### 9. Integration Issues

- **Symptoms:**
  - Third-party integrations (e.g., PSA, IT Glue) fail to sync data.
  - Integration errors in the NinjaOne dashboard.

- **Resolution:**
  - Verify API keys and credentials for the integration.
  - Check for updates or changes in the third-party software.
  - Contact NinjaOne support for assistance with integration issues.

### 10. Dashboard or UI Issues

- **Symptoms:**
  - NinjaOne dashboard is slow or unresponsive.
  - UI elements are missing or not functioning.

- **Resolution:**
  - Clear the browser cache or try a different browser.
  - Verify that the browser is supported (e.g., Chrome, Firefox, Edge).
  - Check for network latency or connectivity issues.

## General Troubleshooting Tips

- **Check Logs:** Review NinjaOne agent logs (`C:\\ProgramData\\NinjaRMMAgent\\logs`) for errors.
- **Test on a Single Endpoint:** Always test configurations on a single endpoint before applying them to multiple endpoints.
- **Update the Agent:** Ensure the NinjaOne agent is updated to the latest version.
- **Community Support:** https://discord.com/invite/ninjaone

[go to Contents](#contents)

---

# Proxmox Virtual Environment (PVE)

**Proxmox Virtual Environment (PVE)** is an open-source server management platform for enterprise virtualization.

**Features**
 - Supports both full VM and container based virtualization on the same host
 - Kernel-based Virtual Machine (KVM) for virtual machines
 - Linux Containers (LXC) for containers
 - Support clustering for HALB

**PVE Documentation:** https://pve.proxmox.com/pve-docs/

[YouTube video: Proxmox 8.0!](https://www.youtube.com/watch?v=sZcOlW-DwrU)

Below are some common problems reported by users:

## Proxmox Virtual Environment (PVE) Issues

### 1. Cluster Communication Issues

- **Symptoms:**
  - Nodes show as offline or disconnected in the web interface.
  - Quorum errors or split-brain scenarios.
  - VMs/CTs fail to migrate or start.

- **Resolution:**
  - Check network connectivity and firewall rules for Corosync traffic (ports **5404-5405 UDP**).
  - Verify NTP synchronization on all nodes.
  - Restart Corosync and PVE cluster services:

    ```bash
    systemctl restart corosync
    systemctl restart pve-cluster
    ```

### 2. Storage Problems

- **Symptoms:**
  - Slow VM/CT performance or high latency.
  - Storage-related errors in logs (`/var/log/syslog`).
  - Inability to create or resize disks.

- **Resolution:**
  - Optimize storage configuration (e.g., use SSDs for Ceph OSDs).
  - Check network latency and bandwidth for shared storage.
  - Monitor disk I/O using `iostat` or `iotop`.

### 3. Networking Issues

- **Symptoms:**
  - VMs/CTs cannot access the network.
  - Network interfaces show errors or packet loss.
  - Misconfigured VLANs or bridges.

- **Resolution:**
  - Verify network configuration in `/etc/network/interfaces`.
  - Test connectivity using `ping`, `traceroute`, and `tcpdump`.
  - Restart networking services:

    ```bash
    systemctl restart networking
    ```

### 4. VM/CT Issues

- **Symptoms:**
  - VMs/CTs fail to start or crash unexpectedly.
  - High CPU or memory usage.
  - Disk-related errors in logs.

- **Resolution:**
  - Check disk images for corruption (use `qm disk rescan` for VMs).
  - Allocate sufficient resources (CPU, RAM, disk I/O).
  - Disable unnecessary snapshots or merge them.

### 5. Backup and Restore Problems

- **Symptoms:**
  - Backup jobs fail or take too long.
  - Restore operations fail or produce errors.

- **Resolution:**
  - Ensure sufficient storage space on the backup target.
  - Verify network connectivity and bandwidth.
  - Check permissions for backup storage.

### 6. Upgrade and Compatibility Issues

- **Symptoms:**
  - System instability or failures after upgrading.
  - Broken dependencies or unsupported configurations.

- **Resolution:**
  - Back up configurations before upgrading.
  - Check the Proxmox forum for known issues.
  - Reinstall or downgrade problematic packages.

### 7. Resource Management

- **Symptoms:**
  - High CPU, memory, or disk I/O usage.
  - VMs/CTs experience performance degradation.

- **Resolution:**
  - Monitor resource usage using `htop`, `vmstat`, or Proxmox's web interface.
  - Adjust resource limits for VMs/CTs.
  - Use resource pools to allocate resources effectively.

### 8. User and Permission Issues

- **Symptoms:**
  - Users cannot log in or access resources.
  - Permission errors when managing VMs/CTs or storage.

- **Resolution:**
  - Verify user roles and permissions in the Proxmox web interface.
  - Check LDAP/AD configuration for authentication issues.
  - Ensure API tokens or keys are correctly configured.

### 9. High Availability (HA) Failures

- **Symptoms:**
  - HA services fail to migrate or restart VMs/CTs.
  - Quorum errors or node failures.

- **Resolution:**
  - Verify HA configuration in `/etc/pve/ha/`.
  - Ensure quorum is maintained and all nodes are online.
  - Test HA failover scenarios in a controlled environment.

### 10. Ceph Integration Issues

- **Symptoms:**
  - Slow or unstable Ceph performance.
  - Ceph OSDs failing or going offline.

- **Resolution:**
  - Monitor Ceph health using `ceph -s`.
  - Ensure proper network configuration for Ceph.
  - Replace failing OSDs or adjust CRUSH maps.

---

# Proxmox Backup Server (PBS)

**Proxmox Backup Server (PBS)** is an enterprise-class, client-server backup package which provides strong encryption without compromising on speed or flexibility. The Backup Server is based on a minimal Debian GNU/Linux distribution with built-in ZFS support. Thus, it can be installed on bare-metal, using the downloadable ISO image.

**Features**
 - Incremental backups. after the initial backup, only changes since the last backup are saved.
    - Saves storage space
    - Reduces the time and bandwidth needed for subsequent backups

**PBS Documentation:** https://pbs.proxmox.com/docs/

Below are some common problems reported by users:

### 1. Backup Failures

- **Symptoms:**
  - Backup jobs fail with errors.
  - Incomplete or corrupted backups.

- **Resolution:**
  - Check network connectivity between PVE and PBS.
  - Verify storage space and permissions on the PBS server.
  - Review PBS logs (`/var/log/proxmox-backup/`) for errors.

### 2. Slow Backup/Restore Performance

- **Symptoms:**
  - Backup or restore operations take too long.
  - High network or disk I/O usage.

- **Resolution:**
  - Optimize network bandwidth (e.g., use a dedicated backup network).
  - Use faster storage (e.g., SSDs) for PBS.
  - Adjust compression and deduplication settings.

### 3. Data Integrity Issues

- **Symptoms:**
  - Corrupted backups or checksum errors.
  - Restore operations fail.

- **Resolution:**
  - Verify backup integrity using the `proxmox-backup-client` tool.
  - Check for hardware issues (e.g., failing disks or RAM).
  - Regularly test backups by performing restores.

### 4. Storage Management

- **Symptoms:**
  - Running out of disk space on the PBS server.
  - Backup jobs fail due to insufficient storage.

- **Resolution:**
  - Implement retention policies to automatically clean up old backups.
  - Add more storage or migrate backups to a larger disk.

### 5. Authentication and Access Problems

- **Symptoms:**
  - PBS authentication failures.
  - Users cannot access backups or manage PBS.

- **Resolution:**
  - Verify API keys, tokens, and user permissions.
  - Check PBS configuration files (`/etc/proxmox-backup/`).

### 6. Upgrade Issues

- **Symptoms:**
  - System instability or failures after upgrading.
  - Broken backups or compatibility issues.

- **Resolution:**
  - Back up configurations before upgrading.
  - Check the Proxmox forum for known issues.
  - Reinstall or downgrade problematic packages.

### 7. Snapshot and Retention Policy Problems

- **Symptoms:**
  - Snapshots not being cleaned up.
  - Backup storage filling up unexpectedly.

- **Resolution:**
  - Verify retention policy settings in the PBS web interface.
  - Manually clean up old snapshots if needed.

### 8. Network Configuration Issues

- **Symptoms:**
  - Backup jobs fail due to network errors.
  - Slow backup or restore speeds.

- **Resolution:**
  - Verify network configuration in `/etc/network/interfaces`.
  - Ensure firewall rules allow backup traffic (default port **8007 TCP**).

### 9. Hardware Compatibility

- **Symptoms:**
  - Slow performance or frequent hardware failures.
  - High disk or CPU usage on the PBS server.

- **Resolution:**
  - Use recommended hardware (e.g., SSDs for storage, sufficient RAM).
  - Monitor hardware health using tools like `smartctl`.

### 10. Restore Failures

- **Symptoms:**
  - Inability to restore VMs/CTs.
  - Errors during restore operations.

- **Resolution:**
  - Verify backup integrity using the `proxmox-backup-client` tool.
  - Ensure the target environment matches the source (e.g., storage configuration).

## General Troubleshooting Tips

- **Check Logs:** Always review logs (`/var/log/syslog`, `journalctl`, or Proxmox-specific logs) for errors.
- **Test Backups:** Regularly test backups by performing restores to ensure data integrity.
- **Monitor Resources:** Use monitoring tools to identify and resolve resource bottlenecks.
- **Community Support:** https://forum.proxmox.com/

[go to Contents](#contents)

---

# Synology Backup Server

**Synology Backup Server** (often used with **Active Backup for Business** or **Hyper Backup**) is a robust solution for backing up data, virtual machines, and physical servers. While it is reliable, users may encounter issues. Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Synology Backup Server

### 1. Backup Job Failures

- **Symptoms:**
  - Backup jobs fail to start or complete.
  - Error messages in the Synology Backup Server logs.

- **Resolution:**
  - Verify that the source device (e.g., VM, physical server) is accessible over the network.
  - Check for sufficient storage space on the Synology NAS.
  - Review backup job settings (e.g., schedule, retention policies).
  - Update Synology DSM and the backup application to the latest version.

### 2. Slow Backup Performance

- **Symptoms:**
  - Backup jobs take longer than expected.
  - High CPU or network usage during backups.

- **Resolution:**
  - Optimize network bandwidth (e.g., use a dedicated backup network).
  - Use faster storage (e.g., SSDs) on the Synology NAS.
  - Enable **deduplication** and **compression** to reduce backup size.
  - Schedule backups during off-peak hours.

### 3. Restore Failures

- **Symptoms:**
  - Unable to restore files or systems from backups.
  - Restore process halts or produces errors.

- **Resolution:**
  - Verify that the backup files are not corrupted.
  - Check for sufficient storage space on the target device.
  - Test restores regularly to ensure backup integrity.

### 4. VM Backup Issues (Active Backup for Business)

- **Symptoms:**
  - VM backups fail or are incomplete.
  - Error messages related to VMware/Hyper-V integration.

- **Resolution:**
  - Verify that the Synology NAS has proper permissions to access the VM host.
  - Check for sufficient resources (e.g., CPU, RAM) on the VM host.
  - Update VMware/Hyper-V integration tools and drivers.

### 5. Hyper Backup Issues

- **Symptoms:**
  - Hyper Backup jobs fail or produce errors.
  - Backup files are missing or corrupted.

- **Resolution:**
  - Verify that the destination (e.g., cloud, external drive) is accessible.
  - Check for sufficient storage space on the destination.
  - Review backup job settings (e.g., encryption, compression).

### 6. Network Connectivity Issues

- **Symptoms:**
  - Backup jobs fail due to network errors.
  - Synology NAS is unreachable from the source device.

- **Resolution:**
  - Verify network connectivity using `ping` or `traceroute`.
  - Check firewall rules to allow backup traffic (default port **5001** for DSM).
  - Ensure the Synology NAS and source device are on the same network.

### 7. Storage Pool or Volume Issues

- **Symptoms:**
  - Backup jobs fail due to storage errors.
  - Storage pool or volume shows as degraded or critical.

- **Resolution:**
  - Check the storage pool status in **Storage Manager**.
  - Replace failing drives and rebuild the storage pool.
  - Ensure proper RAID configuration for redundancy.

### 8. Permission Issues

- **Symptoms:**
  - Backup jobs fail due to permission errors.
  - Unable to access files or folders on the source device.

- **Resolution:**
  - Verify that the Synology NAS has proper permissions to access the source device.
  - Check user roles and permissions in **Control Panel > User & Group**.
  - Use a dedicated backup account with sufficient privileges.

### 9. Cloud Backup Issues (Hyper Backup)

- **Symptoms:**
  - Cloud backup jobs fail or are incomplete.
  - Error messages related to cloud storage providers.

- **Resolution:**
  - Verify that the cloud storage account is accessible and has sufficient space.
  - Check for API rate limits or throttling from the cloud provider.
  - Update Hyper Backup and DSM to the latest version.

### 10. Backup Verification Failures

- **Symptoms:**
  - Backup verification fails or produces errors.
  - Backup integrity cannot be confirmed.

- **Resolution:**
  - Verify that the backup files are not corrupted.
  - Test restores regularly to ensure backup integrity.
  - Use checksum or hash verification tools if available.

## General Troubleshooting Tips

- **Check Logs:** Review Synology Backup Server logs (**Log Center**) for errors.
- **Test on a Single Device:** Always test backup and restore processes on a single device before scaling.
- **Update Software:** Ensure DSM and backup applications are updated to the latest version.
- **Community Support:** https://community.synology.com/enu

[go to Contents](#contents)

---

# Active Backup for Office 365

**Active Backup for Office 365** is a Synology solution designed to back up Microsoft 365 data (e.g., Exchange Online, SharePoint, OneDrive, and Teams). 

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Active Backup for Office 365

### 1. Backup Job Failures

- **Symptoms:**
  - Backup jobs fail to start or complete.
  - Error messages in the Active Backup for Office 365 logs.

- **Resolution:**
  - Verify that the Synology NAS has internet access and can reach Microsoft 365 services.
  - Check for sufficient storage space on the Synology NAS.
  - Review backup job settings (e.g., schedule, retention policies).
  - Update Synology DSM and Active Backup for Office 365 to the latest version.

### 2. Authentication Issues

- **Symptoms:**
  - Backup jobs fail due to authentication errors.
  - Unable to connect to Microsoft 365.

- **Resolution:**
  - Verify that the Microsoft 365 account used for backup has the necessary permissions (e.g., **Global Admin** or **Application Impersonation** roles).
  - Re-authenticate the Microsoft 365 account in Active Backup for Office 365.
  - Check for changes in Microsoft 365 tenant settings (e.g., multi-factor authentication, conditional access policies).

### 3. Slow Backup Performance

- **Symptoms:**
  - Backup jobs take longer than expected.
  - High CPU or network usage during backups.

- **Resolution:**
  - Optimize network bandwidth (e.g., use a dedicated backup network).
  - Use faster storage (e.g., SSDs) on the Synology NAS.
  - Schedule backups during off-peak hours.
  - Reduce the number of items being backed up in a single job.

### 4. Restore Failures

- **Symptoms:**
  - Unable to restore files or data from backups.
  - Restore process halts or produces errors.

- **Resolution:**
  - Verify that the backup files are not corrupted.
  - Check for sufficient storage space on the target device.
  - Test restores regularly to ensure backup integrity.

### 5. Incomplete Backups

- **Symptoms:**
  - Backup jobs complete but miss some data (e.g., emails, files).
  - Backup logs show skipped or incomplete items.

- **Resolution:**
  - Verify that the Microsoft 365 account has permissions to access all data.
  - Check for throttling or API limits from Microsoft 365.
  - Increase the backup job timeout settings in Active Backup for Office 365.

### 6. Microsoft 365 API Issues

- **Symptoms:**
  - Backup jobs fail due to API errors.
  - Error messages related to Microsoft 365 API limits or throttling.

- **Resolution:**
  - Check Microsoft 365 service health for API-related issues.
  - Reduce the number of concurrent backup jobs to avoid hitting API limits.
  - Use Microsoft 365's **Retry-After** header to handle throttling.

### 7. Storage Pool or Volume Issues

- **Symptoms:**
  - Backup jobs fail due to storage errors.
  - Storage pool or volume shows as degraded or critical.

- **Resolution:**
  - Check the storage pool status in **Storage Manager**.
  - Replace failing drives and rebuild the storage pool.
  - Ensure proper RAID configuration for redundancy.

### 8. Permission Issues

- **Symptoms:**
  - Backup jobs fail due to permission errors.
  - Unable to access specific data (e.g., SharePoint sites, OneDrive folders).

- **Resolution:**
  - Verify that the Microsoft 365 account has proper permissions to access the data.
  - Check user roles and permissions in Microsoft 365 Admin Center.
  - Use a dedicated backup account with sufficient privileges.

### 9. Backup Verification Failures

- **Symptoms:**
  - Backup verification fails or produces errors.
  - Backup integrity cannot be confirmed.

- **Resolution:**
  - Verify that the backup files are not corrupted.
  - Test restores regularly to ensure backup integrity.
  - Use checksum or hash verification tools if available.

### 10. Teams Backup Issues

- **Symptoms:**
  - Teams data (e.g., channels, chats) is not backed up.
  - Backup jobs fail for Teams data.

- **Resolution:**
  - Verify that the Microsoft 365 account has permissions to access Teams data.
  - Check for API limitations or throttling related to Teams.
  - Update Active Backup for Office 365 to the latest version.

## General Troubleshooting Tips

- **Check Logs:** Review Active Backup for Office 365 logs (**Log Center**) for errors.
- **Test on a Single Account:** Always test backup and restore processes on a single account before scaling.
- **Update Software:** Ensure DSM and Active Backup for Office 365 are updated to the latest version.
- **Community Support:**

[go to Contents](#contents)

---

# Active Backup for Endpoint

**Active Backup for Endpoint** is a Synology solution designed to back up endpoints such as Windows PCs, Linux machines, and physical servers. While it is a powerful tool, users may encounter issues. Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Active Backup for Endpoint

### 1. Agent Installation Failures

- **Symptoms:**
  - Active Backup for Endpoint agent fails to install on endpoints.
  - Error messages during installation (e.g., "Installation failed").

- **Resolution:**
  - Ensure the endpoint meets the minimum system requirements.
  - Check for conflicting software (e.g., antivirus or other backup tools).
  - Verify network connectivity and firewall rules (default port **5510** for Active Backup for Endpoint).
  - Use the **Manual Installer** for troubleshooting.

### 2. Agent Communication Issues

- **Symptoms:**
  - Endpoints show as offline in the Active Backup for Endpoint dashboard.
  - Backup jobs fail due to communication errors.

- **Resolution:**
  - Verify network connectivity between the endpoint and Synology NAS.
  - Check firewall rules to allow outbound traffic on port **5510**.
  - Restart the Active Backup for Endpoint agent service on the endpoint:

    ```bash
    net stop ActiveBackupForEndpointAgent
    net start ActiveBackupForEndpointAgent
    ```

### 3. Backup Job Failures

- **Symptoms:**
  - Backup jobs fail to start or complete.
  - Error messages in the Active Backup for Endpoint logs.

- **Resolution:**
  - Verify that the endpoint is accessible over the network.
  - Check for sufficient storage space on the Synology NAS.
  - Review backup job settings (e.g., schedule, retention policies).
  - Update Synology DSM and Active Backup for Endpoint to the latest version.

### 4. Slow Backup Performance

- **Symptoms:**
  - Backup jobs take longer than expected.
  - High CPU or network usage during backups.

- **Resolution:**
  - Optimize network bandwidth (e.g., use a dedicated backup network).
  - Use faster storage (e.g., SSDs) on the Synology NAS.
  - Enable **deduplication** and **compression** to reduce backup size.
  - Schedule backups during off-peak hours.

### 5. Restore Failures

- **Symptoms:**
  - Unable to restore files or systems from backups.
  - Restore process halts or produces errors.

- **Resolution:**
  - Verify that the backup files are not corrupted.
  - Check for sufficient storage space on the target device.
  - Test restores regularly to ensure backup integrity.

### 6. Permission Issues

- **Symptoms:**
  - Backup jobs fail due to permission errors.
  - Unable to access files or folders on the endpoint.

- **Resolution:**
  - Verify that the Active Backup for Endpoint agent has proper permissions to access the endpoint.
  - Check user roles and permissions in **Control Panel > User & Group**.
  - Use a dedicated backup account with sufficient privileges.

### 7. Storage Pool or Volume Issues

- **Symptoms:**
  - Backup jobs fail due to storage errors.
  - Storage pool or volume shows as degraded or critical.

- **Resolution:**
  - Check the storage pool status in **Storage Manager**.
  - Replace failing drives and rebuild the storage pool.
  - Ensure proper RAID configuration for redundancy.

### 8. Backup Verification Failures

- **Symptoms:**
  - Backup verification fails or produces errors.
  - Backup integrity cannot be confirmed.

- **Resolution:**
  - Verify that the backup files are not corrupted.
  - Test restores regularly to ensure backup integrity.
  - Use checksum or hash verification tools if available.

### 9. Endpoint Performance Issues

- **Symptoms:**
  - Endpoint performance degrades during backups.
  - High CPU or disk usage on the endpoint.

- **Resolution:**
  - Schedule backups during off-peak hours.
  - Reduce the number of concurrent backup jobs.
  - Optimize backup settings (e.g., exclude unnecessary files).

### 10. Backup Retention Policy Issues

- **Symptoms:**
  - Backups are not retained according to the configured policies.
  - Backup storage fills up unexpectedly.

- **Resolution:**
  - Verify retention policy settings in Active Backup for Endpoint.
  - Manually clean up old backups if needed.
  - Adjust retention policies to better match storage capacity.

## General Troubleshooting Tips

- **Check Logs:** Review Active Backup for Endpoint logs (**Log Center**) for errors.
- **Test on a Single Endpoint:** Always test backup and restore processes on a single endpoint before scaling.
- **Update Software:** Ensure DSM and Active Backup for Endpoint are updated to the latest version.
- **Community Support:**

[go to Contents](#contents)

---

# Docker

**Docker** is a popular platform for developing, shipping, and running applications in containers. While it is highly efficient, users may encounter issues. 

**Docker Documentation:** https://docs.docker.com/

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Docker

### 1. Container Fails to Start

- **Symptoms:**
  - Container exits immediately after starting.
  - Error messages like `Cannot start service` or `Exited with code 1`.

- **Resolution:**
  - Check container logs for errors:

    ```bash
    docker logs <container_id>
    ```
  - Verify that the container image is not corrupted:

    ```bash
    docker pull <image_name>
    ```
  - Ensure all required environment variables and volumes are correctly configured.

### 2. Image Pull Failures

- **Symptoms:**
  - Unable to pull Docker images from a registry.
  - Error messages like `Error response from daemon` or `pull access denied`.

- **Resolution:**
  - Verify internet connectivity and DNS settings.
  - Check if the image name and tag are correct.
  - Log in to the Docker registry if required:

    ```bash
    docker login
    ```

### 3. Networking Issues

- **Symptoms:**
  - Containers cannot communicate with each other or the host.
  - Network-related errors in container logs.

- **Resolution:**
  - Verify Docker network configuration:

    ```bash
    docker network ls
    docker network inspect <network_name>
    ```
  - Ensure containers are connected to the correct network.
  - Check firewall rules and ensure Docker's default ports are open.

### 4. Volume Mounting Issues

- **Symptoms:**
  - Data is not persisted or accessible in mounted volumes.
  - Error messages like `Volume not found` or `Permission denied`.

- **Resolution:**
  - Verify the volume path exists on the host.
  - Check permissions for the host directory:

    ```bash
    chmod -R 777 /path/to/volume
    ```
  - Use absolute paths for volume mounts in `docker-compose.yml` or `docker run` commands.

### 5. High Resource Usage

- **Symptoms:**
  - Containers consume excessive CPU, memory, or disk space.
  - Host system becomes slow or unresponsive.

- **Resolution:**
  - Monitor resource usage:

    ```bash
    docker stats
    ```
  - Set resource limits for containers in `docker-compose.yml` or `docker run` commands:

    ```yaml
    resources:
      limits:
        cpus: "1.0"
        memory: "512m"
    ```

### 6. Docker Daemon Issues

- **Symptoms:**
  - Docker commands fail with `Cannot connect to the Docker daemon`.
  - Docker service is not running.

- **Resolution:**
  - Restart the Docker service:

    ```bash
    sudo systemctl restart docker
    ```
  - Check Docker daemon logs for errors:

    ```bash
    journalctl -u docker.service
    ```

### 7. Port Conflicts

- **Symptoms:**
  - Containers fail to start due to port conflicts.
  - Error messages like `Port is already allocated`.

- **Resolution:**
  - Check for running containers using the same port:

    ```bash
    docker ps
    ```
  - Change the host port in `docker-compose.yml` or `docker run` commands:

    ```yaml
    ports:
      - "8081:80"
    ```

### 8. Docker Compose Issues

- **Symptoms:**
  - `docker-compose up` fails or produces errors.
  - Services do not start as expected.

- **Resolution:**
  - Verify the `docker-compose.yml` file for syntax errors.
  - Check service dependencies and ensure they are correctly defined.
  - Use `docker-compose logs` to troubleshoot service-specific issues.

### 9. Image Build Failures

- **Symptoms:**
  - `docker build` fails with errors.
  - Error messages like `Build failed` or `Step failed`.

- **Resolution:**
  - Check the Dockerfile for syntax errors or missing dependencies.
  - Ensure all required files and context are included in the build directory.
  - Use `--no-cache` to rebuild the image without cache:

    ```bash
    docker build --no-cache -t <image_name> .
    ```

### 10. Container Logs Not Showing

- **Symptoms:**
  - No logs are displayed for a running container.
  - Logs are incomplete or missing.

- **Resolution:**
  - Verify the logging driver in Docker configuration:

    ```bash
    docker inspect <container_id> --format='{{.HostConfig.LogConfig.Type}}'
    ```
  - Use the `json-file` logging driver for detailed logs:

    ```yaml
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    ```

## General Troubleshooting Tips

- **Check Logs:** Use `docker logs <container_id>` to view container logs.
- **Update Docker:** Ensure Docker and Docker Compose are updated to the latest version.
- **Test Locally:** Test configurations on a local environment before deploying to production.
- **Community Support:** https://forums.docker.com/

[go to Contents](#contents)

---

# Kubernetes

**Kubernetes** is a powerful container orchestration platform, but it can be complex, and users may encounter issues. 

[YouTube video: NetworkChuck](https://www.youtube.com/watch?v=7bA0gTroJjw)

[YouTube video: Crash Course](https://www.youtube.com/watch?v=s_o8dwzRlu4)

**Kubernetes Documentation:** https://kubernetes.io/docs/home/supported-doc-versions/

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Kubernetes

### 1. Pods Stuck in Pending State

- **Symptoms:**
  - Pods remain in the `Pending` state and do not start.
  - Error messages like `Insufficient CPU` or `Insufficient memory`.

- **Resolution:**
  - Check resource requests and limits in the pod definition:

    ```yaml
    resources:
      requests:
        cpu: "500m"
        memory: "512Mi"
      limits:
        cpu: "1"
        memory: "1Gi"
    ```
  - Verify node capacity and availability:

    ```bash
    kubectl describe node <node_name>
    ```
  - Ensure there are no taints or tolerations blocking scheduling.

### 2. Pods CrashLoopBackOff

- **Symptoms:**
  - Pods repeatedly crash and restart.
  - Error messages in pod logs.

- **Resolution:**
  - Check pod logs for errors:

    ```bash
    kubectl logs <pod_name>
    ```
  - Verify container images and configurations.
  - Ensure all required environment variables and secrets are correctly configured.

### 3. Service Not Accessible

- **Symptoms:**
  - Services are not reachable from within or outside the cluster.
  - Error messages like `Connection refused` or `Timeout`.

- **Resolution:**
  - Verify service configuration:

    ```bash
    kubectl get svc <service_name>
    ```
  - Check pod selectors and labels:

    ```yaml
    selector:
      app: my-app
    ```
  - Ensure network policies or firewall rules are not blocking traffic.

### 4. Persistent Volume (PV) Issues

- **Symptoms:**
  - Pods fail to mount Persistent Volumes (PVs).
  - Error messages like `FailedMount` or `Volume not found`.

- **Resolution:**
  - Verify PV and Persistent Volume Claim (PVC) configurations:

    ```bash
    kubectl get pv
    kubectl get pvc
    ```
  - Check storage class and provisioner settings.
  - Ensure the PV is bound to the correct PVC.

### 5. Node Not Ready

- **Symptoms:**
  - Nodes show as `NotReady` in the cluster.
  - Error messages in node logs.

- **Resolution:**
  - Check node status and logs:

    ```bash
    kubectl describe node <node_name>
    ```
  - Verify kubelet and container runtime (e.g., Docker, containerd) are running:

    ```bash
    systemctl status kubelet
    systemctl status docker
    ```
  - Ensure the node has sufficient resources (CPU, memory, disk).

### 6. Network Policy Issues

- **Symptoms:**
  - Pods cannot communicate with each other.
  - Network policies are not enforced as expected.

- **Resolution:**
  - Verify network policy configurations:

    ```bash
    kubectl get networkpolicies
    ```
  - Check if the network plugin (e.g., Calico, Flannel) supports network policies.
  - Ensure pods are correctly labeled and selected by the network policy.

### 7. Image Pull Failures

- **Symptoms:**
  - Pods fail to start due to image pull errors.
  - Error messages like `ImagePullBackOff` or `ErrImagePull`.

- **Resolution:**
  - Verify the image name and tag are correct.
  - Check if the image registry is accessible and credentials are correct:

    ```bash
    kubectl describe pod <pod_name>
    ```
  - Use a private registry secret if required:

    ```yaml
    imagePullSecrets:
      - name: regcred
    ```

### 8. ConfigMap and Secret Issues

- **Symptoms:**
  - Pods fail to start or misbehave due to missing or incorrect ConfigMaps or Secrets.
  - Error messages like `ConfigMap not found` or `Secret not found`.

- **Resolution:**
  - Verify ConfigMap and Secret configurations:

    ```bash
    kubectl get configmap <configmap_name>
    kubectl get secret <secret_name>
    ```
  - Ensure pods reference the correct ConfigMap or Secret:

    ```yaml
    env:
      - name: MY_ENV_VAR
        valueFrom:
          configMapKeyRef:
            name: my-config
            key: my-key
    ```

### 9. Ingress Issues

- **Symptoms:**
  - Ingress resources are not routing traffic correctly.
  - Error messages like `404 Not Found` or `503 Service Unavailable`.

- **Resolution:**
  - Verify Ingress configuration:

    ```bash
    kubectl get ingress <ingress_name>
    ```
  - Check if the Ingress controller (e.g., Nginx, Traefik) is running:

    ```bash
    kubectl get pods -n <ingress_controller_namespace>
    ```
  - Ensure backend services are correctly configured and accessible.

### 10. RBAC Issues

- **Symptoms:**
  - Users or service accounts cannot perform actions.
  - Error messages like `Forbidden` or `Unauthorized`.

- **Resolution:**
  - Verify Role-Based Access Control (RBAC) configurations:

    ```bash
    kubectl get roles,rolebindings,clusterroles,clusterrolebindings
    ```
  - Ensure service accounts have the necessary permissions:

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: my-role
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["get", "list", "watch"]
    ```

## General Troubleshooting Tips

- **Check Logs:** Use `kubectl logs <pod_name>` to view pod logs.
- **Describe Resources:** Use `kubectl describe <resource_type> <resource_name>` for detailed information.
- **Update Kubernetes:** Ensure Kubernetes and its components are updated to the latest version.
- **Community Support:** https://discuss.kubernetes.io/

[go to Contents](#contents)

---

# S3

AWS Object Storage.

**Feature**

 - Stores data as objects within buckets
 - Key is a unique identifier for an object within a bucket
 - Storage capacity is virtually unlimited
 - S3 Standard, S3 Standard-IA, and Glacier storage classes, your objects are automatically stored across multiple devices spanning a minimum of three Availability Zones


**S3 Storage Class**

Frequently Access :

 - **Standard** - For frequently accessed data
 - **Express One Zone** - High-performance, single AZ storage class designed to deliver consistent single-digit millisecond data access

Infrequently Access :

 - **Standard IA** -  Object data redundantly across multiple geographically separated AZs
 - **OneZone IA** - Data store in only one AZ (best for data that can be easily reproduce)

Archiving - **Glacier** : 

 - **S3 Glacier Instant Retrieval** - Rarely accessed and must be retrieved in milliseconds
 - **S3 Glacier Flexible Retrieval** - Data that is accessed once or twice per year
 - **S3 Glacier Deep Archive** - Data that is accessed rarely in a year.

Retrieval Options:

 - **Expedited** - Quickly access your data when urgent requests are required (1–5 minutes)
 - **Standard** - Access any of archived objects within several hours (3–5 hours)
 - **Bulk** - Lowest-cost retrieval option (5–12 hours)

[go to Contents](#contents)

---

# Zabbix

**Zabbix** is a powerful open-source monitoring solution used to track the performance and availability of IT infrastructure. While it is highly effective, users may encounter issues. 

**Zabbix Documentation:** https://www.zabbix.com/manuals

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Zabbix

### 1. Zabbix Server Not Starting

- **Symptoms:**
  - Zabbix server service fails to start.
  - Error messages in logs (`/var/log/zabbix/zabbix_server.log`).

- **Resolution:**
  - Check for configuration errors in `/etc/zabbix/zabbix_server.conf`.
  - Verify database connectivity and credentials:

    ```bash
    mysql -u zabbix -p -h <db_host> -D zabbix
    ```
  - Ensure the database schema is up-to-date:

    ```bash
    zabbix_server -c /etc/zabbix/zabbix_server.conf -R dbversion
    ```

### 2. Zabbix Agent Not Reporting Data

- **Symptoms:**
  - Hosts show as "No data" in the Zabbix frontend.
  - Zabbix agent logs show errors (`/var/log/zabbix/zabbix_agentd.log`).

- **Resolution:**
  - Verify the Zabbix agent is running:

    ```bash
    systemctl status zabbix-agent
    ```
  - Check firewall rules to allow communication between the agent and server (default port **10050**).
  - Ensure the agent configuration (`/etc/zabbix/zabbix_agentd.conf`) points to the correct Zabbix server.

### 3. High CPU or Memory Usage

- **Symptoms:**
  - Zabbix server or database consumes excessive resources.
  - Slow performance or system instability.

- **Resolution:**
  - Optimize Zabbix server configuration (e.g., reduce `StartPollers` or `StartDiscoverers` in `zabbix_server.conf`).
  - Clean up old data from the database:

    ```sql
    DELETE FROM history WHERE clock < UNIX_TIMESTAMP(DATE_SUB(NOW(), INTERVAL 30 DAY));
    ```
  - Use partitioning for large tables (e.g., `history`, `trends`).

### 4. Database Connection Issues

- **Symptoms:**
  - Zabbix server cannot connect to the database.
  - Error messages like `Cannot connect to database` in logs.

- **Resolution:**
  - Verify database credentials in `zabbix_server.conf`.
  - Check if the database service is running:

    ```bash
    systemctl status mysql
    ```
  - Ensure the database user has sufficient privileges:

    ```sql
    GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost' IDENTIFIED BY 'password';
    ```

### 5. Frontend Access Issues

- **Symptoms:**
  - Unable to access the Zabbix web interface.
  - Error messages like `404 Not Found` or `500 Internal Server Error`.

- **Resolution:**
  - Verify the web server (e.g., Apache, Nginx) is running:

    ```bash
    systemctl status apache2
    ```
  - Check PHP configuration and ensure required extensions are enabled:

    ```bash
    php -m | grep -E 'mbstring|bcmath|gd|xml|ldap'
    ```
  - Ensure the Zabbix frontend files are correctly placed in the web server directory.

### 6. Trigger Not Firing

- **Symptoms:**
  - Triggers do not fire even when conditions are met.
  - No alerts are generated.

- **Resolution:**
  - Verify trigger expressions and dependencies in the Zabbix frontend.
  - Check if the item is collecting data correctly:

    ```bash
    zabbix_get -s <host> -k <item_key>
    ```
  - Ensure the trigger is enabled and not in maintenance mode.

### 7. Email Notifications Not Working

- **Symptoms:**
  - Email alerts are not sent.
  - Error messages in Zabbix server logs.

- **Resolution:**
  - Verify email settings in the Zabbix frontend (**Administration > Media Types**).
  - Check if the mail server is accessible and credentials are correct.
  - Test email delivery using a script or command-line tool:

    ```bash
    echo "Test email" | mail -s "Test" user@example.com
    ```

### 8. Discovery Not Working

- **Symptoms:**
  - Network discovery does not find devices.
  - No hosts or services are discovered.

- **Resolution:**
  - Verify discovery rules in the Zabbix frontend (**Configuration > Discovery**).
  - Check if the Zabbix server can reach the target network:

    ```bash
    ping <target_ip>
    ```
  - Ensure the discovery interval is set correctly.

### 9. Proxy Issues

- **Symptoms:**
  - Zabbix proxy does not forward data to the server.
  - Error messages in proxy logs (`/var/log/zabbix/zabbix_proxy.log`).

- **Resolution:**
  - Verify proxy configuration (`/etc/zabbix/zabbix_proxy.conf`).
  - Check network connectivity between the proxy and server.
  - Ensure the proxy is registered in the Zabbix frontend.

### 10. Performance Issues with Large Environments

- **Symptoms:**
  - Slow performance in the Zabbix frontend or server.
  - High database load or query timeouts.

- **Resolution:**
  - Optimize database queries and indexes:

    ```sql
    ANALYZE TABLE history;
    OPTIMIZE TABLE history;
    ```
  - Use housekeeping to clean up old data:

    ```bash
    zabbix_server -c /etc/zabbix/zabbix_server.conf -R housekeeper_execute
    ```
  - Consider using distributed monitoring with proxies.

## General Troubleshooting Tips

- **Check Logs:** Review Zabbix server, agent, and proxy logs (`/var/log/zabbix/`) for errors.
- **Test Connectivity:** Use tools like `ping`, `telnet`, and `zabbix_get` to verify connectivity.
- **Update Zabbix:** Ensure Zabbix and its components are updated to the latest version.
- **Community Support:** https://www.zabbix.com/forum/

[go to Contents](#contents)

---

# Chocolatey

**Chocolatey** is a package manager for Windows that simplifies software installation, updates, and management. 

```bash
choco new testpackage
```

**Script**
 - chocolateyBeforeModify.ps1 
 - chocolateyInstall.ps1	
 - chocolateyUninstall.ps1

**Chocolatey Documentation:** https://docs.chocolatey.org/en-us/

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Chocolatey

### 1. Package Installation Failures

- **Symptoms:**
  - Package installation fails with errors.
  - Error messages like `Installation failed` or `Package not found`.

- **Resolution:**
  - Verify the package name and version:

    ```bash
    choco list <package_name>
    ```
  - Check for sufficient permissions (run Command Prompt or PowerShell as Administrator).
  - Use the `--force` flag to reinstall the package:

    ```bash
    choco install <package_name> --force
    ```

### 2. Chocolatey Not Recognized

- **Symptoms:**
  - `choco` command is not recognized in Command Prompt or PowerShell.
  - Error messages like `'choco' is not recognized as an internal or external command`.

- **Resolution:**
  - Ensure Chocolatey is installed correctly:

    ```bash
    @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
    ```
  - Verify the Chocolatey installation path is in the system `PATH` environment variable.

### 3. Proxy Issues

- **Symptoms:**
  - Unable to download packages due to proxy restrictions.
  - Error messages like `Unable to connect to the remote server`.

- **Resolution:**
  - Configure proxy settings in Chocolatey:

    ```bash
    choco config set proxy <proxy_url>
    choco config set proxyUser <username>
    choco config set proxyPassword <password>
    ```
  - Alternatively, set environment variables for proxy:

    ```bash
    set HTTP_PROXY=http://<proxy_url>:<port>
    set HTTPS_PROXY=https://<proxy_url>:<port>
    ```

### 4. Package Update Failures

- **Symptoms:**
  - Packages fail to update.
  - Error messages like `Update failed` or `Package not installed`.

- **Resolution:**
  - Verify the package is installed:

    ```bash
    choco list --local-only
    ```
  - Use the `--force` flag to force an update:

    ```bash
    choco upgrade <package_name> --force
    ```

### 5. Dependency Issues

- **Symptoms:**
  - Package installation fails due to missing dependencies.
  - Error messages like `Dependency not found`.

- **Resolution:**
  - Install missing dependencies manually:

    ```bash
    choco install <dependency_name>
    ```
  - Use the `--ignore-dependencies` flag to bypass dependency checks (not recommended):

    ```bash
    choco install <package_name> --ignore-dependencies
    ```

### 6. Chocolatey Source Issues

- **Symptoms:**
  - Unable to find or install packages from a specific source.
  - Error messages like `Source not found` or `Unable to load the service index`.

- **Resolution:**
  - Verify the source is available:

    ```bash
    choco source list
    ```
  - Add or enable the source:

    ```bash
    choco source add --name=<source_name> --source=<source_url>
    choco source enable --name=<source_name>
    ```

### 7. Permissions Issues

- **Symptoms:**
  - Package installation or update fails due to insufficient permissions.
  - Error messages like `Access denied` or `Permission denied`.

- **Resolution:**
  - Run Command Prompt or PowerShell as Administrator.
  - Check and modify file/folder permissions if necessary.

### 8. Package Corruption

- **Symptoms:**
  - Installed packages are corrupted or do not function correctly.
  - Error messages like `Corrupted package` or `Invalid checksum`.

- **Resolution:**
  - Reinstall the package:

    ```bash
    choco uninstall <package_name>
    choco install <package_name>
    ```
  - Use the `--force` flag to overwrite existing files:

    ```bash
    choco install <package_name> --force
    ```

### 9. Chocolatey Configuration Issues

- **Symptoms:**
  - Chocolatey behaves unexpectedly or fails to execute commands.
  - Error messages like `Invalid configuration`.

- **Resolution:**
  - Check and reset Chocolatey configuration:

    ```bash
    choco config list
    choco config unset <config_key>
    ```
  - Restore default settings if necessary.

### 10. Performance Issues

- **Symptoms:**
  - Slow package installation or update.
  - High CPU or disk usage during operations.

- **Resolution:**
  - Optimize Chocolatey settings (e.g., reduce concurrent downloads):

    ```bash
    choco config set commandExecutionTimeoutSeconds 2700
    ```
  - Use a faster internet connection or local repository.

## General Troubleshooting Tips

- **Check Logs:** Review Chocolatey logs (`%ChocolateyInstall%\logs\`) for errors.
- **Update Chocolatey:** Ensure Chocolatey is updated to the latest version:

  ```bash
  choco upgrade chocolatey
  ```
- **Community Support:** https://community.chocolatey.org/

[go to Contents](#contents)

---

# pfSense

**pfSense** is a powerful open-source firewall and router platform based on FreeBSD.

**pfSense Documentation:** https://docs.netgate.com/pfsense/en/latest/

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with pfSense

### 1. No Internet Connectivity

- **Symptoms:**
  - Devices behind the pfSense firewall cannot access the internet.
  - WAN interface shows no IP address or connectivity.

- **Resolution:**
  - Verify WAN interface configuration (**Interfaces > WAN**).
  - Check for correct IP address, gateway, and DNS settings.
  - Ensure the ISP modem/router is in bridge mode if necessary.
  - Test connectivity from the pfSense shell:

    ```bash
    ping 8.8.8.8
    ```

### 2. DNS Resolution Issues

- **Symptoms:**
  - Devices cannot resolve domain names.
  - DNS queries fail or timeout.

- **Resolution:**
  - Verify DNS server settings (**System > General Setup**).
  - Use public DNS servers like Google (8.8.8.8, 8.8.4.4) or Cloudflare (1.1.1.1).
  - Check DNS resolver or forwarder settings (**Services > DNS Resolver/Forwarder**).

### 3. High CPU or Memory Usage

- **Symptoms:**
  - pfSense becomes slow or unresponsive.
  - High CPU or memory usage reported in the dashboard.

- **Resolution:**
  - Identify resource-intensive processes:

    ```bash
    top
    ```

  - Optimize firewall rules and NAT configurations.
  - Disable unnecessary packages or services.

### 4. VPN Connection Issues

- **Symptoms:**
  - VPN clients cannot connect to pfSense.
  - VPN tunnels fail to establish.

- **Resolution:**
  - Verify VPN configuration (**VPN > OpenVPN/IPsec**).
  - Check firewall rules to allow VPN traffic.
  - Ensure correct certificates and keys are configured.
  - Test connectivity and logs for errors.

### 5. Firewall Rules Not Working

- **Symptoms:**
  - Traffic is blocked or allowed unexpectedly.
  - Firewall rules do not behave as configured.

- **Resolution:**
  - Verify rule order and priorities (**Firewall > Rules**).
  - Check for conflicting rules or misconfigured NAT settings.
  - Use packet capture to diagnose traffic flow:

    ```bash
    tcpdump -i <interface>
    ```

### 6. Interface Configuration Issues

- **Symptoms:**
  - Network interfaces are not functioning.
  - Interfaces show as down or disconnected.

- **Resolution:**
  - Verify physical connections and link status.
  - Check interface settings (**Interfaces > [Interface Name]**).
  - Reassign or reconfigure interfaces if necessary.

### 7. Captive Portal Issues

- **Symptoms:**
  - Users cannot authenticate or access the captive portal.
  - Captive portal pages do not load.

- **Resolution:**
  - Verify captive portal settings (**Services > Captive Portal**).
  - Check firewall rules to allow captive portal traffic.
  - Ensure correct authentication settings (e.g., RADIUS, local users).

### 8. Package Installation Failures

- **Symptoms:**
  - pfSense packages fail to install or update.
  - Error messages like `Package not found` or `Installation failed`.

- **Resolution:**
  - Verify internet connectivity and DNS resolution.
  - Check package repository settings (**System > Package Manager > Settings**).
  - Manually download and install packages if necessary.

### 9. Configuration Backup/Restore Issues

- **Symptoms:**
  - Configuration backup fails or cannot be restored.
  - Error messages during backup/restore process.

- **Resolution:**
  - Verify sufficient storage space for backups.
  - Check file permissions and paths.
  - Use the XML configuration file for manual backup/restore.

### 10. Logging and Monitoring Issues

- **Symptoms:**
  - Logs are not populated or show errors.
  - Monitoring graphs are missing or incorrect.

- **Resolution:**
  - Verify logging settings (**Status > System Logs**).
  - Check disk space and log rotation settings.
  - Enable and configure monitoring settings (**Status > Monitoring**).

## General Troubleshooting Tips

- **Check Logs:** Review system logs (**Status > System Logs**) for errors.
- **Test Connectivity:** Use tools like `ping`, `traceroute`, and `tcpdump` to diagnose network issues.
- **Update pfSense:** Ensure pfSense is updated to the latest version:

  ```bash
  pfSense-upgrade
  ```
- **Community Support:** https://forum.netgate.com/

[go to Contents](#contents)

---

# FreePBX

**FreePBX** is a popular open-source GUI (Graphical User Interface) for managing Asterisk, a powerful telephony platform. 

**PBX** = Private Branch eXchange, a hardware system that handles routing and switching of calls between a business location and the telephone network.

**Asterisk** = Software implementation of a private branch exchange (PBX) used to establish and control telephone calls. 

**FreePBX Documentation:** https://sangomakb.atlassian.net/wiki/spaces/FP/pages/9699329/FreePBX+OpenSource+Project

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with FreePBX

### 1. No Audio (One-Way or No-Way Audio)

- **Symptoms:**
  - Callers cannot hear each other.
  - One-way audio (only one party can hear).

- **Resolution:**
  - Check NAT settings (**Settings > Asterisk SIP Settings**):
    - Set **NAT** to "Yes" if behind a firewall.
    - Use `externip` and `localnet` settings if necessary.
  - Verify firewall rules to allow RTP traffic (UDP ports **10000-20000**).
  - Use `sip show channels` and `rtp debug` in the Asterisk CLI to diagnose.

### 2. Calls Fail to Connect

- **Symptoms:**
  - Calls do not go through.
  - Error messages like `No Response` or `Call Failed`.

- **Resolution:**
  - Verify trunk configuration (**Connectivity > Trunks**):
    - Check SIP credentials and registration status.
    - Use `sip show registry` in the Asterisk CLI to confirm registration.
  - Ensure firewall rules allow SIP traffic (UDP port **5060**).

### 3. Voicemail Issues

- **Symptoms:**
  - Voicemails are not recorded or delivered.
  - Users cannot access voicemail.

- **Resolution:**
  - Verify voicemail settings (**Applications > Voicemail Admin**):
    - Check mailbox configurations and email settings.
  - Ensure the `asterisk` user has permissions to send emails.
  - Test email delivery using the **Email Test** tool (**Admin > Email Templates**).

### 4. Extension Registration Failures

- **Symptoms:**
  - Extensions fail to register.
  - Error messages like `Registration Failed` or `No Response`.

- **Resolution:**
  - Verify extension settings (**Applications > Extensions**):
    - Check SIP credentials and NAT settings.
  - Ensure the extension device is configured correctly (e.g., SIP username, password, and server).
  - Use `sip show peers` in the Asterisk CLI to check registration status.

### 5. High CPU or Memory Usage

- **Symptoms:**
  - FreePBX becomes slow or unresponsive.
  - High CPU or memory usage reported in the system.

- **Resolution:**
  - Identify resource-intensive processes:

    ```bash
    top
    ```

  - Optimize Asterisk settings (e.g., reduce `maxload` in `asterisk.conf`).
  - Disable unnecessary modules or features.

### 6. Call Quality Issues (Jitter, Latency, Packet Loss)

- **Symptoms:**
  - Poor call quality (e.g., choppy audio, delays).

- **Resolution:**
  - Check network performance using tools like `ping` and `iperf`.
  - Prioritize VoIP traffic using QoS (Quality of Service) settings on the router.
  - Verify RTP packet flow using `rtp debug` in the Asterisk CLI.

### 7. Outbound Call Failures

- **Symptoms:**
  - Outbound calls fail to connect.
  - Error messages like `No Route to Destination` or `Call Rejected`.

- **Resolution:**
  - Verify outbound route configuration (**Connectivity > Outbound Routes**):
    - Check dial patterns and trunk selections.
  - Ensure the trunk is registered and operational.
  - Use `sip show channels` in the Asterisk CLI to diagnose.

### 8. Inbound Call Failures

- **Symptoms:**
  - Inbound calls fail to connect.
  - Error messages like `No Response` or `Call Rejected`.

- **Resolution:**
  - Verify inbound route configuration (**Connectivity > Inbound Routes**):
    - Check DID (Direct Inward Dialing) and destination settings.
  - Ensure the trunk is registered and operational.
  - Use `sip show channels` in the Asterisk CLI to diagnose.

### 9. Database Issues

- **Symptoms:**
  - FreePBX GUI shows errors or fails to load.
  - Error messages like `Database Connection Failed`.

- **Resolution:**
  - Verify database settings (`/etc/amportal.conf`):
    - Check MySQL credentials and connectivity.
  - Restart the database service:

    ```bash
    systemctl restart mariadb
    ```

  - Repair the database if necessary:

    ```bash
    amportal a ma repair
    ```

### 10. Module or Feature Issues

- **Symptoms:**
  - Modules fail to install or update.
  - Features do not work as expected.

- **Resolution:**
  - Verify internet connectivity and DNS resolution.
  - Check module settings (**Admin > Module Admin**):
    - Update or reinstall problematic modules.
  - Use the **Module Troubleshooter** to diagnose issues.

## General Troubleshooting Tips

- **Check Logs:** Review Asterisk logs (`/var/log/asterisk/full`) and FreePBX logs (**Reports > Asterisk Logs**) for errors.
- **Test Connectivity:** Use tools like `ping`, `traceroute`, and `sip show peers` to diagnose network and SIP issues.
- **Update FreePBX:** Ensure FreePBX and Asterisk are updated to the latest version:

  ```bash
  fwconsole upgrade
  ```
- **Community Support:** https://community.freepbx.org/

[go to Contents](#contents)

---

# CephFS

**CephFS** is a distributed file system built on top of the Ceph storage system. 

**CephFS Documentation:** https://docs.ceph.com/en/reef/cephfs/ , https://docs.redhat.com/en/documentation/red_hat_ceph_storage/2/html/ceph_file_system_guide_technology_preview/what_is_the_ceph_file_system_cephfs

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with CephFS

### 1. Mount Failures

- **Symptoms:**
  - Unable to mount CephFS on a client.
  - Error messages like `mount error`, `no mds server is up`, or `permission denied`.

- **Resolution:**
  - Verify the CephFS file system is created and active:

    ```bash
    ceph fs status
    ```

  - Ensure the MDS (Metadata Server) is running:

    ```bash
    ceph mds stat
    ```

  - Check client keyring permissions and mount command syntax:

    ```bash
    mount -t ceph <monitor_ip>:6789:/ /mnt/cephfs -o name=client.admin,secret=<key>
    ```

### 2. Metadata Server (MDS) Issues

- **Symptoms:**
  - MDS service crashes or fails to start.
  - Error messages like `mds rank unavailable` or `mds stuck`.

- **Resolution:**
  - Check MDS logs for errors (`/var/log/ceph/ceph-mds.*.log`).
  - Verify MDS configuration and resource allocation:

    ```bash
    ceph mds stat
    ceph mds fail <mds_name>
    ```

  - Restart the MDS service:

    ```bash
    systemctl restart ceph-mds@<mds_name>
    ```

### 3. Performance Issues

- **Symptoms:**
  - Slow file operations (e.g., read/write, metadata access).
  - High latency or low throughput.

- **Resolution:**
  - Check cluster health and performance metrics:

    ```bash
    ceph status
    ceph osd perf
    ```

  - Optimize MDS cache settings (`mds_cache_memory_limit`, `mds_cache_size`).
  - Use multiple active MDS instances for better metadata performance:

    ```bash
    ceph fs set <fs_name> max_mds <num_mds>
    ```

### 4. Quota Issues

- **Symptoms:**
  - Quota limits are not enforced.
  - Error messages like `quota exceeded` or `no space left on device`.

- **Resolution:**
  - Verify quota settings on directories:

    ```bash
    ceph fs quota get <path>
    ```

  - Set or update quotas as needed:

    ```bash
    ceph fs quota set <path> --max_bytes <size> --max_files <count>
    ```

  - Ensure the `ceph-quota` xattr is enabled on the directory.

### 5. Permission Issues

- **Symptoms:**
  - Users cannot access files or directories.
  - Error messages like `permission denied` or `access denied`.

- **Resolution:**
  - Verify file and directory permissions:

    ```bash
    ls -l /mnt/cephfs
    ```

  - Check client keyring permissions and capabilities:

    ```bash
    ceph auth get client.<client_name>
    ```

  - Use `chmod` or `chown` to adjust permissions.

### 6. Snapshot Issues

- **Symptoms:**
  - Snapshots fail to create or restore.
  - Error messages like `snapshot creation failed` or `snapshot not found`.

- **Resolution:**
  - Verify snapshot support is enabled on the file system:

    ```bash
    ceph fs set <fs_name> allow_new_snaps true
    ```

  - Check for sufficient space in the file system.
  - Use `ceph fs snapshot` commands to manage snapshots.

### 7. Data Corruption or Integrity Issues

- **Symptoms:**
  - Files are corrupted or missing.
  - Error messages like `checksum error` or `data integrity error`.

- **Resolution:**
  - Run a scrub to check for data inconsistencies:

    ```bash
    ceph pg scrub <pg_id>
    ```

  - Verify cluster health and repair any damaged objects:

    ```bash
    ceph health detail
    ceph pg repair <pg_id>
    ```

### 8. Client Disconnections

- **Symptoms:**
  - Clients are disconnected from CephFS.
  - Error messages like `client session timed out` or `connection reset`.

- **Resolution:**
  - Check network connectivity between clients and Ceph cluster.
  - Verify client and server logs for errors (`/var/log/ceph/`).
  - Adjust client mount options for better stability:

    ```bash
    mount -t ceph <monitor_ip>:6789:/ /mnt/cephfs -o name=client.admin,secret=<key>,noatime
    ```

### 9. Capacity Issues

- **Symptoms:**
  - File system runs out of space.
  - Error messages like `no space left on device` or `quota exceeded`.

- **Resolution:**
  - Check cluster usage and add more OSDs (Object Storage Daemons) if needed:

    ```bash
    ceph osd df
    ```

  - Clean up unused files or snapshots.
  - Expand the file system by adding more storage.

### 10. Upgrade Issues

- **Symptoms:**
  - Problems after upgrading Ceph or CephFS.
  - Error messages like `incompatible version` or `feature not supported`.

- **Resolution:**
  - Follow the official Ceph upgrade guide for your version.
  - Check cluster health and compatibility:

    ```bash
    ceph status
    ceph versions
    ```

  - Roll back to the previous version if necessary.

## General Troubleshooting Tips

- **Check Logs:** Review Ceph logs (`/var/log/ceph/`) for errors.
- **Monitor Cluster Health:** Use `ceph status` and `ceph dashboard` to monitor cluster health and performance.
- **Update Ceph:** Ensure Ceph and CephFS are updated to the latest version.

[go to Contents](#contents)

---

# LAPS

**LAPS (Local Administrator Password Solution)** is a Microsoft solution for managing local administrator passwords on domain-joined computers. 

**Enable LAPS**
Azure AD --> Device --> Device Settings

**Create Policy**
- Azure AD

  Endpoint Security --> Account Protection --> + Create Policy

- AD

  GPO --> Admin Templates Policy --> Systems --> LAPS

Checking if the policy applied on the client side

1. RSOP from command prompt
2. `HKLM>SOFTWARE>Microsoft>Windows>CurrentVersion>Policies>LAPS`
3. Intune manage `HKLM>SOFTWARE>Microsoft>Policies>LAPS`
4. Event Viewer `Application and Services Logs>Microsoft>Windows>LAPS`
   - Event 10020 : When password get updated
   - Event 10021 : AD Settings
   - Event 10022 : Intune Settings

Monitor who access the password:

Azure AD --> Audit Logs --> Add `Activity` filter `recover`



**LAPS Documentation:** https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with LAPS

### 1. Password Not Updating

- **Symptoms:**
  - The local administrator password is not updated as expected.
  - The `ms-Mcs-AdmPwd` attribute in Active Directory is not populated.

- **Resolution:**
  - Verify the LAPS Group Policy Object (GPO) is applied to the target computers:
    - Check **Computer Configuration > Administrative Templates > LAPS**.
  - Ensure the LAPS client extension (`AdmPwd.dll`) is installed on the target computers.
  - Run `gpupdate /force` on the target computer to refresh Group Policy.

### 2. Password Not Retrievable

- **Symptoms:**
  - Users cannot retrieve the local administrator password.
  - Error messages like `Access Denied` or `Password Not Found`.

- **Resolution:**
  - Verify the user has the necessary permissions to read the `ms-Mcs-AdmPwd` attribute:
    - Use `Set-AdmPwdReadPasswordPermission` to grant read permissions.
  - Ensure the LAPS GPO is configured to allow password retrieval:
    - Check **Computer Configuration > Administrative Templates > LAPS > Configure Password Backup Directory**.

### 3. LAPS Client Not Installed

- **Symptoms:**
  - The LAPS client extension (`AdmPwd.dll`) is not installed on the target computer.
  - The `ms-Mcs-AdmPwd` attribute is not populated.

- **Resolution:**
  - Install the LAPS client extension on the target computer:
    - Download and install the LAPS client from the Microsoft Download Center.
  - Verify the installation by checking for `AdmPwd.dll` in the `C:\Windows\System32` directory.

### 4. GPO Configuration Issues

- **Symptoms:**
  - LAPS settings are not applied to the target computers.
  - The `ms-Mcs-AdmPwd` attribute is not populated.

- **Resolution:**
  - Verify the LAPS GPO is linked to the correct Organizational Unit (OU) in Active Directory.
  - Check GPO inheritance and enforcement settings.
  - Use `gpresult /r` or `rsop.msc` to verify GPO application on the target computer.

### 5. Active Directory Schema Issues

- **Symptoms:**
  - The `ms-Mcs-AdmPwd` attribute is missing in Active Directory.
  - Error messages like `Schema Extension Not Found`.

- **Resolution:**
  - Extend the Active Directory schema to include the LAPS attributes:
    - Run `Import-AdmPwdSchema.ps1` on a domain controller.
  - Verify the schema extension by checking for the `ms-Mcs-AdmPwd` attribute in Active Directory.

### 6. Password Complexity Issues

- **Symptoms:**
  - The local administrator password does not meet complexity requirements.
  - Error messages like `Password Does Not Meet Complexity Requirements`.

- **Resolution:**
  - Verify the password complexity settings in the LAPS GPO:
    - Check **Computer Configuration > Administrative Templates > LAPS > Configure Password Complexity**.
  - Ensure the password meets the domain's password policy requirements.

### 7. Password Expiry Issues

- **Symptoms:**
  - The local administrator password does not expire as expected.
  - The `ms-Mcs-AdmPwdExpirationTime` attribute is not updated.

- **Resolution:**
  - Verify the password expiration settings in the LAPS GPO:
    - Check **Computer Configuration > Administrative Templates > LAPS > Configure Password Expiration**.
  - Ensure the target computer's clock is synchronized with the domain controller.

### 8. LAPS UI Issues

- **Symptoms:**
  - The LAPS UI (`LAPS UI.msc`) does not display passwords or shows errors.
  - Error messages like `UI Not Found` or `Access Denied`.

- **Resolution:**
  - Verify the LAPS UI is installed on the management workstation:
    - Download and install the LAPS UI from the Microsoft Download Center.
  - Ensure the user has the necessary permissions to use the LAPS UI.

### 9. Replication Issues

- **Symptoms:**
  - Password changes are not replicated across domain controllers.
  - The `ms-Mcs-AdmPwd` attribute is inconsistent across domain controllers.

- **Resolution:**
  - Verify Active Directory replication is functioning correctly:
    - Use `repadmin /showrepl` to check replication status.
  - Ensure the LAPS GPO is applied consistently across all domain controllers.

### 10. Script Execution Issues

- **Symptoms:**
  - Custom scripts for password management fail to execute.
  - Error messages like `Script Execution Failed`.

- **Resolution:**
  - Verify the script is correctly configured in the LAPS GPO:
    - Check **Computer Configuration > Administrative Templates > LAPS > Configure Script Execution**.
  - Ensure the script has the necessary permissions to run on the target computer.

## General Troubleshooting Tips

- **Check Logs:** Review the LAPS client logs (`C:\ProgramData\LAPS\Logs`) for errors.
- **Test Permissions:** Use `Get-AdmPwdPassword` to test password retrieval permissions.
- **Update LAPS:** Ensure LAPS components are updated to the latest version.

[go to Contents](#contents)

---

# Active Directory

**Active Directory (AD)** is a critical component of many IT environments, providing directory services, authentication, and authorization. 

Below is a list of **common problems**, their **symptoms**, and **possible resolutions**:

## Common Issues with Active Directory

### 1. Domain Controller (DC) Not Responding

- **Symptoms:**
  - Clients cannot authenticate or access domain resources.
  - Error messages like `No logon servers available` or `Domain Controller not found`.

- **Resolution:**
  - Verify the DC is online and reachable:

    ```bash
    ping <dc_hostname>
    ```

  - Check DNS settings on clients and DCs:
    - Ensure clients point to the correct DNS server.
  - Restart the DC if necessary:

    ```bash
    shutdown /r /t 0
    ```

### 2. DNS Issues

- **Symptoms:**
  - Clients cannot resolve domain names.
  - Error messages like `DNS name does not exist` or `DNS server not responding`.

- **Resolution:**
  - Verify DNS server settings on clients and DCs:
    - Ensure clients point to the correct DNS server.
  - Check DNS records and zones in the DNS Manager:
    - Ensure `_msdcs`, `_sites`, `_tcp`, and `_udp` records are present.
  - Use `dcdiag /test:dns` to diagnose DNS issues.

### 3. Replication Issues

- **Symptoms:**
  - Changes are not replicated across domain controllers.
  - Error messages like `Replication failed` or `Inconsistent data`.

- **Resolution:**
  - Use `repadmin /showrepl` to check replication status.
  - Verify network connectivity between DCs:

    ```bash
    ping <dc_ip>
    ```

  - Use `repadmin /syncall` to force replication.

### 4. Authentication Issues

- **Symptoms:**
  - Users cannot log in or access domain resources.
  - Error messages like `Invalid username or password` or `Account locked out`.

- **Resolution:**
  - Verify user credentials and account status:
    - Check if the account is locked, disabled, or expired.
  - Reset the user password if necessary:

    ```bash
    net user <username> <new_password> /domain
    ```

  - Ensure the user has the correct permissions and group memberships.

### 5. Group Policy Issues

- **Symptoms:**
  - Group Policy settings are not applied to clients.
  - Error messages like `Group Policy not applied` or `GPO not found`.

- **Resolution:**
  - Verify GPO links and inheritance in the Group Policy Management Console (GPMC).
  - Use `gpresult /r` or `rsop.msc` to check GPO application on the client.
  - Run `gpupdate /force` on the client to refresh Group Policy.

### 6. FSMO Role Issues

- **Symptoms:**
  - Certain operations fail (e.g., schema updates, domain joins).
  - Error messages like `FSMO role owner not found`.

- **Resolution:**
  - Identify the current FSMO role holders:

    ```bash
    netdom query fsmo
    ```

  - Transfer or seize FSMO roles if necessary:

    ```bash
    ntdsutil
    roles
    connections
    connect to server <dc_name>
    transfer <role>
    ```

### 7. Trust Relationship Issues

- **Symptoms:**
  - Clients cannot authenticate with the domain.
  - Error messages like `Trust relationship failed` or `Secure channel broken`.

- **Resolution:**
  - Rejoin the client to the domain:

    ```bash
    netdom reset <computer_name> /domain:<domain_name> /userd:<admin_user> /passwordd:<admin_password>
    ```

  - Verify the computer account in Active Directory Users and Computers (ADUC).

### 8. Schema Issues

- **Symptoms:**
  - Schema updates fail or cause issues.
  - Error messages like `Schema update failed` or `Inconsistent schema`.

- **Resolution:**
  - Verify the schema master role is held by a functioning DC.
  - Use `adprep` to prepare the forest and domain for schema updates.
  - Check for schema conflicts and inconsistencies.

### 9. Backup and Restore Issues

- **Symptoms:**
  - AD backups fail or cannot be restored.
  - Error messages like `Backup failed` or `Restore failed`.

- **Resolution:**
  - Verify backup settings and schedules in Windows Server Backup.
  - Use `wbadmin` to perform manual backups and restores:

    ```bash
    wbadmin start backup -backuptarget:<drive_letter> -include:<volume>
    wbadmin start recovery -version:<backup_version> -itemtype:app -items:<ad_components>
    ```

### 10. Time Synchronization Issues

- **Symptoms:**
  - Clients and DCs have incorrect time settings.
  - Error messages like `Time skew too great` or `Kerberos authentication failed`.

- **Resolution:**
  - Verify time synchronization settings:
    - Ensure DCs sync with a reliable time source (e.g., NTP server).
  - Use `w32tm` to configure and diagnose time settings:

    ```bash
    w32tm /query /status
    w32tm /resync
    ```

## General Troubleshooting Tips

- **Check Logs:** Review Event Viewer logs (**Applications and Services Logs > Directory Service**) for errors.
- **Test Connectivity:** Use tools like `ping`, `nslookup`, and `dcdiag` to diagnose network and DNS issues.
- **Update AD:** Ensure all DCs are updated to the latest version of Windows Server.

[go to Contents](#contents)

---
