# tokenHash()

The `tokenHash()` function calculates a "structure hash," which is a hash value that is the same for inputs that are structurally similar, regardless of the order of the words or the specific values.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_tokenHash` | The name of the output field. |
| **field** | string | required | The name of the field to hash. The parameter name `field` can be omitted. |

***

## Function Operation

This function tokenizes an input string (splitting it by spaces), creates a hash for each individual token, and then adds those hashes together. Because the hashes are added, the final "structure hash" is identical for strings that contain the same tokens, even if they are in a different order. For example, the strings `"abc def ghi"` and `"def ghi abc"` will produce the same token hash. This function is intended for grouping and pattern analysis, **not** for cryptographic purposes.

***

## Example

### Group Similar Log Lines Using tokenHash()

This example uses `tokenHash()` to find common patterns in log messages by grouping together log lines that have the same structure but contain different specific values (like usernames or IP addresses).

* **Query Example**
    ```
    h := tokenHash(@rawstring)
    | groupBy(h, limit=max, function=[count(), collect(@rawstring, limit=3)])
    ```

* **Input Event Data**

| @rawstring |
| :--- |
| `User john.doe logged in from 192.168.1.100` |
| `User jane.smith logged in from 192.168.1.101`|
| `Failed login attempt from 10.0.0.1` |
| `Failed login attempt from 10.0.0.2` |
| `Database connection error: timeout after 30 seconds`|
| `Database connection error: timeout after 45 seconds`|

* **Step-by-Step**
    1.  `h := tokenHash(@rawstring)`: This first step calculates the structure hash for the `@rawstring` of each event and stores it in a new field called `h`. Log lines with the same structure (e.g., all the successful logins) will get the same hash value.
    2.  `groupBy(h, ...)`: The query then groups the events by this new hash value `h`.
    3.  Within each group, it performs two aggregations:
        * `count()`: Counts how many log lines share that structure.
        * `collect(@rawstring, limit=3)`: Collects up to three example raw strings from that group to provide context.

* **Result Event Data**
    The output is a table that groups structurally identical log messages, showing a count for each pattern and a few examples.

| h | _count | @rawstring |
| :--- | :--- | :--- |
| `1111b796` | 3 | `["User admin logged in from 192.168.1.102", "User jane.smith logged in from 192.168.1.101", "User john.doe logged in from 192.168.1.100"]` |
| `356fb767` | 2 | `["Failed login attempt from 10.0.0.2", "Failed login attempt from 10.0.0.1"]` |
| `90fadc1e` | 2 | `["Database connection error: timeout after 45 seconds", "Database connection error: timeout after 30 seconds"]` |