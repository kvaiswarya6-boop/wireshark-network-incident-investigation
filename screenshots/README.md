# Wireshark Investigation Evidence

Public-safe screenshots selected from the PCAP investigation.

## Evidence files

1. `dns-xos-aploq.png` — DNS response for `www.xos-aploq.com`
2. `tls-xos-aploq.png` — TLS Client Hello showing SNI `www.xos-aploq.com`
3. `dns-soft-xos-aploq.png` — DNS response for `soft.xos-aploq.com`
4. `tls-soft-xos-aploq.png` — TLS Client Hello showing SNI `soft.xos-aploq.com`
5. `dns-know-mom-nower.png` — DNS response mapping `know.mom-nower.com` to `86.106.87.134`
6. `http-get-and-406-redacted.png` — HTTP GET to `know.mom-nower.com` and `406 Not Acceptable` response; cookie/session values redacted
7. `benign-nodejs-sni.png` — contextual TLS evidence showing SNI `nodejs.org`

## Public-sharing note

Cookie/session values were removed from the HTTP evidence before publishing. The original PCAP should not be uploaded unless redistribution is explicitly permitted.
