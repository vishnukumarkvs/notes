coroot
======


### Opensource improvements

- Add coroot pvc storage size, or sqlite3 storage size in coroot UI

eks any

TLS verification


1. Check Server Configuration
# If using Coroot operator, check if TLS config exists
kubectl get coroot coroot -n coroot -o yaml | grep -A5 tls:
# Check if grpc section shows enabled
kubectl get coroot coroot -n coroot -o yaml | grep -A2 grpc:
2. Use openssl s_client to Test Connection
# Test TLS connection to Coroot
openssl s_client -connect coroot.coroot.svc.cluster.local:4317 -alpn h2
# If TLS is enabled, you'll see:
#   Protocol: TLSv1.3
#   Cipher: TLS_AES_256_GCM_SHA384
#   Server certificate info
# If TLS is NOT enabled, you'll see:
#   socket: Connection refused
#   or
#   Protocol: UNKNOWN
3. Capture and Analyze Network Traffic with tcpdump
# Capture traffic on port 4317
tcpdump -i any -nn port 4317 -w tls-capture.pcap
# Or for a quick check (hex dump of first packet)
tcpdump -i any -nn port 4317 -c 1 -X
# What to look for:
# TLS handshake:    Packet contains "17 03 03" (TLS record type)
# Plaintext HTTP/2: Packet starts with "PRI * HTTP/2"
Packet Analysis
# TLS Packet (encrypted)
18:03:23.123456 IP ... > ...: Flags [P.], length: 142
    0x0000:  17 03 03 00 8a 4b 7c 89  ...    <- "17 03 03" = TLS record
    0x0010:  a3 d2 f1 89 ...
# Plaintext Packet (no TLS)
18:03:24.234567 IP ... > ...: Flags [P.], length: 142
    0x0000:  50 52 49 20 2a 20 48 54    <- "PRI * HT" = HTTP/2 preface
    0x0010:  54 50 2f 32 2e 30 0d 0a    <- "TP/2.0\r\n"
4. Use wireshark or tshark to Analyze pcap
# Install tshark (part of wireshark)
# Then analyze the capture
tshark -r tls-capture.pcap -Y "tls" -V
# Check for TLS handshake
tshark -r tls-capture.pcap -Y "tls.handshake" -V
# Check if traffic is encrypted
tshark -r tls-capture.pcap -Y "frame" -T fields -e frame.len | head -5
5. Test with curl (for HTTP endpoints)
# Test with verbose TLS info
curl -v https://coroot.example.com/v1/traces
# Look for:
# * TLSv1.3 (or TLSv1.2)
# * connection using cipher
# * certificate info
6. Check Application Logs
# Coroot gRPC server logs
kubectl logs -n coroot -l app=coroot | grep -i tls
# Should show something like:
# "grpc server: tls enabled"
# "listening on :4317"
7. Verify Certificate on Server
# Check what certificate Coroot is serving
openssl s_client -connect coroot.coroot.svc.cluster.local:4317 </dev/null 2>/dev/null | \
  openssl x509 -text -noout
# Check certificate expiration
openssl s_client -connect coroot.coroot.svc.cluster.local:4317 </dev/null 2>/dev/null | \
  openssl x509 -noout -dates
8. Quick Verification Script
#!/bin/bash
HOST="coroot.coroot.svc.cluster.local"
PORT=4317
echo "=== Testing connection to $HOST:$PORT ==="
# Try TLS connection
echo "Testing TLS..."
OUTPUT=$(echo "" | timeout 5 openssl s_client -connect "$HOST:$PORT" -alpn h2 2>&1)
if echo "$OUTPUT" | grep -q "Protocol.*TLS"; then
    echo "✅ TLS IS ENABLED"
    echo "Protocol: $(echo "$OUTPUT" | grep "Protocol" | head -1)"
    echo "Cipher: $(echo "$OUTPUT" | grep "Cipher" | head -1)"
    echo "Certificate:"
    echo "$OUTPUT" | openssl x509 -noout -subject -issuer 2>/dev/null
else
    echo "❌ TLS is NOT enabled"
    echo "This is a plaintext connection"
fi
9. Check with nmap