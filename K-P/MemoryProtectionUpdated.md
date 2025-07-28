````markdown
## Crowdstrike Table: MemoryProtectionUpdated

### 1. Table Name
**MemoryProtectionUpdated**

---

### 2. Table Usage
This table triggers if a new executable memory region is mapped which is writable or mapped from a file. It is useful for detecting potential code injection or memory manipulation techniques.

---

### 3. Important Parameter Description

| Parameter             | Description                                                                                          | Platforms Affected |
|-----------------------|------------------------------------------------------------------------------------------------------|--------------------|
| MemoryProtectionFlags | Values indicating the memory protection applied to the region: PROT_NONE (0x00000000), PROT_READ (0x00000001), PROT_WRITE (0x00000002), PROT_EXEC (0x00000004). | Linux, Falcon Container |
| Length                | The length of the memory region.                                                                     | Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "MemoryProtectionFlags": 4,
  "Length": 4096
}
````

---

### 5. Example Queries

```xql
// Find all MemoryProtectionUpdated events where the memory region is executable and writable
event_simpleName=MemoryProtectionUpdated MemoryProtectionFlags=(2 OR 4)

// Find MemoryProtectionUpdated events with a specific length
event_simpleName=MemoryProtectionUpdated Length=4096
```

---

```