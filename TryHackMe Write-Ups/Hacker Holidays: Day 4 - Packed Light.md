# Hacker Holidays: Day 4 - Packed Light

## **TryHackMe Challenge**

https://tryhackme.com/room/hh-packedlight-02e5330c

## Overview

"Tiny packets. Odd hours. Suspiciously regular. Someone's smuggling out the data equivalent of a hotel towel every night, folded neatly inside traffic that looks ordinary until you decode it.

A short capture from the guest network is all VERA could pull before the connection dropped. Somewhere in that traffic, a quiet little errand is running on a loop, and it isn't part of any service the hotel actually offers."

User @0xMia says: `not me watching my laptop ping some random :8080 address every single second like clockwork 🚩 the request headers are giving 'not a real app' ngl also what is with the crypto 😭 #HackerHolidays`

#### Category: Network Forensics // PCAP Analysis // Cryptography
#### Difficulty: Easy

## Objectives

1. Analyze the provided capture for a covert communication channel.
2. Identify where the exfiltrated data is being hidden and reassemble it.
3. Decode the recovered data and submit the flag.

## Step 1: Initial PCAP triage in Wireshark

- Downloaded the pcap file `traffic.pcapng` and opened it in Wireshark.
- Observed repeated HTTP traffic to an external host on port 8080, with suspiciously regular timing (every second).
- Identified a notable packet: **Packet 19** `HTTP/1.0 200 OK (text/x-python)` from `34.41.103.191:8080` pointing to `http://byte-lotus-hotel.thm:8080/temp/updates.py`.

**This indicates a suspicious Python script being server on an unusual port.**

## Step 2: Extracting and Analyzing

- Used Wireshark's **Export Objects → HTTP** feature to extract `updates.py`
- Opened the script and review its logic:
    - The script defines a static XOR key constructed from `H0t3lSt@ff0Nly` + `K3epS3cr3t!`
    - It XORs data with this key and sends the results as a **Base64-encoded** names `hotel_sess_state`.

**This revealed the exfiltration method: data is XOR-encrypted, then Base64-encoded, and exfiltrated in HTTP cookies to an external server on port 8080.**

## Step 3: Isolate the Covert Channel Traffic

- Now, back at Wireshark, applied the filter `tcp.port==8080 && http.request && http.cookie` to isolate only relevant HTTP requests from port 8080 containing a cookie.
- To make the analysis easier, right-clicked on `Cookie: hotel_sess_state=...` and then **"Apply as column"**, to display all cookie values directly in the packet list.

**At this point, is clear that the exfiltrated data was being split into small chunks and sent repeatedly as Base64 in the `hotel_sess_state` cookie.**

## Step 4: Automating Cookie Extraction with `tshark`

- However, manually extracting each chunk of data would be inefficient, so instead I swithed to `tshark` for a more automated approach: 
`tshark -r traffic.pcapng -Y 'tcp.port == 8080 && http.request && http.cookie' -T fields -e http.cookie | sed 's/^hotel_sess_state=//'`
    - Explanation:
        - `-r traffic.pcapng` reads the capture file.
        - `-Y 'tcp.port == 8080 && http.request && http.cookie'` filters fot HTTP requests on port 8080 that contain cookies.
        - `-T fields -e http.cookie` outputs only the cookie header.
        - `| sed 's/^hotel_sess_state=//'` takes the output and strips the `hotel_sess_state=` prefix, leaving just the Base64 blobs.

**The result is a clean list of Base64 strings, one per line.**

## Step 5: Base64 decoding with CyberChef

- Copied the `tshark` output into **CyberChef**
- Applied the **From Base64** operation to each line
- Cyberchef decoded chunks of strings to our flag

**`THM{V3r4_1s_w4tch1ng_0veR_y0u}`**

## Conclusion

The **Packet Light** challenge demonstrates how a simple covert channel can hide data exfiltration in seemingly normal HTTP traffic. By combining PCAP triage in Wireshark, script analysis to identify the XOR key, and automated cookie extraction followed by Base64 decoding, the hidden flag was successfully recovered.
