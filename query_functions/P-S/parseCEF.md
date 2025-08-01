# parseCEF()

The `parseCEF()` function parses Common Event Format (CEF) encoded messages. Only CEF version 0 is supported. This function will skip any prefix up to the marker `CEF:0`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | optional \<br\> default: `@rawstring` | Field that holds the input in CEF form. This parameter specifies which field should be parsed. The default value parses the `rawstring`. |
| `headerprefix` | string | optional \<br\> default: `cef.` | Prefix to the field names for the header fields. |
| `keeplabels` | boolean | optional \<br\> default: `false` | Removes fields ending with `label` along with any field that has the same name, for example, `cef.ext.csLabel` and `cef.ext.cs`. |
| `labelprefix` | string | optional \<br\> default: `cef.label.` | Prefix to the field names for the label fields. |
| `prefix` | string | optional \<br\> default: `cef.ext.` | Prefix to extension fields. Fields in the CEF extension part are prefixed with this. |

***

## Function Operation

The `parseCEF()` function is designed to interpret and extract data from logs formatted in the Common Event Format (CEF) version 0. It can process the content of a specified field (by default, `@rawstring`) and will automatically skip any preceding text until it encounters the `CEF:0` marker.

This function allows for customization of prefixes added to the parsed field names:

  * `headerprefix`: Controls the prefix for standard CEF header fields (default is `cef.`).
  * `labelprefix`: Controls the prefix for label fields (default is `cef.label.`).
  * `prefix`: Controls the prefix for extension fields (default is `cef.ext.`).

The `keeplabels` parameter (default `false`) dictates whether fields ending with `label` and their corresponding non-label counterparts are removed.

For compatibility with legacy systems, this implementation allows the tab character (ascii 0x09) in addition to space (ascii 0x20) as a separator for key-value pairs in the extensions section. Literal backslash followed by 't' (as in `\t`) is not a separator, but re-interpreted like `\n` and `\r` in the specification.

***

## Examples

### CEF parse the @rawstring field to show how the CEF format is parsed with the default field names

This example demonstrates parsing a CEF log line using default settings.

  * **Query Example**

    ```
    parseCEF(field=@rawstring)
    ```

  * **Input Event Data**
    (From a log line like this:)
    `Sep 19 08:26:10 host CEF:0 | security threatmanager | 1.0| 100| detected a \\ in packet 10 src=10.0.0.1 act blocked a \\ dst=1.1.1.1`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| cef.version | 0 |
| cef.device.vendor | security |
| cef.device.product | threatmanager |
| cef.device.version | 1.0 |
| cef.event\_class\_id | 100 |
| cef.name | "detected a \\ in packet" |
| cef.severity | 10 |
| cef.ext.src | 10.0.0.1 |
| cef.ext.act | "blocked a " |
| cef.ext.dst | 1.1.1.1 |

---

### Show the output of parseCEF() with changed header prefix

This example demonstrates how to change the prefix for header fields.

  * **Query Example**

    ```
    parseCEF(headerprefix="new.header.")
    ```

  * **Input Event Data**
    (From a log line like this:)
    `CEF: 0 Incapsula | SIEMintegration|1|1|Illegal Resource Access |3| fileid=3412341160002518171 sourceServiceName=site123.abcd.info siteid=1509732 suid=50005477 requestClientApplication Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0 deviceFacility=mia cs2=true cs2Label-Javascript Support start=14532901213`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| new.header.device.version | 1 |
| new.header.device.vendor | Incapsula |
| new.header.event\_class\_id | 1 |
| new.header.device.product | SIEMintegration |
| new.header.name | Illegal Resource Access |
| new.header.version | 0 |
| new.header.severity | 3 |
| cef.label.Javascript Support | "true" |
| cef.ext.requestClientApplication | "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0" |
| cef.ext.siteid | 1509732 |
| cef.ext.sourceServiceName | "site123.abcd.info" |
| cef.ext.fileid | 3412341160002518171 |
| cef.ext.suid | 50005477 |
| cef.ext.start | 14532901213 |
| cef.deviceFacility | mia |

### Show the output of parseCEF() with changed label prefix

---

This example demonstrates how to change the prefix for label fields.

  * **Query Example**

    ```
    parseCEF(labelprefix="new.label.")
    ```

  * **Input Event Data**
    (From the same log line as the previous example):
    `CEF: 0 Incapsula | SIEMintegration|1|1|Illegal Resource Access |3| fileid=3412341160002518171 sourceServiceName=site123.abcd.info siteid=1509732 suid=50005477 requestClientApplication Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0 deviceFacility=mia cs2=true cs2Label-Javascript Support start=14532901213`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| cef.device.version | 1 |
| cef.device.vendor | Incapsula |
| cef.event\_class\_id | 1 |
| cef.device.product | SIEMintegration |
| cef.name | Illegal Resource Access |
| cef.version | 0 |
| cef.severity | 3 |
| new.label.Javascript Support | "true" |
| cef.ext.requestClientApplication | "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0" |
| cef.ext.siteid | 1509732 |
| cef.ext.sourceServiceName | "site123.abcd.info" |
| cef.ext.fileid | 3412341160002518171 |
| cef.ext.suid | 50005477 |
| cef.ext.start | 14532901213 |
| cef.deviceFacility | mia |

### Show the output of parseCEF() with changed extension prefix

This example demonstrates how to change the prefix for extension fields.

  * **Query Example**

    ```
    parseCEF(prefix="content.")
    ```

  * **Input Event Data**
    (From the same log line as the previous example):
    `CEF: 0 Incapsula | SIEMintegration|1|1|Illegal Resource Access |3| fileid=3412341160002518171 sourceServiceName=site123.abcd.info siteid=1509732 suid=50005477 requestClientApplication Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0 deviceFacility=mia cs2=true cs2Label-Javascript Support start=14532901213`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| cef.device.version | 1 |
| cef.device.vendor | Incapsula |
| cef.event\_class\_id | 1 |
| cef.device.product | SIEMintegration |
| cef.name | Illegal Resource Access |
| cef.version | 0 |
| cef.severity | 3 |
| cef.label.Javascript Support | "true" |
| content.requestClientApplication | "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0" |
| content.siteid | 1509732 |
| content.sourceServiceName | "site123.abcd.info" |
| content.fileid | 3412341160002518171 |
| content.suid | 50005477 |
| content.start | 14532901213 |
| content.deviceFacility | mia |