# Wireshark Network Incident Investigation

## Overview

A practical SOC-style network traffic investigation performed using Wireshark. The project analyzes a PCAP capture to identify unusual communications, correlate DNS activity with network connections, inspect HTTP/TLS traffic, and document potential indicators of suspicious activity.

> **Note:** The original PCAP is not included in this repository because it may contain sensitive network traffic.

## Objectives

- Analyze captured network traffic using Wireshark.
- Identify the internal workstation and DNS server.
- Investigate DNS queries and responses.
- Correlate domains, IP addresses, ports, and timestamps.
- Examine HTTP requests and TLS Client Hello/SNI information.
- Build an evidence-based incident timeline.
- Document indicators, limitations, and recommended next steps.

## Tools & Technologies

- Wireshark
- PCAP analysis
- DNS
- TCP/IP
- HTTP
- TLS
- Network incident investigation
- IOC analysis

## Investigated Host

| Role | Address |
|---|---|
| Internal workstation | `10.9.11.135` |
| Internal DNS server | `10.9.11.2` |

## Key Findings

### 1. `know.mom-nower.com`

- DNS resolved the domain to `86.106.87.134`.
- HTTP communication was observed over TCP port 80.
- An HTTP GET request used a long, randomized-looking URI.
- The server returned `HTTP/1.1 406 Not Acceptable`.
- The traffic is potentially suspicious and warrants further investigation, but the PCAP alone does not prove malicious activity or endpoint compromise.

### 2. `www.xos-aploq.com`

- DNS returned `172.67.155.32` and `104.21.34.58`.
- A TLS connection was observed to `172.67.155.32:443`.
- The TLS Client Hello contained SNI `www.xos-aploq.com`.
- This provides strong domain-to-connection correlation.
- The encrypted application data could not be inspected from the available capture.

### 3. `soft.xos-aploq.com`

- DNS returned `104.21.34.58` and `172.67.155.32`.
- These IPs are shared infrastructure, so IP-only attribution is unreliable.
- A TLS connection to the shared IP showed SNI `www.xos-aploq.com`, not `soft.xos-aploq.com`.
- Therefore, that connection should **not** be attributed to `soft.xos-aploq.com` without additional evidence.

## Investigation Methodology

1. Identify the internal host and DNS server.
2. Review DNS queries and responses.
3. Correlate resolved domains with subsequent network connections.
4. Inspect TCP conversations and application protocols.
5. Examine HTTP request/response details.
6. Inspect TLS Client Hello and SNI values where available.
7. Build a timeline using packet timestamps.
8. Record indicators and confidence levels.
9. Document limitations and recommended follow-up actions.

## Investigation Timeline

| Approx. capture time | Event |
|---:|---|
| ~46.55s | DNS query for `www.xos-aploq.com` |
| ~46.58s | DNS response returned `172.67.155.32` / `104.21.34.58` |
| ~46.67s | TLS connection to `172.67.155.32:443` |
| ~46.67s | TLS Client Hello SNI: `www.xos-aploq.com` |
| ~56.79s | DNS query for `soft.xos-aploq.com` |
| ~56.80s | DNS response returned `104.21.34.58` / `172.67.155.32` |
| ~61.63s | DNS query for `know.mom-nower.com` |
| ~61.65s | DNS response returned `86.106.87.134` |
| Later | HTTP communication with `86.106.87.134:80` |
| Later | HTTP GET to `know.mom-nower.com`; server returned 406 |

> Timestamps are relative to the beginning of the capture.

## Network Indicators

| Indicator | Type | Observation | Assessment |
|---|---|---|---|
| `10.9.11.135` | Internal IP | Investigated workstation | Internal host |
| `10.9.11.2` | Internal IP | DNS server | Internal infrastructure |
| `know.mom-nower.com` | Domain | HTTP traffic and unusual GET URI | Potentially suspicious |
| `86.106.87.134` | IP | HTTP/80 communication | Confirmed communication |
| `www.xos-aploq.com` | Domain | TLS SNI observed | Requires investigation |
| `172.67.155.32` | IP | TLS/443; SNI matched `www.xos-aploq.com` | Shared infrastructure |
| `104.21.34.58` | IP | Returned by DNS | Shared infrastructure |
| `soft.xos-aploq.com` | Domain | DNS resolution observed | Attribution to a specific flow inconclusive |

## Limitations

- TLS 1.3 application data could not be decrypted because session secrets were unavailable.
- A PCAP alone cannot prove that an endpoint was compromised.
- Shared CDN/hosting infrastructure makes IP-only attribution unreliable.
- Endpoint process information and Windows event logs were not available.
- Domain reputation and threat-intelligence checks would be required for stronger classification.

## Recommended Next Steps

1. Check the observed domains against reputable threat-intelligence sources.
2. Review endpoint logs from `10.9.11.135`.
3. Identify the process responsible for the suspicious connections.
4. Check for persistence mechanisms and unusual processes.
5. Review additional DNS and HTTP activity in the capture.
6. If malicious activity is confirmed, contain the host and block confirmed indicators.

## Skills Demonstrated

- Wireshark packet analysis
- DNS investigation
- TCP/IP traffic analysis
- HTTP analysis
- TLS/SNI analysis
- Network IOC identification
- Timeline construction
- Evidence-based incident reporting
- SOC investigation methodology
