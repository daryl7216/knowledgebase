# hashMatch()

The `hashMatch()` function provides a secure way to filter events by calculating a cryptographic hash of an input string and matching it against values in your data. It is designed to work with hashes created by the `hashRewrite()` function.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **bits** | integer | optional <br> default: 256 | The number of bits of the hash to keep (must be a multiple of 8, between 8 and 512). |
| **field** | string | optional <br> default: `@rawstring` | The field to look for an exact match in. If not set, it searches for a substring in `@rawstring`. |
| **hash** | string | optional <br> default: `sha256` | The hash algorithm to use. Values can be `sha256` or `sha512`. |
| **input** | string | required | The constant value to hash and use as the search term. The parameter name `input` can be omitted. |
| **salt** | string | required | The name of the secret salt to use for hashing. |

***

## Example

### Filter Events by Matching a Hashed SSN

This example demonstrates how to find events where the `ssn` field contains the secure hash of a specific Social Security Number, "12345678".

* **Query Example**
    ```
    ssn = hashMatch("12345678", salt="salt1")
    ```

* **Summary**
    This query is used to search for a sensitive value without exposing the value itself in the search.
    1.  The `hashMatch()` function takes the input string `"12345678"` and the secret `salt` named `"salt1"`.
    2.  It computes a secure hash of the input string using the specified salt.
    3.  The query then filters your events, keeping only those where the `ssn` field contains a value that exactly matches the computed hash.