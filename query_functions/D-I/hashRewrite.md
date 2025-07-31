# hashRewrite()

The `hashRewrite()` function calculates a secure, cryptographic hash of a field's value and uses it to replace the original value, which is useful for obscuring sensitive data.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: input field name | The name of the output field. If not set, it defaults to the input field, effectively overwriting it. |
| **bits** | integer | optional <br> default: 256 | The number of bits of the hash to keep (must be a multiple of 8, between 8 and 512). |
| **field** | string | required | The name of the field to hash. The parameter name `field` can be omitted. |
| **hash** | string | optional <br> default: `sha256` | The hash algorithm to use. Values can be `sha256` or `sha512`. |
| **replaceInRawstring**| boolean | optional <br> default: `true` | If `true`, it also replaces all occurrences of the original value in the `@rawstring` with the hash. |
| **salt** | string | required | The name of the secret salt to use for hashing. |

***

## Example

### Replace a Sensitive Field with its Hash

This example shows how to replace the value in a sensitive field (like `ssn` for a social security number) with its secure hash.

* **Query Example**
    ```
    hashRewrite(ssn, salt="salt1")
    ```

* **Summary**
    This query is used to securely redact sensitive data during parsing.
    1.  The `hashRewrite()` function takes the value from the `ssn` field.
    2.  It computes a secure `sha256` hash of this value using a secret `salt` named `"salt1"`.
    3.  The original value in the `ssn` field is replaced with the computed hash.
    4.  By default, it will also find and replace the original SSN value within the event's `@rawstring`.