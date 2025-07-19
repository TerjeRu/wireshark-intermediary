# █▓▒▒░░░ WIRESHARK: A PRACTICAL ANALYSIS MANUAL ░░░▒▒▓█

> Network traffic is the definitive record of all activity on a network. While applications provide an abstraction, packet analysis allows for direct observation of the underlying protocols and data. This guide provides a series of practical, hands-on exercises for learning Wireshark, from initial setup to advanced analysis techniques.
>
> Launch your Kali Linux virtual machine and prepare for direct data inspection.

---

## 零 – INSTALLATION & SETUP

> Before collecting packets, ensure Wireshark is installed, updated, and you have proper privileges. No root-level sniffing as a habit—use group permissions or elevate only when necessary.

1. **On Linux (Fedora/KDE & Kali)**

   ```bash
   sudo dnf install wireshark-qt    # Fedora
   sudo apt update && sudo apt install wireshark    # Kali/Ubuntu
   sudo usermod -aG wireshark $USER
   # Log out and back in to apply group changes
   ```
2. **On Windows & macOS**

   * Download from [https://www.wireshark.org/download.html](https://www.wireshark.org/download.html)
   * Run installer; approve Npcap/WinPcap (Windows) or PacketWriter (macOS) when prompted.
3. **Permissions & Best Practices**

   * **Non-root captures**: Use the `wireshark` group on Linux.
   * **Sudo only when required**: `sudo wireshark` or `sudo dumpcap`.
   * **Keep Wireshark updated**: New protocols and bug fixes are released frequently.

---

## 壱 – THE ENVIRONMENT: INTERFACE & FIRST CAPTURE

> The NIC is your sensor. Activate promiscuous or monitor mode as needed to see everything on the wire—or the air.

### ► **LIVE EXERCISE: Tapping the Stream**

1. **Elevate Privileges:**

   ```bash
   sudo wireshark
   ```
2. **Select Interface:**

   * `eth0` = wired, `wlan0` = wireless, `lo` = loopback.
   * An active graph beside the interface indicates traffic.
   * **Double‑click** to start.
3. **Generate Traffic:**

   * Browse a non-HTTPS site: `http://httpforever.com/`.
   * Watch packet count climb in real time.
4. **Stop Capture:**

   * Click the red square (◼️).
   * Save capture as `.pcapng` for modern compatibility.

---

## 弐 – THE TOOL: DECODING THE INTERFACE

> Wireshark’s UI is three linked panes: list, details, bytes. Mastery of their interaction is non-negotiable.

```
+--------------------------------------------------+
| Pane 1: Packet List (Summary View)               |
+--------------------------------------------------+
| Pane 2: Packet Details (Protocol Tree View)      |
+--------------------------------------------------+
| Pane 3: Packet Bytes (Raw Hexadecimal View)      |
+--------------------------------------------------+
```

### ► **LIVE EXERCISE: Trisecting Reality**

1. **Packet List:**

   * Columns: `No.`, `Time`, `Source`, `Destination`, `Protocol`, `Length`, `Info`.
   * **Click** any row to populate Panes 2 & 3.
2. **Packet Details:**

   * Shows OSI layers.
   * **Expand** Ethernet ▶ IP ▶ TCP ▶ HTTP for an HTTP packet.
3. **Packet Bytes:**

   * Raw hex + ASCII.
   * **Click** a field in Pane 2 to highlight its bytes here.

---

## 参 – DISPLAY FILTERS: ISOLATING SIGNALS

> Display filters refine your view—never alter the capture file, just the presentation.

### ► **LIVE EXERCISE: Applying Lenses**

*Valid syntax turns the bar green; invalid, red.*

1. **By IP Address:**

   * `ip.addr == 8.8.8.8`
   * `ip.src == 192.168.1.100`
   * `ip.dst != 192.168.1.1`
2. **By Protocol:**

   * `tcp`, `udp`, `icmp`, `dns`, `http`
3. **By Port:**

   * `tcp.port == 443`
   * `tcp.dstport == 80`
4. **Logical Ops:**

   * `ip.addr==192.168.1.50 && tcp.port==445`
   * `http || dns`
   * `!(arp || icmp)`
5. **Content:**

   * `http contains "password"`
   * `tcp contains "USER"`

---

## 四 – CAPTURE FILTERS: PRE-EMPTIVE DATA REDUCTION

> Capture filters shrink your dataset in real time. Use BPF syntax before you click 'Start'.

### ► **LIVE EXERCISE: Recording with Intent**

1. **Locate:**

   * On the welcome screen, enter filter in **Capture Filter** field.
2. **Examples:**

   ```
   host 8.8.8.8        # only traffic to/from host
   port 53            # only DNS
   net 192.168.1.0/24 # local subnet
   port not 22        # exclude SSH
   ```
3. **Test:**

   * Set `host httpforever.com`.
   * Ping `8.8.8.8`; no ICMP appears.
   * Browse `httpforever.com`; only that HTTP traffic is captured.

---

## 五 – STREAM RECONSTRUCTION: FOLLOWING CONVERSATIONS

> Reassembly converts fragments into coherent dialogue. Stream-Follow is your friend.

### ► **LIVE EXERCISE: Rebuilding a TCP Conversation**

1. **Capture FTP Session:**

   ```bash
   curl ftp://test.rebex.net/ --user demo:password
   ```
2. **Filter:**

   * `ftp`
3. **Follow:**

   * Right-click `Request: USER demo` ➔ **Follow** ▶ **TCP Stream**
4. **Inspect:**

   * Commands in red, responses in blue.
   * Credentials and server replies laid bare.

---

## 六 – THE HUNT: ACTIVE ANALYSIS PATTERNS

> Identify signatures of attacks or leaks—transform passive viewing into proactive detection.

### ► **EXERCISE 1: Port Scan Signature**

1. **Scan:**

   ```bash
   nmap -sT localhost
   ```
2. **Capture on `lo`:**

   * Multiple SYNs to various ports.
3. **Filter:**

   * `tcp.flags.syn==1 && tcp.flags.ack==0`
4. **Result:**

   * Rapid SYN packet flood—a TCP connect scan.

### ► **EXERCISE 2: Cleartext Credentials**

1. **Navigate:**

   * `http://testphp.vulnweb.com/login.php`
2. **Login:**

   * `test` / `test`
3. **Filter:**

   * `http.request.method=="POST"`
4. **Inspect:**

   * Expand `HTML Form URL Encoded` layer; see credentials in plain text.

---

## 七 – WLAN ANALYSIS: DECONSTRUCTING WIRELESS TRAFFIC

> Wireless requires monitor mode. You’re not on the network—you’re in the air.

### ► **LIVE EXERCISE: Capturing WPA2 Handshake**

1. **Monitor Mode:**

   ```bash
   sudo airmon-ng start wlan0
   # new interface: wlan0mon
   ```
2. **Capture:**

   * Select `wlan0mon`.
3. **Trigger:**

   * Connect a device to Wi-Fi.
4. **Filter:**

   * `eapol`
5. **Result:**

   * Four EAPOL frames—the WPA2 handshake.
   * Stop with `sudo airmon-ng stop wlan0mon`.

---

## 八 – COMMAND-LINE ANALYSIS: `tshark`

> For headless or batch analysis, `tshark` rules.

### ► **LIVE EXERCISE: Automated Field Extraction**

1. **Capture:**

   ```bash
   tshark -i eth0 -a duration:30 -w /tmp/webtraffic.pcap
   ```
2. **Extract:**

   ```bash
   tshark -r /tmp/webtraffic.pcap -T fields -e ip.src -e http.host -Y "http.host"
   ```
3. **Output:**

   * Two-column list: source IP ↔ HTTP host.

---

## 九 – STATISTICAL ANALYSIS & VISUALIZATION

> Wireshark’s stats tools turn raw captures into graphs and tables.

### ► **LIVE EXERCISE: Building I/O Graphs**

1. **Open I/O Graph:**

   * **Statistics** ▶ **I/O Graph**
2. **Configure:**

   * Graph 1: All packets/tick (default).
   * Graph 2 Filter: `tcp.flags.syn==1`.
   * Y-axis unit: Packets/Tick or Bits/Tick.
3. **Interpret:**

   * Spikes in SYN graph may indicate SYN floods or scans.

---

> True proficiency comes from continuous, curious application. The network never sleeps, and neither should your curiosity.
>
> **// END OF LINE //**
