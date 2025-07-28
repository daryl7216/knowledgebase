## PackedExecutableWritten Table: Packed Executable Written Event

### 1. Table Name
**PackedExecutableWritten**

---

### 2. Table Usage
This event indicates that a packed executable was written to disk on a Windows host. Packers are often used by malware authors to obfuscate their code, making it harder for antivirus software and security analysts to examine. Monitoring the creation of packed files is a critical technique for identifying potentially malicious software that is attempting to evade static analysis.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path to the packed executable file that was written to disk. | Windows Only |
| FileSubType | A numeric code indicating the type of packer detected. Common values include: `1` (ASPack), `2` (MPRESS), `3` (Themida), `4` (UPX), `5` (VMProtect). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\packed_malware.exe",
  "FileSubType": "4"
}
```

---

### 5. Example Queries
```xql
// Find all events where a packed executable was written to disk
event_simpleName=PackedExecutableWritten

// Hunt for UPX-packed files being written, as this is a very common packer
event_simpleName=PackedExecutableWritten FileSubType=4

// Search for executables written to disk that were packed with Themida or VMProtect
event_simpleName=PackedExecutableWritten FileSubType IN (3, 5)

// Find packed executables written to a specific directory
event_simpleName=PackedExecutableWritten TargetFileName="*\\Downloads\\*"
```
---