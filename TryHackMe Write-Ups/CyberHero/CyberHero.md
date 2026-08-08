# CyberHeroes

**TryHackMe Challenge**

https://tryhackme.com/room/cyberheroes

## Overview

This room focuses on authentication bypass. The goal is to identify and exploit a vulnerable login mechanism to obtain the flag.

## Objective

Use the AttackBox to find valid credentials and log in to the target application at `http://TARGET_IP`.

**Category: `Authentication Bypass`**

## Reconnaissance

### 1. Port and service enumeration

I used `nmap` to discover open ports and services on the target:

```
nmap -sC -sV -p- TARGET_IP
```

**Findings:**

- `22/tcp` open ssh — OpenSSH 8.2p1 Ubuntu 4ubuntu0.4
- `80/tcp` open http — Apache httpd 2.4.48

![nmap](nmappng.png)

### 2. Directory enumeration

I scanned the web server for hidden directories using `gobuster`:

```
gobuster dir -u http://10.64.156.118 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

**Findings:**

- Discovered the `/assets` directory.

![gobuster](gobuster.png)

## Application analysis

### 3. Inspecting the login page

I returned to `http://TARGET_IP`, navigated to the login page, and examined the page source for hidden information or client-side logic.

![pagesource](pagesource.png)

**Findings:**

The page source contained a hard-coded username and an obfuscated password comparison:

```javascript
if (a.value=="h3ck3rBoi" & b.value==RevereString"(54321@terceSrepuS"))
```

![sourceflag](sourceflag.png)

## Exploitation

### 4. Decoding the password

The password was reversed in the source code. I decoded it with:

```
echo "54321@terceSrepuS" | rev
```

**Decoded password:**

```
SuperSecret@12345
```

![decoded](decoded.png)

### 5. Logging in and capturing the flag

Using the discovered credentials, I logged in successfully and retrieved the flag:

```
flag{edb0be532c540b1a150c3a7e85d2466e}
```

![flag](flag.png)

## Summary

This exercise demonstrated a simple client-side authentication bypass through credential disclosure in page source. The target application exposed the login logic, allowing the username and reversed password string to be recovered without any server-side exploitation.

## Lessons learned

- Always inspect client-side code for hidden logic or hard-coded values.
- Sensitive authentication checks should never rely on client-side validation.
- Directory enumeration can reveal additional resources that support analysis of the application.
- Simple obfuscation is not a substitute for secure authentication design.
