# hash()

The `hash()` function computes a non-cryptographic hash of one or more fields and returns the result as an integer. It is designed to produce consistent output, meaning the same input values will always generate the same hash. This is useful for creating consistent sample groups or reducing the number of groups in a `groupBy()` aggregation.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_hash` | The name of the output field that will store the hash value. |
| **field** | array of strings | required | The field or fields to be hashed. The parameter name `field` can be omitted. |
| **limit** | number | optional | An upper bound on the returned number. The hash will be modulo this value, constraining it to the range `[0, limit-1]`. |
| **seed** | string | optional | An optional seed for the hash function. Using a different seed will produce a different hash for the same input data. |

***

## Examples

### Create Sample Groups Using a Hash

This example shows how to use `hash()` to group a high-cardinality field (like `ip_address`) into a small, fixed number of buckets for analysis.

* **Query Example**
    ```
    hash(ip_address, limit=10)
    | groupBy(_hash, function=count())
    ```

* **Input Event Data**

| ip_address | ... |
| :--- | :--- |
| 192.168.1.100 | ... |
| 192.168.1.201 | ... |
| 192.168.10.100| ... |
| 192.168.15.102| ... |
| ... | ... |

* **Step-by-Step**
    1.  `hash(ip_address, limit=10)`: This step processes each event, takes the value from the `ip_address` field, and computes a hash. The `limit=10` parameter ensures the resulting hash is an integer between 0 and 9. The result is stored in the default `_hash` field.
    2.  `groupBy(_hash, function=count())`: The events, now containing a `_hash` field, are grouped by that hash value. The `count()` function then counts how many events fall into each of the 10 hash buckets.

* **Result Event Data**
    The output is a summary table showing the distribution of events across the 10 hash buckets.

| _hash | _count |
| :--- | :--- |
| 2 | 1 |
| 3 | 1 |
| 6 | 1 |
| 8 | 3 |
| 9 | 1 |

---

### Generate Different But Consistent Hash Values Using a Seed

This example demonstrates how to use the `seed` parameter to create multiple, independent hash distributions from the same input data. This is useful for A/B/C testing or creating different sampling groups.

* **Query Example**
    ```
    hash(field=user_id, limit=5, as="hash1")
    | hash(field=user_id, limit=5, seed=10, as="hash2")
    | hash(field=user_id, limit=5, seed=20, as="hash3")
    ```

* **Input Event Data**

| user_id | action |
| :--- | :--- |
| user123 | login |
| user456 | logout |
| user123 | view |
| user789 | login |
| ... | ... |

* **Step-by-Step**
    1.  The first `hash()` call creates a hash of `user_id` and stores it in `hash1`.
    2.  The second `hash()` call uses the same `user_id` but with `seed=10`, creating a different but still consistent hash distribution in `hash2`.
    3.  The third `hash()` call uses `seed=20` to create a third independent hash distribution in `hash3`.

* **Result Event Data**
    The output shows that for a given `user_id`, the hash value is always the same within a single column (e.g., `user123` is always `3` in `hash1`), but the values differ across columns due to the different seeds.

| user_id | hash1 | hash2 | hash3 |
| :--- | :--- | :--- | :--- |
| user123 | 3 | 0 | 2 |
| user456 | 2 | 2 | 2 |
| user789 | 4 | 4 | 1 |