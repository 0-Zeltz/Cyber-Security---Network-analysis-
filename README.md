SOC-Style Network Traffic Analysis and Anomaly Investigation Lab
Executive Summary

This project presents a controlled network laboratory designed to observe, capture, and analyze HTTP traffic behavior from a Security Operations Center (SOC) perspective. The lab focuses on distinguishing normal client-server communication from abnormal traffic patterns, using packet captures and server-side logs as primary sources of evidence.

Two full packet capture files were collected:

One representing baseline (normal) HTTP traffic

One representing anomalous high-volume HTTP traffic

These captures were analyzed to identify behavioral differences at the network and transport layers, correlate findings with application logs, and assess the operational impact on both the target service and the originating client.

Environment Overview

The environment was built using the VirtualBox hypervisor and consists of a centralized network architecture with the following components:

OPNsense Firewall/Router

Default gateway for all internal hosts

Provides routing, NAT, DNS, and DHCP services

Single enforcement and inspection point

Ubuntu Server

Hosts an Apache HTTP web server

Serves a static HTML file used for testing HTTP communication

Internal IP address: 192.168.1.174

Ubuntu Desktop

Acts as a legitimate client

Used for packet capture with Wireshark/dumpcap

NIC configured in promiscuous mode

Additional Linux Host

Used exclusively to generate increased HTTP request volume

Represents a misbehaving internal client

All internal machines access the Internet exclusively through the OPNsense firewall, allowing clear visibility of traffic flow and side effects during abnormal conditions.

Network Architecture

The network was almost entirely configured within the OPNsense virtual machine, using two network interfaces to enforce segmentation:

LAN Interface

Connects all internal hosts

Handles internal routing and service delivery

WAN Interface

Connected to a NAT network

Provides controlled Internet access to internal systems

This design mirrors a simplified enterprise perimeter model and allows observation of how abnormal internal traffic affects both internal services and external connectivity.

Baseline Traffic Capture (Normal Behavior)
Objective

Establish a baseline of legitimate HTTP traffic to serve as a reference point for anomaly detection.

Method

Normal client-server communication was generated using standard HTTP requests to the Apache server. Traffic was captured on the client machine using Wireshark/dumpcap.

Evidence

File: normal-client-server-traffic.pcapng
This packet capture contains all network traffic observed during normal HTTP communication, including:

TCP three-way handshakes

Stable TCP sessions

HTTP GET requests

HTTP 200 OK responses

Observations

Analysis of the baseline capture revealed:

Low and consistent packet rate

Minimal number of concurrent TCP sessions

Predictable request/response pattern

Stable session duration

No signs of retransmissions or connection churn

This capture establishes what expected and healthy traffic looks like in the environment.

Anomalous Traffic Capture (Abnormal Behavior)
Objective

Simulate abnormal client behavior by generating an excessive volume of legitimate HTTP requests and observe deviations from baseline behavior.

Method

A loop-based request mechanism using standard Linux tools (curl) was used to generate a large number of HTTP GET requests targeting the same web resource in a short time window. No packet crafting, exploitation, or protocol abuse was involved.

Evidence

File: anomalous-client-server-traffic.pcapng
This packet capture contains the complete packet-level data for the abnormal traffic scenario.

Observations

Compared to the baseline capture, the anomalous capture showed:

A significant increase in packet rate

Repeated HTTP GET requests for the same resource

Large number of short-lived TCP connections

Rapid creation and teardown of TCP sessions

Clear deviation from normal traffic patterns

While the traffic remained protocol-compliant, its volume and frequency caused observable operational effects.

Server-Side Correlation (Apache Logs)

During the anomalous traffic generation, the Apache server’s access logs were monitored.

Observed behavior:

Rapid growth of access.log

Repeated requests for the same file

Timestamps clustered very closely together

This provided direct correlation between:

Network-level observations in the .pcap files

Application-layer evidence on the server

This correlation is a critical SOC activity, linking packet-level data to application behavior.

Impact Analysis
Target Service

The Apache web server continued responding to requests

However, the request volume was significantly higher than baseline

Demonstrates how legitimate protocol usage can still create service strain

Originating Client

The client generating abnormal traffic experienced temporary loss of Internet connectivity

Connectivity was restored after traffic generation stopped

Likely causes include local resource exhaustion, NAT table pressure, or connection tracking limits

This highlights an often-overlooked effect:
excessive outbound traffic can negatively impact the source host itself, not just the target service.

Firewall and Defensive Considerations

At the time of testing, no active defensive rules were enabled on the OPNsense firewall. This was intentional to allow unrestricted observation of traffic behavior and impact.

From a conceptual analysis standpoint:

IP-based blocking offers limited protection due to ease of IP changes

TCP session blocking is effective only for short-term containment

Volume-based and distributed attacks cannot be fully mitigated at a single perimeter device

Firewalls serve as containment and visibility tools, not definitive solutions

Advanced mechanisms such as IDS/IPS, WAFs, or upstream mitigation were intentionally excluded from this lab.

Limitations

Traffic originated from a single internal source

No simulation of distributed attacks (DDoS)

No application-layer protections in place

Analysis constrained by virtualized environment resources

Focused on observation and analysis rather than permanent mitigation

Lessons Learned

Establishing a baseline is essential for meaningful anomaly detection

Legitimate protocol traffic can still cause operational impact through volume

Packet captures and application logs complement each other in investigations

Abnormal traffic can affect intermediate devices and the originating host

SOC responsibilities prioritize detection, evidence, and escalation over long-term fixes

Conclusion

This lab demonstrates how full packet captures and server-side logs can be used together to identify and analyze abnormal network behavior. By comparing baseline and anomalous traffic scenarios, it highlights realistic operational impacts and the limitations of perimeter-based defenses.

The project aligns closely with real-world SOC workflows, emphasizing visibility, correlation, and analytical reasoning over tool-driven exploitation.
