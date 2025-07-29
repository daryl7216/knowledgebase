## JarFileWritten Table: File Written Event

### 1. Table Name
**JarFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Java Archive (.jar) file to disk. Attackers often use malicious JAR files to exploit vulnerabilities (such as Log4j) or to deploy cross-platform malware. Monitoring the creation of these files is crucial for investigating Java-based threats and potential exploitation attempts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the full path of the .jar file). | All |
| ContextBaseFileName | The base name of the context file (the name of the .jar file itself). | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\Public\\Downloads\\log4j-exploit.jar",
  "ContextBaseFileName": "log4j-exploit.jar"
}
```
---

### 5. Example Queries
```xql
// Find all JAR files written to disk
event_simpleName=JarFileWritten TargetFileName="*.jar"

// Hunt for JAR files being written to suspicious locations like Temp or Public directories
event_simpleName=JarFileWritten TargetFileName IN ("*\\Temp\\*", "*\\Users\\Public\\*", "/tmp/*", "/var/tmp/*")

// Find JAR files with names that could indicate an exploit, such as for Log4j
event_simpleName=JarFileWritten ContextBaseFileName="*log4j*" OR ContextBaseFileName="*exploit*"
```
---