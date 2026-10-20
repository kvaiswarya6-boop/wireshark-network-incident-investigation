# Network Traffic Analysis & Security Incident Investigation Using Wireshark

## 1. Objective

Analyze a captured network traffic dataset (PCAP) using Wireshark to identify unusual network communications, investigate DNS resolutions, correlate domains with IP addresses, examine TCP/TLS/HTTP traffic, and document potential indicators of suspicious activity.

## 2. Scope

The investigation focused on traffic originating from internal workstation `10.9.11.135`, with DNS activity involving internal DNS server `10.9.11.2`.

## 3. Methodology

- Identified internal hosts and major conversations.
- Reviewed DNS queries and responses.
- Correlated DNS results with subsequent connections.
- Inspected TCP and application-layer traffic.
- Examined HTTP request and response metadata.
- Examined TLS Client Hello and SNI information.
- Built a relative timeline from packet timestamps.
- Recorded findings without treating suspicious indicators as confirmed malicious activity unless directly supported by the capture.

## 4. Finding: `know.mom-nower.com`

The workstation `10.9.11.135` queried the internal DNS server `10.9.11.2` for `know.mom-nower.com`. The response resolved the domain to `86.106.87.134`.

Subsequent traffic showed HTTP communication with `86.106.87.134` over TCP port 80. An HTTP request used the Host header `know.mom-nower.com` and contained a long, randomized-looking URI. The server returned `HTTP/1.1 406 Not Acceptable` with no response body.

### Assessment

This activity is potentially suspicious because of the unusual URI and external HTTP communication. However, these characteristics alone do not prove maliciousness or compromise. Additional endpoint and threat-intelligence evidence is required.

## 5. Finding: `www.xos-aploq.com`

DNS activity showed that `www.xos-aploq.com` resolved to `172.67.155.32` and `104.21.34.58`.

A subsequent connection from the workstation to `172.67.155.32:443` contained a TLS 1.3 Client Hello with SNI `www.xos-aploq.com`.

### Assessment

This is a strong correlation between the DNS result, destination IP, and TLS SNI. The encrypted application data was not available for inspection, so the actual content or purpose of the session could not be determined from the PCAP alone.

## 6. Finding: `soft.xos-aploq.com`

A DNS query for `soft.xos-aploq.com` returned `104.21.34.58` and `172.67.155.32`.

Because these addresses are shared infrastructure, they cannot be treated as unique identifiers for the domain. In addition, the examined TLS connection to `172.67.155.32` presented SNI `www.xos-aploq.com`, not `soft.xos-aploq.com`.

### Assessment

The DNS resolution is confirmed, but attribution of the examined TLS connection to `soft.xos-aploq.com` is inconclusive. This is an important example of why analysts should not rely on IP addresses alone when investigating shared hosting or CDN infrastructure.

## 7. Timeline

- **~46.55s:** DNS query for `www.xos-aploq.com`.
- **~46.58s:** DNS response returned `172.67.155.32` and `104.21.34.58`.
- **~46.67s:** Connection to `172.67.155.32:443`.
- **~46.67s:** TLS Client Hello contained SNI `www.xos-aploq.com`.
- **~56.79s:** DNS query for `soft.xos-aploq.com`.
- **~56.80s:** DNS response returned `104.21.34.58` and `172.67.155.32`.
- **~61.63s:** DNS query for `know.mom-nower.com`.
- **~61.65s:** DNS response returned `86.106.87.134`.
- **Later:** HTTP communication with `86.106.87.134:80` and request to `know.mom-nower.com`.
- **Later:** Server returned HTTP 406.

## 8. Indicators

### Confirmed network observations

- `10.9.11.135` — investigated internal workstation
- `10.9.11.2` — DNS server
- `know.mom-nower.com` — DNS and HTTP activity
- `86.106.87.134` — confirmed HTTP destination
- `www.xos-aploq.com` — DNS and TLS SNI correlation
- `172.67.155.32` — TLS destination associated with `www.xos-aploq.com`
- `104.21.34.58` — DNS-resolved shared infrastructure
- `soft.xos-aploq.com` — DNS resolution observed

### Important analyst note

Not every observed domain or IP should automatically be classified as an IOC. A network indicator becomes more useful when supported by context, endpoint evidence, reputation data, or repeated suspicious behavior.

## 9. Limitations

- TLS application data was encrypted and session secrets were unavailable.
- The PCAP does not contain endpoint process information.
- Windows event logs and endpoint telemetry were not available.
- Shared infrastructure can host many unrelated domains.
- No external reputation result was used as proof of maliciousness.

## 10. Recommended Response Actions

1. Perform reputation and threat-intelligence checks on the domains.
2. Identify the originating process on `10.9.11.135`.
3. Review browser history, proxy logs, and endpoint telemetry where available.
4. Check for persistence, scheduled tasks, unusual services, or recently created files.
5. Review related DNS and HTTP activity for additional indicators.
6. If compromise is confirmed, isolate the endpoint and block confirmed malicious indicators.

## 11. Conclusion

The investigation identified several unusual external communications from internal workstation `10.9.11.135`. DNS analysis was used to correlate domains with resolved IP addresses, followed by TCP, TLS, and HTTP analysis.

The strongest observations were the TLS connection associated with `www.xos-aploq.com` and the HTTP activity associated with `know.mom-nower.com`, including an unusual GET request and a 406 response. These findings are worthy of further investigation, but the available PCAP alone is insufficient to conclusively determine that the workstation was compromised.
