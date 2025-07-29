## JavaClassFileWritten Table: File Written Event

### 1. Table Name
**JavaClassFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Java Class file (`.class`). Attackers may drop malicious Java classes on a system as part of an exploit chain or to execute malicious code using the Java Virtual Machine (JVM). Monitoring the creation of these files is useful for investigating Java-based threats and malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the path of the .class file). | All |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\AppData\\Local\\Temp\\Exploit.class",
  "ContextBaseFileName": "Exploit.class"
}
```


---

### 5. Example Queries
```xql
// Find all Java Class files written to disk
event_simpleName=JavaClassFileWritten TargetFileName="*.class"

// Hunt for Java Class files being written to suspicious locations like Temp or user profile directories
event_simpleName=JavaClassFileWritten TargetFileName IN ("*\\Temp\\*", "*\\AppData\\*")

// Find Java Class files with a potentially malicious name
event_simpleName=JavaClassFileWritten ContextBaseFileName="*Exploit*" OR ContextBaseFileName="*Shell*"
```
---
