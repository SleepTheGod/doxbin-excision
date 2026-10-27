# Operation Excision – Chat Layer & Administrative Identity Exposure

**Complete Intelligence Write-up**  
**Compiled by:** SleepTheGod (Taylor Christian Newsome)  
**Primary Dossier:** https://github.com/WinterGate-IC/doxbin-excision  
**Date of Issue:** 2026-10-26  
**Classification:** Public Intelligence / Law Enforcement Support  
**Document Type:** Additive Intelligence Product  

---

## 1. Executive Summary

This document consolidates all newly recovered intelligence concerning Doxbin’s real-time chat subsystem, its primary administrative identity, authentication model, and related research tooling developed under the SleepTheGod / WinterGate-IC effort.

A dedicated chat monitoring application (DoxbinChat Snooper) was recovered and analyzed. The tool establishes a persistent, authenticated WebSocket connection to the live Doxbin platform and continuously streams chat traffic. Most significantly, the application hardcodes the platform’s primary administrative identity and presents itself as that administrator upon connection.

**Key confirmed findings:**

- Primary administrative account:  
  - User ID: `1`  
  - Username: `kt`  
  - Email: `kt@doxbin.com`
- Real-time chat gateway: `https://doxbin.com` with Socket.IO path `/gateway/`
- Authentication depends solely on Cloudflare `cf_clearance` + application `session` cookies
- The recovered tool was purpose-built for long-term, unattended surveillance of the chat channel

This material is strictly additive to the existing Operation Excision dossier and does not replace any prior Phase 1, infrastructure, or national security findings.

---

## 2. Confirmed Administrative Identity

The following identity is hardcoded as a constant inside the recovered application:

| Field     | Value              |
|-----------|--------------------|
| User ID   | `1`                |
| Username  | `kt`               |
| Email     | `kt@doxbin.com`    |
| Domain    | `doxbin.com`       |

Upon successful connection the tool injects the following local system message:

> Administrator kt connected – kt@doxbin.com on doxbin.com

This is not a reconstructed or guessed identity. It is an explicit developer-supplied constant treated by the tool as the authoritative administrative account.

**Assessment:** User ID 1 / username `kt` currently represents the highest-value open-source attribution target for the active Doxbin operator set. Any historical pastes, administrative actions, or public statements linked to this identity can be treated with high confidence as operator activity.

---

## 3. Technical Findings – Chat Gateway

### 3.1 Connection Parameters

- **Endpoint:** `https://doxbin.com`
- **Socket.IO path:** `/gateway/`
- **Transport:** WebSocket only (`transports=["websocket"]`)
- **Required headers:**
  - `Cookie: cf_clearance=<value>; session=<value>`
  - `Origin: https://doxbin.com`
  - `Connection: keep-alive, Upgrade`
  - Modern Firefox User-Agent string

### 3.2 Authentication Model

Access to the live chat stream is gated exclusively by two cookies:

1. Cloudflare clearance cookie (`cf_clearance`)
2. Application session cookie (`session`)

The recovered tool stores these values in a local `cookies.json` file and implements:

- Continuous monitoring of the cookie file for changes
- Automatic disconnection and forced reconnection when cookies are updated
- An administrative web panel that allows an operator to update the cookies without restarting the process

**Implication:** Possession of a valid clearance + session cookie pair is sufficient for persistent, authenticated access to the platform’s real-time communication channel. No additional multi-factor or secondary authentication mechanism is required once the cookies are obtained.

### 3.3 Observed Socket.IO Events

| Event                | Direction             | Function                                      |
|----------------------|-----------------------|-----------------------------------------------|
| `send_message`       | Server → Client       | Delivers new chat messages                    |
| `online`             | Server → Client       | Reports current online user count             |
| connect / disconnect | Bidirectional         | Session lifecycle management                  |

### 3.4 Message Object Structure (Observed)

```json
{
  "createdAt": "HH:MM UTC",
  "displayName": "string",
  "content": "string (HTML stripped client-side)",
  "rankData": {
    "suffix": "[System] | [Admin] | ...",
    "rankColor": "red | green | ...",
    "rankStyle": "bold"
  }
}
```

The tool maintains a rolling history capped at the 100 most recent messages and tracks unread counts and online user totals.

---

## 4. Recovered Tool Capabilities

The DoxbinChat Snooper is a full Flask application with the following features:

- Real-time chat message interception via WebSocket
- Online user counter
- Admin auto-login simulation using the hardcoded `kt` identity
- Cookie hot-reload and automatic reconnection
- Embeddable widget intended for static HTML sites
- Administrative panel for live cookie management
- Message history with user profiles and ranks
- Connection status monitoring and system message injection

**Stack:** Flask + Waitress + python-socketio (asyncio client) + eventlet + aiohttp + flask-cors

The design of the tool (persistent connection, cookie monitoring loop, auto-reconnect, long-lived session) indicates it was built for continuous, unattended surveillance rather than opportunistic observation.

---

## 5. Operational Significance

### 5.1 Attribution Value

The explicit binding of User ID 1 to the username `kt` and the email address `kt@doxbin.com` provides a concrete, high-confidence anchor for historical correlation. This identity should be treated as the primary open-source attribution target for the current operator set.

