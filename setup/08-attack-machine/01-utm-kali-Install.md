# UTM Kali Installation

Step 1 – Download Kali ISO

Step 2 – Create VM in UTM

In UTM on Mac:

* Virtualize
* Linux
* 2 CPU cores
* 4GB RAM (recommended for tools)

Storage:

* 40GB disk

Network:

* Shared Network (bridged to Mac interface)
* Must receive IP from 192.168.1.x


After installation, verify:

```bash
ip a
```

You should see:

```
192.168.1.xxx
```

If you get 172.x.x.x → wrong network mode.

We want attacker on HOME network.

## Network Integration with pfSense

Now the important part.

Right now:

pfSense WAN = 192.168.1.101 
pfSense LAN = 172.16.1.1

Attack machine = 192.168.1.xxx

By default:

pfSense blocks inbound traffic from WAN to LAN.

That is GOOD.

We will create **controlled test rules**.

## Step 1 – Allow ICMP from Attacker

In pfSense:

Firewall → Rules → WAN

Add rule:

Action: Pass
Interface: WAN
Protocol: ICMP
Source: 192.168.1.xxx (Kali IP)
Destination: 172.16.1.0/24

Save + Apply.

After Kali is installed:

Update system:

```bash
sudo apt update && sudo apt upgrade -y
```

Install essential IR-relevant tooling:

```bash
sudo apt install bloodhound neo4j impacket-scripts crackmapexec responder enum4linux smbclient -y
```

Verify:

```bash
impacket-secretsdump -h
```

 Operational Security Notes

This attacker VM:

* Should NEVER have direct route to 172.16.1.x without firewall logging
* All attack traffic should pass through pfSense

You don’t just attack.
You investigate your own attack.
