# Snapshot and Rollback Strategy

Security testing intentionally introduces instability.

Common risks include:

* Firewall lockouts
* LDAP authentication failures
* Domain Controller corruption
* Misconfigured GPO policies
* Broken Wazuh decoders
* Rule parsing failures
* Service crashes
* Accidental service exposure
* Hardening misconfiguration

Snapshots provide:

* Rapid rollback capability
* Clean forensic baseline comparison
* Controlled testing workflow
* Enterprise-style change safety net

In real enterprise environments, this role is performed by:

* Backup systems
* DR replication
* Snapshot orchestration platforms
* Change management workflows

In this lab, Proxmox snapshots fulfill that function.

## Snapshot Naming Convention

To maintain clarity and traceability, use structured naming.

### Baseline Snapshots

```
BASELINE-STABLE-INFRA-YYYY-MM-DD
```

# Snapshot Scope Requirements

The following virtual machines must be snapshotted before major changes:

| Layer     | VM                                      | Snapshot Required |
| --------- | --------------------------------------- | ----------------- |
| Network   | pfSense                                 | Yes               |
| Identity  | Domain Controller (Windows Server 2022) | Yes               |
| Security  | Wazuh Server                            | Yes               |
| Jump Host | Debian Bastion                          | Recommended       |
| DevSecOps | Ubuntu DevOps VM                        | Recommended       |
| Client    | Windows 11                              | Optional          |
| Attack    | Kali / UTM Attack VM                    | Optional          |

Minimum required before attack simulations:

* pfSense
* Domain Controller
* Wazuh

## Creating a Snapshot in Proxmox

Step 1: Stop Critical Services
Step 2: Create Snapshot
1. Log into Proxmox Web UI
2. Select the VM
3. Navigate to **Snapshots**
4. Click **Take Snapshot**
5. Enter snapshot name (follow naming convention)
6. Add description:
7. Choose:
   * ✔ Include RAM (optional)
   * ✔ Include filesystem state
8. Click **Create**

## Snapshot Description Best Practice

Always include:

* Infrastructure state
* Logging validation status
* LDAP status
* Firewall status
* Wazuh agent connectivity
* Known issues (if any)

Example:

> All systems operational.
> Wazuh archives enabled.
> pfSense logs flowing to UDP 5514.
> AD LDAP integrated with Wazuh and Gitea.
> No pending updates.
> Ready for MITRE T1059 simulation.

## Rollback Procedure

 Warning: Rollback will discard all changes made after snapshot.

Step 1: Power Off VM
Never rollback while running.

Step 2: Select Snapshot
* VM → Snapshots
* Select desired snapshot

Step 3: Click Rollback
* Confirm rollback
* Wait for operation to complete

Step 4: Start VM
* Monitor boot
* Verify:

  * Services running
  * Network reachable
  * Wazuh agents connected
  * Domain authentication working

## Post-Rollback Verification Checklist

After rollback:

### Network

* pfSense accessible
* Firewall rules intact
* Log forwarding working

### Identity

* AD services running
* LDAP authentication working
* Domain login functional

### Security

* Wazuh manager running
* Filebeat running
* Archives index active
* Logs flowing

### Validation Commands

On Wazuh:

```
systemctl status wazuh-manager
systemctl status filebeat
curl -k -u user:pass https://localhost:9200/_cat/indices?v
```

On pfSense:

* Confirm Remote Logging still enabled
* Confirm UDP 5514 traffic visible via tcpdump


## Baseline Snapshot Requirement

Before starting Enterprise Attack Simulations, create:

```
BASELINE-STABLE-INFRA
```

Requirements:

* All VMs operational
* pfSense logging to Wazuh verified
* Wazuh archives index active
* Sysmon installed on Windows
* Advanced auditing enabled
* No active alerts

This snapshot becomes the master restore point.


## Change Control Workflow (Enterprise Model)

Every major change must follow:

```
Snapshot
Perform Change / Attack
Validate System
Document Observations
If unstable → Rollback
If stable → Continue
```
## Snapshot Retention Strategy

Do not accumulate excessive snapshots.

Recommended:

* Keep one baseline
* Keep one per active attack scenario
* Remove obsolete snapshots after documentation complete


# When NOT to Use Snapshot

Snapshots are not long-term backup solutions.

Do not rely on them for:

* Permanent storage
* Long-term data retention
* Disaster recovery simulation

They are tactical rollback mechanisms.

---

# Final Rule

Never begin:

* MITRE simulation
* Hardening phase
* Firewall restructuring
* AD schema modification
* Wazuh decoder creation

Without first taking a snapshot.
