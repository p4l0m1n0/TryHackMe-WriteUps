# The Greenholt Phish

**TryHackMe Challenge**

https://tryhackme.com/room/phishingemails5fgjlzxc

## Overview

A sales executive at Greenholt PLC has reported a suspicious email received from a known customer. The message raised several red flags: a generic greeting, an unexpected request for a money transfer, and an unsolicited attachment. According to the employee, this behavior does not align with the customer's usual communication style. Concerned that the email may be malicious, the message has been escalated to the SOC (Security Operations Center) for further investigation. Your goal is to analyze the provided email sample and determine whether it is legitimate or part of a phishing attempt.

## Objectives

1. Analyze the provided email to identify and extract key artifacts
2. Investigate the message source to determine its origin and authenticity
3. Use analysis tools to assess the potential maliciousness of the email

## Investigation Steps

### Step 1: Extract Transfer Reference Number

There's an email file on Desktop called 'challenge.eml' which is the email for analysis.
The first question of the challenge is to identify the Transfer Reference Number listed in the email's Subject line.
So first things first; just by analyzing the email's Subject "webmaster@redacted.org your: Transfer Reference Number:(09674321)", we can clearly find the answer to the first question (09674321).

**Answer:** `09674321`

![Email Subject Line/Transfer Reference Number](screenshots/subject.png)

### Step 2: Identify Sender's Display Name

The second question to identify the Display Name of the sender, and just by analyzing the email header we find that the sender's Display Name is "Mr. James Jackson".

**Answer:** `Mr. James Jackson`

![Sender's Display Name](screenshots/sender_name.png)

### Step 3: Identify Sender's Email Address

I should continue investigating the email headers to identify the sender's email address, which in this case is info@mutawamarine.com
[same screenshot as 2nd step]

**Answer:** `info@mutawamarine.com`

![Sender email in header (same as Step 2)](screenshots/sender_name.png)

### Step 4: Identify Reply Email Address

By analyzing the email header, the "Reply Email" is identified. Note that it is similar but different from the sender's actual email.

**Answer:** `info.mutawamarine@mail.com`

![Reply Email Address](screenshots/replyto.png)

### Step 5: Identify Originating IP Address

Fifth question is to identify the originating IP address of this email.
So, here we have to analyze the "Message Source" of the email to be able to find the originating IP;
After a bit of analysis and reading thought the Message Source, I was able to find the section containing the actual sender's email and IP address:
```
Received: from hwsrv-737338.hostwindsdns.com ([192.119.71.157]:51810 helo=mutawamarine.com)
```

**Answer:** `192.119.71.157`

![Originating IP address](screenshots/ip.png)

### Step 6: Determine IP Owner

I should continue investigating the IP address from the previous question, but now find out who is the owner of the originating IP.
First I tried perfomming both WHOIS/dig lookup on terminal but had no success or any interesting response, so decided to try online IP lookup tools, such as whatismyipaddress.com, and was able to determine the IP owner's name as "HostPapa"

**Answer:** `HostPapa`

![CMMD Fail](screenshots/dig-whois.png)
![IP Owner](screenshots/IPowner.png)

### Step 7: Perform SPF Record Check

Now, I need to run a SPF record check on the Return-Path domain identified in the email headers.
By running the command:
```bash
dig +short TXT mutawamarine.com
```
I'm now able to fetch the TXT records for the Return-Path domain (mutawamarine.com) and read the full SPF string from the output, which is:
```bash
'v=spf1 include:spf.protection.outlook.com -all'
```

The SPF record for the Return-Path domain (mutawamarine.com) is retrieved.

**Answer:** `v=spf1 include:spf.protection.outlook.com -all`

![SPF Check](screenshots/spf.png)

### Step 8: Perform DMARC Lookup

Now, to perform a DMARC lookup for the Return-Path domain found in the email headers, I prefered using the online DMARC checker tool 'dmarcian.com/domain-checker/' to check on the domain (mutawamarine.com), which gave me the answer I needed:
```bash
"v=DMARC1; p=quarantine; fo=1"
```

**Answer:** `v=DMARC1; p=quarantine; fo=1`

![DMARC Lookup](screenshots/dmarc.png)

### Step 9: Identify Attachment File Name

Now I have to identify the file name of the attachment found in the email.
So, by analyzing the Message Source a little deeper, I was able to found the real attachment name by the Content-Disposition function:

```
Content-Disposition: attachment; filename="SWT_#09674321____PDF__.CAB"
```

**Answer:** `SWT_#09674321____PDF__.CAB`

![Attachment File Name](screenshots/attachment.png)

### Step 10: Calculate SHA256 Hash

For the tenth question, I need to find out the SHA256 hash of the email attachment file; so by downloading the attachment into my virtual environment and simply running:

```bash
sha256sum 'SWT_#09674321____PDF__.CAB'
```
The terminal gave me the hash and the answer for the question:

**Answer:** `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f`

![SHA256](screenshots/sha.png)

### Step 11: Determine File Size

Now, I need to investigate the file hash from the previous question using VirusTotal, and determine the file size in KB (e.g., 122.31 KB).
By examining the Details tab within Virus Total, I was able to identify the file has a size of 400.26 KB.
**Answer:** `400.26 KB`

![VirusTotal](screenshots/virustotal.png)

### Step 12: Identify Actual File Type

For the 12th and last step, Im asked to continue the research on the file, and identify the actual file type of the attachment.
So, if we keep analyzing the Virus Total Details page, we notice its a .RAR file being spoofed as a PDF file.

**Answer:** `.RAR` file (spoofed as PDF)

![File Type](screenshots/filetype.png)

## Summary

This lab reinforced that phishing analysis is less about the subject line and more about validating the full message context: headers, sender identity, reply-to behavior, delivery path, and attachment intelligence all have to line up before an email can be trusted. It also showed how small inconsistencies, such as a display-name mismatch, a suspicious reply address, and a disguised attachment, can reveal a malicious message even when the email appears routine at first glance.

## What I learned

- Email headers are the most reliable starting point for tracing the true source of a message, especially when the visible sender name is spoofed.

- Authentication checks such as SPF and DMARC help determine whether the sending domain is configured to authorize the message path, but they should be reviewed alongside the rest of the header data rather than in isolation.

- Suspicious attachments should be handled as indicators of compromise, with the filename, hash, file size, and actual file type all validated before any further action.

- Threat intelligence tools such as VirusTotal add useful context by confirming whether a file is benign, malicious, or masquerading as a different format.

- A structured workflow makes phishing triage faster and more accurate: extract artifacts, verify sender infrastructure, inspect authentication results, hash attachments, and compare findings against external intelligence sources.
