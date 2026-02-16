# Security Layer – Wazuh Full Logging & Archive Configuration

## Overview

This document describes the complete configuration of Wazuh to:

* Enable full log archiving
* Index raw logs into OpenSearch
* Create dashboard data views
* Validate archive ingestion
* Prepare environment for DFIR-level visibility

This configuration ensures that both **alerts and raw logs** are available for:

* Manual threat hunting
* Timeline reconstruction
* Incident investigation
* Behavioral analysis

---

# Logging Architecture Overview

Wazuh 4.x logging pipeline:

```
Windows Agent / Linux Agent
        ↓
Wazuh Manager
        ↓
Archive Files (archives.json)
        ↓
Filebeat
        ↓
OpenSearch Indexer
        ↓
Wazuh Dashboard
```

Two types of indexed data:

| Type     | Index Pattern        | Purpose                |
| -------- | -------------------- | ---------------------- |
| Alerts   | wazuh-alerts-*       | Triggered rule matches |
| Archives | wazuh-archives-4.x-* | Raw, unfiltered logs   |

For IR training, **archives are mandatory**.

---

# Enable Full Log Archiving

## Step 1 – Edit Wazuh Manager Configuration

On Wazuh server:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Inside `<global>` section, ensure:

```xml
<global>
  <jsonout_output>yes</jsonout_output>
  <alerts_log>yes</alerts_log>
  <logall>yes</logall>
  <logall_json>yes</logall_json>
</global>
```

Save and exit.

---

## Step 2 – Restart Wazuh Manager

```bash
systemctl restart wazuh-manager
```

Verify:

```bash
systemctl status wazuh-manager
```

Status must show:

```
active (running)
```

---

# Verify Archive Log Creation

Check archive directory:

```bash
ls -lah /var/ossec/logs/archives/
```

Expected output:

```
archives.json
archives.log
```

File size should increase over time.

Example:

```
archives.json
archives.log
```

This confirms raw logs are being generated.

---

# Verify Archive Indexing in OpenSearch

Check indexed data:

```bash
curl -k -u <username>:<password> https://127.0.0.1:9200/_cat/indices?v | grep wazuh
```

Expected output should include:

```
wazuh-archives-4.x-YYYY.MM.DD
wazuh-alerts-4.x-YYYY.MM.DD
```

If archives index exists → Filebeat shipping is working.

---

# Filebeat Configuration Verification

Check Filebeat module configuration:

```bash
nano /etc/filebeat/filebeat.yml
```

Ensure this section exists:

```yaml
filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: true
```

Restart Filebeat if changes were made:

```bash
systemctl restart filebeat
```

Verify:

```bash
systemctl status filebeat
```

---

# Create Archive Data View in Dashboard

Go to:

Dashboards Management → Index patterns

Click:

Create Index Pattern

Enter:

```
wazuh-archives-4.x-*
```

Click Next.

Select timestamp field:

```
@timestamp
```

Save.

---

# Validate Archive Visibility

Go to:

Discover → Select `wazuh-archives-4.x-*`

You should now see:

* Windows logon events
* Background authentication
* Service activity
* Process creation (4688)
* System events
* Non-alerted normal traffic

This index contains:

All raw logs BEFORE alert filtering.
