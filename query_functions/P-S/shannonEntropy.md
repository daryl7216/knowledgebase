# shannonEntropy()

The `shannonEntropy()` function calculates an entropy measure from a string of characters.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_shannonentropy` | The output name of the field to set. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `shannonEntropy()` function computes the Shannon entropy value for a given string of characters within a specified input field. Shannon entropy is a measure of randomness or unpredictability in data. The calculated entropy value is then stored in a new output field, which by default is named `_shannonentropy`, but a custom name can be assigned using the `as` parameter.

Shannon entropy values typically range from approximately 0 to 8.

  * A lower value indicates more predictable strings (less randomness).
  * A higher value indicates more random strings (less predictability).

This calculation is primarily used in security analysis and threat detection scenarios. For example, it can help identify randomly generated malware filenames or deviations from the expected distribution of characters in a domain name. It's important to note that the Shannon entropy value alone is not conclusive; context also matters. The calculated value can be used for further filtering or combined with other aggregate functions for more comprehensive analysis.

***

## Examples

### Calculate Shannon Entropy Value For String

This example calculates the Shannon Entropy value for the string "example.com" in the `dns_name` field.

  * **Query Example**

    ```
    entropy: shannonEntropy(dns_name)
    ```

  * **Input Event Data**
    `dns_name: example.com`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `entropy: shannonEntropy(dns_name)`: Calculates the Shannon entropy value of the field `dns_name` and returns the result in a new field named `entropy`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to calculate the Shannon entropy value for a string in a field. In this example, the returned value for `example.com` is 3.095795. This value is considered medium level and is typical for normal domain names. The Shannon entropy calculation is primarily used in security analysis and threat detection scenarios to identify, for example, randomly generated malware filenames or expected distribution of characters in a domain name. The Shannon value alone is not conclusive; context also matters. Use the value for further filtering or combine it with other aggregate functions.

  * **Result Event Data**

| entropy |
| :--- |
| 3.095795 |