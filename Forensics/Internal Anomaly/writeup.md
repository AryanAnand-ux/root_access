# Internal Anomaly

**Category:** Forensics  
**Artifact:** `root.pcapng`  
**Flag:** `root{p4ck3t_4n4l7515_55ucc355f0l}`

## Overview
The pcap contains multiple encrypted channels. The key to solving it is noticing a plain HTTP download that quietly contains TLS session secrets. Once those keys are fixed and loaded into tshark, the encrypted HTTPS stream decrypts cleanly and reveals the flag.

## Step 1: Triage the Capture
- Checked protocol breakdown and TCP conversations with `tshark`.
- Spotted heavy SSH traffic between `10.31.43.117` and `10.31.42.201` on port `22`.
- Found a cleartext HTTP transfer from a Python `SimpleHTTP` server on port `8000`: `GET /runtime_cache.dat`.
- Noted HTTPS on port `8443` to `10.31.42.158` (encrypted payload).

## Step 2: Key Insight
The `runtime_cache.dat` download was not random data. It contained TLS keylog entries in `SSLKEYLOGFILE` format (e.g., `SERVER_HANDSHAKE_TRAFFIC_SECRET`, `CLIENT_TRAFFIC_SECRET_0`, etc.). With those session secrets, TLS 1.3 traffic becomes decryptable.

## Step 3: Fix the Keylog File
The keylog lines were wrapped across multiple lines when extracted, so I joined continuation lines that didn’t begin with a key type:

```python
fixed = re.sub(r'\n(?!SERVER_|CLIENT_|EXPORTER_)', '', full_text)
```

Saved the fixed output to `/tmp/sslkeylog.txt` (55 entries total).

## Step 4: Decrypt TLS and Extract the Flag
Loaded the keylog file in tshark and followed the decrypted stream:

```bash
tshark -r root.pcapng -o "tls.keylog_file:/tmp/sslkeylog.txt" \
  -q -z "follow,tls,ascii,3"
```

The decrypted HTTP response contained the report:

```text
===== System Health Summary =====
Node: srv-prod-02
Timestamp: 2026-02-06 02:11:43
...
audit_maker= ROOT{P4cK3T_4n4L7515_55ucC355F0L}
...
```

Lowercased as required:

```text
root{p4ck3t_4n4l7515_55ucc355f0l}
```

## Tools Used
- `tshark`: stream following and TLS decryption
- Python 3: keylog cleanup and formatting
- `xxd`: binary inspection