### 5.2 Infrastructure Mapping

The previously undocumented `/gateway/` Socket.IO endpoint expands the known communication topology of the platform. When combined with existing endpoint mapping work, it produces a clearer picture of both the public web application surface and the real-time subsystem.

### 5.3 Operational Security Failure

Hardcoding the primary administrative identity inside a surveillance tool constitutes a significant OPSEC failure. It demonstrates that at least one party with deep knowledge of the platform’s internal structure treated the admin account as a static, reusable constant rather than a protected secret.

### 5.4 Persistence & Surveillance Capability

The architecture of the recovered tool underscores the value of the chat channel as an intelligence source. The same design choices that make the tool effective for monitoring also illustrate how accessible the channel is to any party that can obtain valid session cookies.

---

## 6. Relationship to Existing Excision Findings

This write-up is strictly additive. It does not replace or supersede any prior material in the Operation Excision dossier, including:

- Network mapping and origin server documentation
- Vulnerability assessments (reflected/stored XSS, API failures, static asset failures)
- Network-layer observations (state table exhaustion, RST storms, latency anomalies)
- Links between the platform and violent extremist groups (Atomwaffen Division, Insanity Security Team)
- National security threat assessments

The chat-layer findings strengthen the overall assessment that Doxbin functions as a persistent criminal coordination platform with identifiable administrative accounts and weak internal controls.

---

## 7. Related Research Tooling (SleepTheGod)

The following repositories form the broader technical research baseline supporting this and prior Excision products:

| Repository | Purpose | Link |
|------------|---------|------|
| **WinterGate-IC/doxbin-excision** | Primary public intelligence dossier | https://github.com/WinterGate-IC/doxbin-excision |
| **DoxbinEndpointMapper** | Endpoint surface mapping via Chromium console `fetch` | https://github.com/SleepTheGod/DoxbinEndpointMapper |
| **DoxbinTypeScript** | TypeScript tooling / client experiments | https://github.com/SleepTheGod/DoxbinTypeScript |
| **DoxbinPosting** | Reverse-engineered posting content / workflow | https://github.com/SleepTheGod/DoxbinPosting |
| **Doxbin-s-Shitty-Bontet-Scanner** | Analysis of Doxbin’s malware / “botnet” detection logic | https://github.com/SleepTheGod/Doxbin-s-Shitty-Bontet-Scanner |
| **doxbin.org** | Historical notes, nmap output, and security observations | https://github.com/SleepTheGod/doxbin.org |
| **DoxBinFlask** | Educational open-source Flask clone | https://github.com/SleepTheGod/DoxBinFlask |
| **DoxBin** | Educational PHP remake (explicitly not original source) | https://github.com/SleepTheGod/DoxBin |

---

## 8. Recommended Lawful Follow-up Actions

1. Correlate all historical platform activity under username `kt` or User ID `1`.
2. Cross-reference the email address `kt@doxbin.com` against available breach, registration, and public records datasets.
3. Preserve and document the `/gateway/` endpoint and associated Socket.IO event schema for network signature development.
4. Treat the recovered snooper application and its hardcoded administrative identity as evidence of both capability and OPSEC failure by parties with privileged knowledge of the platform.
5. Integrate this write-up into the canonical WinterGate-IC/doxbin-excision repository as an additive intelligence product.

---

## 9. Machine-Readable Summary

```json
{
  "source": "doxbin-chat / DoxbinChat Snooper",
  "extracted": "2026-10-26",
  "compiled_by": "SleepTheGod",
  "primary_dossier": "https://github.com/WinterGate-IC/doxbin-excision",
  "admin_identity": {
    "userid": 1,
    "username": "kt",
    "email": "kt@doxbin.com",
    "domain": "doxbin.com",
    "notes": "Hardcoded as primary administrator. System message injected on connect confirms live use of this identity."
  },
  "infrastructure": {
    "chat_endpoint": "https://doxbin.com",
    "socketio_path": "/gateway/",
    "transports": ["websocket"],
    "auth_cookies": ["cf_clearance", "session"],
    "origin_required": "https://doxbin.com"
  },
  "tool_capabilities": [
    "Real-time message interception",
    "Online user count tracking",
    "Admin auto-login simulation",
    "Cookie hot-reload and auto-reconnect",
    "Embeddable widget",
    "Admin panel for cookie management"
  ],
  "risk_assessment": {
    "opsec_failure": "High – primary admin identity and email hardcoded",
    "surveillance_utility": "High – persistent authenticated chat access",
    "attribution_value": "High – UID 1 / kt@doxbin.com"
  }
}
```

---

## 10. Disclaimer

This document is produced solely for lawful intelligence, defensive research, and public awareness purposes. It does not constitute an attack guide, exploit code, or authorization for unauthorized access of any kind. All material described was derived from publicly available sources and recovered tooling.

Unauthorized use of any information contained herein for harassment, illegal access, or criminal activity is strictly prohibited.

---

**Compiled by:** SleepTheGod  
**GitHub:** https://github.com/SleepTheGod  
**Primary Dossier:** https://github.com/WinterGate-IC/doxbin-excision  
**Date of Issue:** 2026-10-26  

*End of complete write-up.*
