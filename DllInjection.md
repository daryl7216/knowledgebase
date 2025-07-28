## DllInjection Table: DLL Injection Event

### 1. Table Name
**DllInjection**

---

### 2. Table Usage
This table records events where DLL injection occurred in a target process. This is useful for investigations into malicious code injection, privilege escalation, and evasion techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| InjectedDll | The name or path of the DLL that was injected. | Windows Only |
| InjectedThreadFlag | Flags indicating characteristics of the injected thread. Values: NONE (0x00000000) PREV_MODE_KERNEL (0x00000001) START_ADDRESS_IN_NAMED_PE (0x00000002) START_ADDRESS_IN_SYS_RANGE (0x00000004) SYSTEM_THREAD (0x00000008) SOURCE_PROCESS_IS_SYSTEM (0x00000010) TARGET_ANALYSIS_FAILED (0x00000020) CONTEXT_ANALYSIS_FAILED (0x00000040) START_ADDRESS_IN_NTDLL (0x00000080) START_ADDRESS_IN_WIN_DLL (0x00000100) START_ADDRESS_PRIVATE_MEM (0x00000200) | Windows Only |
| TargetProcessId | The unique ID of the target process where the DLL injection occurred. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 1234,
  "ContextThreadId": 5678,
  "TreeId": "abc-123",
  "TargetProcessId": 9876,
  "TargetThreadId": 5432,
  "RawProcessId": 9876,
  "RawThreadId": 5432,
  "ThreadStartContext": "0x12345678",
  "ThreadStartAddress": "0xABCDEF01",
  "InjectedThreadFlag": 256,
  "InjectedDll": "C:\\Windows\\temp\\malicious.dll"
}
```
---

### 5. Example Queries
```xql
// Find all DLL injection events
event_simpleName=DllInjection

// Find DLL injection events for a specific target process ID
event_simpleName=DllInjection TargetProcessId=9876

// Find DLL injection events where a specific DLL was injected
event_simpleName=DllInjection InjectedDll="*malicious.dll*"

// Find DLL injection events with a specific injected thread flag (e.g., START_ADDRESS_IN_WIN_DLL)
event_simpleName=DllInjection InjectedThreadFlag=256
```
---