## SmtpCommand Table: SMTP Command Event

### 1. Table Name
**SmtpCommand**

---

### 2. Table Usage
This event is generated when an SMTP (Simple Mail Transfer Protocol) command is issued on a Linux host. Attackers may use SMTP for data exfiltration, sending stolen data as email messages or attachments. Monitoring these commands can help detect such malicious activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process that issued the SMTP command. | Linux Only |
| CommandLine | The full command line of the process that issued the command. | Linux Only |
| SmtpCommandKind | The type of SMTP command issued, such as `MAIL` (4) or `DATA` (7). | Linux Only |
| CommandArg1 | The first argument of the SMTP command, which often contains sender or recipient information. | Linux Only |
| RemoteAddressIP4 | The IPv4 address of the remote mail server. | Linux Only |
| RemotePort | The port on the remote server to which the connection was made (typically 25, 465, or 587). | Linux Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "/usr/sbin/sendmail",
  "CommandLine": "sendmail -t",
  "SmtpCommandKind": "4",
  "CommandArg1": "FROM:<attacker@evil.com>",
  "RemoteAddressIP4": "198.51.100.10",
  "RemotePort": "25"
}
```
---

### 5. Example Queries
```xql
// Find all SMTP commands
event_simpleName=SmtpCommand

// Find all MAIL FROM commands, which indicate the start of an email transaction
event_simpleName=SmtpCommand SmtpCommandKind=4

// Hunt for specific commands being sent to an external mail server
event_simpleName=SmtpCommand RemoteAddressIP4!="10.0.0.0/8" RemoteAddressIP4!="172.16.0.0/12" RemoteAddressIP4!="192.168.0.0/16"

// Look for processes other than common mail clients issuing SMTP commands
event_simpleName=SmtpCommand ImageFileName!="/usr/sbin/sendmail" ImageFileName!="/usr/sbin/postfix"

// Find commands that specify a particular sender
event_simpleName=SmtpCommand CommandArg1="*<suspicious-sender@domain.com>*"
```
---