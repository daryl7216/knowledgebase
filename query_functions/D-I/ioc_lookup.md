# ioc:lookup()

The `ioc:lookup()` function looks up Indicators of Compromise (IOCs) for IP addresses, URLs, and domains by checking them against a local copy of CrowdStrike's curated IOC database. When a match is found, it enriches the event with the associated security information.

> **Important**
> This function cannot be used to look up custom indicators in a multi-tenant environment.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **confidenceThreshold** | string | optional <br> default: `high` | The minimum confidence level of IOCs to consider. Values: `unverified`, `low`, `medium`, `high`. |
| **field** | array of strings | required | The field(s) containing the IPs, URLs, or domains to look up. The parameter name `field` can be omitted. |
| **include** | array of strings | optional <br> default: all columns | Specifies which columns from the IOC database to include in the output. Values: `indicator`, `labels`, `last_updated`, `malicious_confidence`, `published_date`, `type`. |
| **prefix** | string | optional <br> default: `ioc` | A prefix to use for the names of all the new fields added to the event. |
| **strict** | boolean | optional <br> default: `false` | If `true`, only events with an IOC match are returned. If `false`, all events are passed through, and matching events are enriched. |
| **type** | string | required | Specifies the type of IOC to search for. Must be one of `domain`, `ip_address`, or `url`. |

***

## Function Operation

When a value in one of the specified `field`(s) matches an IOC in the database, `ioc:lookup()` adds several new fields to the event.

### Returned Fields
If an IOC is found, the function adds the following fields:
* **`ioc.detected`**: A boolean field set to `true`. (The `ioc` prefix can be changed with the `prefix` parameter).
* **`ioc[]`**: An array of objects, where each object contains detailed information about a matched IOC. The fields within this object include:
    * **`indicator`**: The IOC that was found.
    * **`type`**: The type of IOC (e.g., `ip_address`).
    * **`published_date`**: When the IOC was first published (Unix timestamp).
    * **`last_updated`**: When the IOC was last updated (Unix timestamp).
    * **`malicious_confidence`**: The confidence level (`high`, `medium`, `low`, `unverified`).
    * **`labels`**: A comma-separated list of labels providing additional context.

### Labels
The `labels` field provides rich context about the IOC, broken down into categories:
* **Actors**: The threat actor associated with the IOC (e.g., `Actor/Panda`).
* **Malware Families**: The malware family associated with the indicator (e.g., `Malware/Zeus`).
* **Kill Chains**: The stage of the attack kill chain (e.g., `KillChain/C2`).
* **Domain Types**: The type of domain (e.g., `DomainType/PhishingDomain`).
* **IP Address Types**: The type of IP address (e.g., `IPAddressType/Sinkhole`).
* **Status**: The current status of the indicator (e.g., `Status/ConfirmedActive`).
* **Target**: The industry vertical targeted (e.g., `Target/Financial`).
* **Threat Type**: The type of threat (e.g., `ThreatType/Ransomware`).
* **Vulnerability**: Any associated CVEs (e.g., `Vulnerability/CVE-2012-0158`).

***

## Example

### Test an IP Address Lookup

This example shows how to test the `ioc:lookup()` function by creating a temporary event with a known sample IOC and then running the lookup on it.

* **Query Example**
    ```
    createEvents("client_ip=45.32.129.185")
    | kvParse()
    | ioc:lookup(field=[client_ip], type="ip_address", confidenceThreshold=unverified)
    ```
* **Step-by-Step**
    1.  `createEvents()`: A temporary event is created with a raw string containing a sample malicious IP address.
    2.  `kvParse()`: This parser extracts the `client_ip` field from the raw string.
    3.  `ioc:lookup()`: The function then takes the value from the `client_ip` field and looks it up in the IOC database, checking for any IP address indicators with any confidence level.

* **Result Event Data**
    The output is an event enriched with IOC information, indicating that the IP address was detected as a known Indicator of Compromise.

| ioc.detected | ioc[0].indicator | ioc[0].labels |
| :--- | :--- | :--- |
| true | 45.32.129.185 | `KillChain/C2, Malware/AsyncRAT, ...` |