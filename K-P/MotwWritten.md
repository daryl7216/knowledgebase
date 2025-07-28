## Crowdstrike Table: MotwWritten

### 1. Table Name

**MotwWritten**

---

### 2. Table Usage

This table records events related to the Mark-of-the-Web (MotW) being written to a file. This event is primarily relevant to Windows platforms and is crucial for tracking the origin of downloaded files, which can be vital for security investigations involving file provenance and potential malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| HostUrl | The host URL from a file's mark of the web. | Windows |
| ReferrerUrl | The referrer URL from a file's mark of the web. | Windows |
| TargetFileName | The resulting file name that was downloaded. | Windows |
| ZoneIdentifier | The zone identifier from a file's mark of the web. Values include: LOCAL\_MACHINE (0), INTRANET (1), TRUSTED\_SITES (2), INTERNET (3), RESTRICTED\_SITES (4). | Windows |

---

### 4. Sample Log Example

```json
{
  "HostUrl": "https://www.example.com/downloads/",
  "ReferrerUrl": "https://www.example.com/downloads/page.html",
  "TargetFileName": "C:\\Users\\user\\Downloads\\document.pdf",
  "ZoneIdentifier": 3
}
```

---

### 5. Example Queries

```xql
// Find all MotW written events
event_simpleName=MotwWritten

// Find MotW events for files downloaded from a specific host URL
event_simpleName=MotwWritten HostUrl="*malicious.com*"

// Find MotW events for files referred by a specific URL
event_simpleName=MotwWritten ReferrerUrl="*phishing.org*"

// Find MotW events for files downloaded from the internet zone
event_simpleName=MotwWritten ZoneIdentifier=3

// Find MotW events for a specific target file name
event_simpleName=MotwWritten TargetFileName="*invoice.zip"
```

---