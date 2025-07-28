## SensitiveWmiQuery Table: WMI Query Event

### 1. Table Name
**SensitiveWmiQuery**

---

### 2. Table Usage
This event is generated when a client process executes a sensitive Windows Management Instrumentation (WMI) query. Attackers often use WMI to perform reconnaissance (e.g., enumerating antivirus products, listing processes), execute code, and move laterally within a network. Monitoring these queries is critical for detecting such malicious activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| WmiQuery | The WQL (WMI Query Language) query that was executed. | Windows Only |
| WmiNamespaceName | The WMI namespace that the query was targeting (e.g., `root\cimv2`). | Windows Only |
| UserName | The name of the user who executed the WMI query. | Windows Only |
| ClientComputerName | The name of the computer from which the query was initiated. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "WmiQuery": "SELECT * FROM AntiVirusProduct",
  "WmiNamespaceName": "root\\SecurityCenter2",
  "UserName": "CORP\\Admin",
  "ClientComputerName": "WORKSTATION-01"
}
```
---
### 5. Example Queries

```xql
// Find all sensitive WMI queries
event_simpleName=SensitiveWmiQuery

// Hunt for WMI queries used to enumerate antivirus products
event_simpleName=SensitiveWmiQuery WmiQuery="*AntiVirusProduct*"

// Find sensitive WMI queries targeting the default CIMv2 namespace
event_simpleName=SensitiveWmiQuery WmiNamespaceName="root\\cimv2"

// Look for queries attempting to find running processes
event_simpleName=SensitiveWmiQuery WmiQuery="*Win32_Process*"

// Investigate queries made by a specific user
event_simpleName=SensitiveWmiQuery UserName="*svc_account*"
```
---
