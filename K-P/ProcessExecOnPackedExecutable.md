## ProcessExecOnPackedExecutable Table: Packed Executable Execution Event

### 1. Table Name
**ProcessExecOnPackedExecutable**

---

### 2. Table Usage
This event is generated when a PE (Portable Executable) file that is known to be packed is executed on a Windows host. Packers are often used by malware authors to obfuscate their code, making it harder for antivirus software and security analysts to examine. Monitoring the execution of packed files is a critical technique for identifying potentially malicious software that is attempting to evade static analysis.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FileSubType | A numeric code indicating the type of packer detected. Common values include: `1` (ASPack), `2` (MPRESS), `3` (Themida), `4` (UPX), `5` (VMProtect). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "FileSubType": "4"
}
```
---
### 5. Example Queries
```xql
// Find all executions of packed executables
event_simpleName=ProcessExecOnPackedExecutable

// Hunt for the execution of UPX-packed files, a very common packer
event_simpleName=ProcessExecOnPackedExecutable FileSubType=4

// Search for executables packed with Themida or VMProtect, which are often used in advanced malware
event_simpleName=ProcessExecOnPackedExecutable FileSubType IN (3, 5)
```
---