# SOC-Style Network Traffic Analysis and Anomaly Investigation Lab

## Executive Summary

This project presents a controlled network laboratory designed to observe, capture, and analyze HTTP traffic behavior from a **Security Operations Center (SOC)** perspective.

The objective of the lab is to distinguish **normal client-server communication** from **abnormal traffic patterns** using packet captures and server-side logs as primary evidence.

Two full packet capture files were collected during the project:

* One representing **baseline (normal) HTTP traffic**
* One representing **anomalous high-volume HTTP traffic**

The project focuses on **visibility, evidence collection, correlation, and impact analysis**, rather than exploitation or penetration testing.

---

## Environment Overview

The lab environment was built using the **VirtualBox hypervisor** and consists of the following virtual machines:

### OPNsense Firewall / Router

* Default gateway for all internal hosts
* Provides routing, NAT, DNS, and DHCP services
* Central point of traffic inspection and control

### Ubuntu Server

* Hosts an Apache HTTP web server
* Serves a static HTML file for testing HTTP communication
* Internal IP address: `192.168.1.174`

### Ubuntu Desktop

* Acts as a legitimate client
* Used for packet capture with Wireshark / dumpcap
* Network interface configured in promiscuous mode

### Additional Linux Client

* Used exclusively to generate increased HTTP request volume
* Represents a misbehaving internal client

---

## Network Architecture

The network was almost entirely configured **within the OPNsense virtual machine**, using two network interfaces to enforce segmentation:

### LAN Interface

* Connects all internal hosts
* Handles internal routing and service delivery

### WAN Interface

* Connected to a NAT network
* Provides Internet access to internal systems

All internal virtual machines access the Internet **exclusively through OPNsense**, allowing clear observation of traffic flow and operational impact.

---

## Web Server Configuration (Ubuntu Server)

The Ubuntu Server virtual machine was configured to host an Apache web server used for HTTP testing.

### Apache Installation and Setup

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install apache2 -y
```

### Apache Service Verification

```bash
systemctl status apache2
```

### Test HTML File Creation

```bash
sudo nano /var/www/html/test.html
```

```html
<html>
  <body>
    <h1>Test HTTP Server</h1>
  </body>
</html>
```

### Apache Restart and Port Verification

```bash
sudo systemctl restart apache2
ss -tulnp | grep :80
```

---

## Client Configuration and Traffic Capture

The Ubuntu Desktop virtual machine was used to capture network traffic using **Wireshark / dumpcap**.

### dumpcap Permissions Configuration

Initial permission issues were encountered when selecting the capture interface.
These were resolved by verifying capabilities and group membership.

#### Verification Steps

```bash
ls -l /usr/bin/dumpcap
getcap /usr/bin/dumpcap
```

#### Capability Adjustment (If Required)

```bash
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
```

### Wireshark Group Configuration

```bash
sudo dpkg-reconfigure wireshark-common
sudo usermod -aG wireshark $USER
```

After logging out and back in, available capture interfaces were verified:

```bash
dumpcap -D
```

Traffic was captured primarily on the **enp0s3** interface, with the NIC operating in **promiscuous mode**.

---

## Baseline Traffic Capture (Normal Behavior)

### Objective

Establish a baseline of legitimate HTTP traffic to enable comparison with abnormal behavior.

### Method

Normal client-server communication was generated using standard HTTP requests:

```bash
curl http://192.168.1.174
curl -v http://192.168.1.174
```

### Evidence

* File: `normal-client-server-traffic.pcapng`

This packet capture contains:

* TCP three-way handshakes
* Stable TCP sessions
* HTTP GET requests
* HTTP 200 OK responses

### Observations

* Low and consistent packet rate
* Predictable request and response pattern
* Minimal number of concurrent TCP sessions
* Stable session duration

---

## Anomalous Traffic Capture (Abnormal Behavior)

### Objective

Simulate abnormal client behavior by generating a high volume of legitimate HTTP requests over a short period of time.

### Method

Traffic was generated using standard Linux command-line tools, without packet crafting or exploitation:

```bash
for i in {1..5000}; do
  curl http://192.168.1.174/test.html
done
```

### Evidence

* File: `anomalous-client-server-traffic.pcapng`

This packet capture contains the complete packet-level data for the abnormal traffic scenario.

### Observations

Compared to baseline traffic:

* Significant increase in packet rate
* Repeated HTTP GET requests for the same resource
* Large number of short-lived TCP connections
* Rapid creation and teardown of TCP sessions
* Clear deviation from expected traffic behavior

---

## Server-Side Log Correlation

Apache server logs were monitored during abnormal traffic generation:

```bash
sudo tail -f /var/log/apache2/access.log
```

### Observed Behavior

* Rapid growth of log entries
* Repeated requests for the same file
* Very short time intervals between requests

This confirmed direct correlation between **network-level packet captures** and **application-layer logs**.

---

## Impact Analysis

### Target Service Impact

* The Apache web server continued responding to requests
* Request volume was significantly higher than baseline
* Demonstrates how legitimate protocol traffic can still cause service strain through volume

### Client-Side Impact

* The client generating traffic experienced temporary loss of Internet connectivity
* Connectivity was restored after stopping the request loop

Likely causes include:

* Local resource exhaustion
* Connection tracking limits

This demonstrates that excessive outbound traffic can negatively impact the originating host itself.

---

## Firewall and Defensive Considerations

No active defensive rules were enabled on the OPNsense firewall during testing.
This was intentional to allow unrestricted observation of traffic behavior and impact.

### Conceptual Considerations

* IP-based blocking has limited effectiveness due to ease of IP changes
* TCP session blocking provides short-term containment only
* Distributed attacks cannot be mitigated solely at the perimeter
* Firewalls provide visibility and containment, not definitive protection

Advanced mechanisms such as **IDS/IPS** or **WAFs** were intentionally excluded.

---

## Limitations

* Traffic originated from a single internal source
* No distributed attack simulation (DDoS)
* No application-layer protection mechanisms
* Analysis constrained by virtualized environment resources

---

## Lessons Learned

* Establishing a baseline is essential for anomaly detection
* Legitimate protocol traffic can still cause operational impact through volume
* Packet captures and application logs complement each other during investigations
* Abnormal traffic can affect services, infrastructure, and the originating host
* SOC activities focus on detection, evidence, and escalation rather than permanent fixes

---

## Conclusion

This lab demonstrates how full packet captures and server-side logs can be used together to identify and analyze abnormal HTTP traffic in a controlled environment.

By comparing baseline and anomalous scenarios, the project highlights realistic operational impacts and the limitations of perimeter-based defenses, aligning closely with real-world **SOC workflows**.
