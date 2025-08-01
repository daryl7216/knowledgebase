# accumulate()

The `accumulate()` function applies an aggregation function cumulatively to a sequence of events. It is useful for calculating running totals, running averages, or other cumulative metrics over time or across a series of events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **current** | enum | optional <br> default: `include` | Controls whether to include the current event in the accumulation. <br> **Values:** `exclude`, `include`. |
| **function** | array of aggregate functions | required | The aggregator function to accumulate (e.g., `sum()`, `avg()`, `count()`). It only accepts functions that output at most a single event. The parameter name `function` can be omitted. |

***

## Function Operation

The `accumulate()` function must be used after an aggregator function (like `head()`, `sort()`, `bucket()`, or `timeChart()`) to ensure event ordering, as it requires a specific order to calculate cumulative values correctly. Only functions that produce a single value per event (e.g., `sum()`, `avg()`, `count()`) can be used within the `accumulate()` function.

***

## Examples

### Calculate Running Average of Field Values

This example uses `accumulate()` with the `avg()` function to compute a running average.

* **Query Example**
    ```
    head()
    | accumulate(avg(value))
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| C | 1 |
| d | 2 |

* **Step-by-Step**
    1.  `head()`: Ensures events are ordered by time, selecting the oldest ones.
    2.  `accumulate(avg(value))`: Computes the running average of the `value` field for all events, including the current one.

* **Result Event Data**

| _avg | key | value |
| :-- | :-- | :-- |
| 5 | a | 5 |
| 5.5 | b | 6 |
| 4 | C | 1 |
| 3.5 | d | 2 |

---

### Compute Cumulative Aggregation Across Buckets

This example shows how to use `accumulate()` with `timeChart()` to aggregate values across time intervals.

* **Query Example**
    ```
    timeChart(span=1000ms, function=sum(value))
    | accumulate(sum(_sum), as=_accumulated_sum)
    ```

* **Input Event Data**

| key @timestamp | value |
| :--- | :-- |
| a 1451606301001 | 5 |
| b 1451606301500 | 6 |
| a 1451606301701 | 1 |
| c 1451606302001 | 2 |
| b 1451606302201 | 6 |

* **Step-by-Step**
    1.  `timeChart(span=1000ms, function=sum(value))`: Groups data into 1-second buckets and sums the `value` field for each bucket, creating a new field called `_sum`.
    2.  `accumulate(sum(_sum), as=_accumulated_sum)`: Calculates a running total of the values in the `_sum` field and stores the result in a new field named `_accumulated_sum`.

* **Result Event Data**

| _bucket | _sum | _accumulated_sum |
| :--- | :-- | :-- |
| 1451606300000 | 0 | 0 |
| 1451606301000 | 12 | 12 |
| 1451606302000 | 8 | 20 |
| 1451606303000 | 0 | 20 |

---

### Compute Cumulative Aggregation For a Specific Group

This example demonstrates using `accumulate()` within the `groupBy()` function to compute a cumulative total for a specific group.

* **Query Example**
    ```
    head()
    | groupBy(key, function = accumulate(sum(value)))
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| a | 1 |
| c | 2 |
| b | 6 |

* **Step-by-Step**
    1.  `head()`: Selects the oldest events, ordered by time.
    2.  `groupBy(key, function = accumulate(sum(value)))`: Groups data by the `key` field and accumulates the sum of the `value` field within each group.

* **Result Event Data**

| key | _sum | value |
| :-- | :-- | :-- |
| a | 5 | 5 |
| a | 6 | 1 |
| b | 6 | 6 |
| b | 12 | 6 |
| c | 2 | 2 |


# array:append()

The `array:append()` function adds one or more values to an existing array. If the specified array does not exist, it will create a new one.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to which values will be appended. The parameter name `array` can be omitted. |
| **values** | array of expressions | required | A list of expressions to be added to the array. |

***

## Function Operation

The function appends values to an array. If any of the expressions provided in the `values` parameter don't evaluate to a value, that expression is simply skipped.

A critical requirement is that the target array must have **continuous, sequential indexes**. If there are gaps or missing indexes in the array, `array:append()` will start inserting new values at the first gap. This can lead to existing elements being overwritten.

For example, given an array with elements at indexes `0`, `1`, and `3`, the index `2` is a gap. A query like `array:append("array[]", values=["x", "y", "z"])` would place `"x"` at index `2`, `"y"` at index `3` (overwriting the original value), and `"z"` at index `4`.

***

## Examples

### Create New Array by Appending Expressions

This example demonstrates how to create a new array by appending expressions.

* **Query Example**
    ```
    array:append(array="related.user[]", values=[lower(source.user.name), lower(destination.user.name)])
    ```

* **Input Event Data**
    ```
    source.user.name="user_1" destination.user.name="USER_2"
    ```

* **Step-by-Step**
    1.  The query takes the values from the `source.user.name` and `destination.user.name` fields.
    2.  It applies the `lower()` function to convert both values to lowercase.
    3.  A new array named `related.user[]` is created containing the two lowercase usernames.

* **Result Event Data**

| source.user.name | destination.user.name | related.user[0] | related.user[1] |
| :--- | :--- | :--- | :--- |
| user_1 | USER_2 | user_1 | user_2 |

---

### Split Comma-Separated Strings into a New Array

This example shows how to split a string into an array and then append a new value to it.

* **Query Example**
    ```
    splitString(field=numbers, by=", ", as=numbarr)
    | array:append(array="numbarr[]", values=[4])
    ```

* **Input Event Data**
    ```
    numbers | "1,2,3"
    ```

* **Step-by-Step**
    1.  The `splitString()` function first converts the comma-separated string in the `numbers` field into a new array named `numbarr`.
    2.  The `array:append()` function then adds the number `4` to the end of the `numbarr` array.

* **Result Event Data**

| numbarr[0] | numbarr[1] | numbarr[2] | numbarr[3] |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 3 | 4 |


# array:contains()

The `array:contains()` function checks if a specified value exists within an array. If the value is not found in the array for a given event, that event is excluded from the search results.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to search within. The parameter name `array` can be omitted. |
| **value** | string | required | The exact value to search for within the array. |

***

## Function Operation

This function acts as a filter for events. It evaluates each event's specified array field to see if it contains an exact match for the provided value. If a match is found, the entire event is included in the output; otherwise, it is dropped.

***

## Examples

### Aggregate Array Content

This example finds all events where the `incidents` array contains "Cozy Bear" and then groups the results by the `host` field.

* **Query Example**
    ```
    array:contains("incidents[]", value="Cozy Bear")
    | groupBy(host)
    ```

* **Input Event Data**
    The query runs on events that contain an `incidents` array and a `host` field. For instance:
    * **Event 1**: `host` is `v1`, `incidents` array contains "Evil Bear" and "Cozy Bear".
    * **Event 2**: `host` is `v15`, `incidents` array contains "Fancy Fly", "Tiny Cat", and "Cozy Bears".

* **Step-by-Step**
    1.  `array:contains("incidents[]", value="Cozy Bear")`: This part of the query filters the events, keeping only those where the `incidents` array contains the exact value "Cozy Bear".
    2.  `groupBy(host)`: The remaining events are then grouped by the `host` field, and a count is generated.

* **Result Event Data**
    The final output is an aggregated count of events that matched, grouped by host.

| field | value |
| :--- | :--- |
| host | v1 |
| _count | 1 |

---

### Check for Values in Array

This example uses the function as a simple filter to check for a value in an array.

* **Query Example**
    ```
    array:contains("incidents[]", value="Cozy Bear")
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  `array:contains("incidents[]", value="Cozy Bear")`: It checks if the value "Cozy Bear" exists within the `incidents` array field of each event.
    3.  If the value is found, the entire event is kept in the search results. If not, the event is excluded.

* **Summary and Results**
    The query is used as a filter to see if a specific value is present in an array within a set of events. If the array contains the searched value, the event is returned in the results, including all other original fields and values in that event.


# array:dedup()

The `array:dedup()` function removes duplicate values from an array. The order of the first occurrence of each unique element is kept.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | array | required | The array from which to remove duplicate elements. The parameter name `array` can be omitted. |
| **asArray** | string | optional <br> default: input array | Specifies where to output the deduplicated array. |

***

## Function Operation

This function processes an array and removes any duplicate entries, ensuring that every element in the resulting array is unique. The original order is preserved based on the first time each element appeared in the input array.

***

## Example

### Deduplicate Values in Array

This example shows how to remove duplicate email addresses from an array named `emails[]`.

* **Query Example**
    ```
    parseJson()
    | array:dedup("emails[]")
    ```

* **Input Event Data**
    ```json
    {"emails": ["john@mail.com", "admin@mail.com", "jane@mail.com", "admin@mail.com"]}
    ```

* **Step-by-Step**
    1.  `parseJson()`: This function parses the raw event data to identify and create fields from the JSON structure.
    2.  `array:dedup("emails[]")`: This function then takes the `emails[]` array and removes any duplicate values.

* **Result Event Data**
    The output array contains only the unique values from the original array, with the duplicate `"admin@mail.com"` removed.

| emails[0] | emails[1] | emails[2] |
| :--- | :--- | :--- |
| john@mail.com | admin@mail.com | jane@mail.com |


# array:drop()

The `array:drop()` function takes the name of an array and removes all of its fields from an event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to be dropped. The parameter name `array` can be omitted. |

***

## Function Operation

This function requires the input array to have continuous, sequential indexes that start at 0 (e.g., `field[0]`, `field[1]`, `field[2]`). If the array has any gaps in its indexes (e.g., `field[0]`, `field[2]`), the function will only drop the elements up to the first gap. If an array with the given name doesn't exist, the function does nothing.

***

## Example

### Drop Fields From an Input Array

This example demonstrates how to use `array:drop()` to remove all fields belonging to a specific array.

* **Query Example**
    ```
    array:drop("a[]")
    ```

* **Input Event Data**
    An event contains multiple arrays (`a[]`, `b[]`, `c[]`) and other fields.

| field | value |
| :--- | :--- |
| a[0] | Dog |
| a[1] | Cat |
| a[42] | Horse |
| b[0] | Cat |
| c[0] | Horse |
| animal| cow |

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  `array:drop("a[]")` takes the array named `a[]` and drops all of its fields.
    3.  The other arrays (`b[]`, `c[]`) and the `animal` field are not affected.

* **Result Event Data**
    The resulting event no longer contains any fields from the `a[]` array.

| b[0] | c[0] | animal |
| :--- | :--- | :--- |
| Cat | Horse | cow |

# array:eval()

The `array:eval()` function evaluates a given function on all values within a specified array. It can overwrite the original array or save the results into a new array if the `asArray` parameter is used. The output array is always compacted, meaning its indices are continuous (0, 1, 2, ...).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the input array. The parameter name `array` can be omitted. |
| **asArray** | string | optional <br> default: overwrites input array | The name of the output array where the results will be stored. |
| **function** | non-aggregate function | required | The function to apply to each element of the array. |
| **var** | string | optional <br> default: input array name | The name of the variable to be used within the `function`. |

***

## Examples

### Evaluate Function Argument on Values in Array

This example squares the value of each item in an array and stores the result in a new array named `squared[]`.

* **Query Example**
    ```
    array:eval("values[]", asArray="squared[]", var="x", function={squared := x*x})
    ```

* **Input Event Data**

| values[0] | values[1] | values[2] |
| :--- | :--- | :--- |
| 2 | 3 | 4 |

* **Step-by-Step**
    1.  The query starts with events containing the `values[]` array.
    2.  `array:eval()` iterates through each element of `values[]`, assigning the element to the variable `x`.
    3.  The `function` `squared := x*x` is executed for each element, and the result is stored in the new `squared[]` array.

* **Result Event Data**

| field | value |
| :--- | :--- |
| squared[0] | 4 |
| squared[1] | 9 |
| squared[2] | 16 |

---

### Perform Formatting on All Values in an Array

This example converts all string values in the `devices[]` array to uppercase and saves them in a new array called `upperDevices[]`.

* **Query Example**
    ```
    array:eval("devices[]", asArray="upperDevices[]", var=d, function={upperDevices := upper(d)})
    ```

* **Input Event Data**
    The input is a series of JSON events, each with a `devices` array.
    ```json
    {"devices":["Thermostat", "Smart Plug"],"room":"Kitchen"}
    {"devices":["Smart Light", "Thermostat", "Smart Plug"],"room":"Living Room"}
    {"devices":["Smart Light", "Smart Plug"],"room":"Bedroom"}
    ```

* **Step-by-Step**
    1.  The query processes events containing the `devices[]` array.
    2.  `array:eval()` iterates through the array, assigning each device name to the variable `d`.
    3.  The `function` `upperDevices := upper(d)` converts the device name to uppercase.
    4.  The results are stored in the new `upperDevices[]` array, leaving the original `devices[]` array unchanged.

* **Result Event Data**

| devices[] | upperDevices[] |
| :--- | :--- |
| Thermostat | THERMOSTAT |
| Smart Plug | SMART PLUG |
| Smart Light| SMART LIGHT |

# array:exists()

The `array:exists()` function filters events by checking if an array contains at least one element that satisfies a specific condition. It is used for more complex scenarios where `array:contains()` is insufficient, such as when comparing array elements to other fields or using query functions in the condition.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to search. The parameter name `array` can be omitted. |
| **condition** | non-aggregate pipeline | required | A pipeline that acts as a condition. If an element passes through the pipeline, the event is included. |
| **var** | string | optional <br> default: input array name | The name of the variable representing an array element within the `condition`. |

***

## Function Operation

This function iterates through each element of the specified `array` and applies the `condition` to it. If any element in the array causes the `condition` pipeline to return true (i.e., the event passes through the filter), the entire event is included in the results. For structured arrays (arrays of objects), the `objectArray:exists()` function must be used instead.

***

## Examples

### Check For Existence of an Element Using Complex Conditions

This example uses a nested `if()` and `in()` function to create a more complex, OR-like filtering condition.

* **Query Example**
    ```
    kvParse()
    | array:exists(
        array="a[]",
        condition=if(in(a, values=[2, 5]), then=true, else=in(a, values=[3, 6]))
      )
    ```

* **Input Event Data**

| a[0] | a[1] |
| :--- | :--- |
| 1 | 2 |
| 1 | 3 |
| 1 | 4 |

* **Step-by-Step**
    1.  `kvParse()`: Parses the raw string data into key-value pairs.
    2.  `array:exists()`: Iterates through the `a[]` array of each event.
    3.  `condition`: For each element in `a[]`, it first checks if the element is `2` or `5`. If it is, the condition is met. If not, it then checks if the element is `3` or `6`.
    4.  The query keeps events where the `a[]` array contains `2`, `3`, `5`, or `6`.

* **Result Event Data**

| a[0] | a[1] |
| :--- | :--- |
| 1 | 2 |
| 1 | 3 |

---

### Check For Existence of an Element Using a Filtering Pipeline

This example demonstrates using a full pipeline within the condition to compare an array element against both a fixed value and another field in the event.

* **Query Example**
    ```
    kvParse()
    | array:exists(
        array="a[]",
        var=x,
        condition={ x=3 OR x=4 | test(x>=b) }
      )
    ```

* **Input Event Data**

| a[0] | a[1] | b |
| :--- | :--- | :-|
| 1 | 2 | 4 |
| 1 | 3 | 4 |
| 1 | 4 | 3 |

* **Step-by-Step**
    1.  `kvParse()`: Parses the raw string data into key-value pairs.
    2.  `array:exists()`: Iterates through the `a[]` array, assigning each element to the variable `x`.
    3.  `condition`: It applies a two-stage filter to `x`:
        * First, it checks if `x` is `3` or `4`.
        * If that passes, it then pipes the event to `test(x>=b)`, which checks if the array element `x` is greater than or equal to the value of the `b` field in the same event.
    4.  The query only keeps events that satisfy the entire pipeline.

* **Result Event Data**

| a[0] | a[1] | b |
| :--- | :--- | :-|
| 1 | 4 | 3 |

# array:filter()

The `array:filter()` function filters an array's elements using a provided function, creating a new array containing only the elements that meet the specified condition. The original order of the elements is maintained in the output.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The input array to be filtered. The parameter name `array` can be omitted. |
| **asArray** | string | optional <br> default: input array | The name of the output array where the filtered results will be stored. |
| **function**| non-aggregate function | required | The function to use for filtering the elements in the array. |
| **var** | string | optional <br> default: input array name | The name of the variable representing an array element to be used within the `function`. |

***

## Examples

### Deduplicate Compound Field Data

This complex example uses `array:filter()` as part of a larger pipeline to parse, filter, and deduplicate browser user agent strings.

* **Query Example**
    ```
    splitString(field=userAgent, by=" ", as=agents)
    | array:filter(array="agents[]", function={bname=/\//}, var="bname")
    | array:union(array=agents, as=browsers)
    | split(browsers)
    ```

* **Input Event Data**
    A single field `userAgent` contains a long string of information.
    ```
    Mozilla/5.0 (Macintosh Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko)
    ```

* **Step-by-Step**
    1.  `splitString()`: The `userAgent` string is split by spaces into a new array called `agents`.
    2.  `array:filter()`: This function filters the `agents` array. The condition `{bname=/\//}` keeps only the elements that contain a forward slash (`/`), which are typically the browser name/version pairs.
    3.  `array:union()`: This aggregates the filtered `agents` arrays from all events into a single array named `browsers`, removing any duplicate entries.
    4.  `split()`: The final `browsers` array is split into individual events, with each event containing one unique browser entry.

* **Result Event Data**
    The final output is a list of individual events, each containing a unique browser/toolkit string.

| _index | row[1] |
| :--- | :--- |
| 0 | Gecko/20100101 |
| 1 | Safari/537.36 |
| 2 | AppleWebKit/605.1.15|

---

### Filter an Array on a Given Condition

This example demonstrates the core use of `array:filter()` to select elements from an array that match a specific pattern.

* **Query Example**
    ```
    array:filter(array="mailto[]", var="addr", function={addr="ba*@example.com"}, asArray="out[]")
    ```

* **Input Event Data**
    An event containing an array of email addresses.
    ```
    mailto[0]=foo@example.com
    mailto[1]=bar@example.com
    mailto[2]=baz@example.com
    ```

* **Step-by-Step**
    1.  The query starts with an event that has a `mailto[]` array.
    2.  `array:filter()` iterates through the `mailto[]` array. Each element is assigned to the variable `addr`.
    3.  The `function` `{addr="ba*@example.com"}` checks if the `addr` variable starts with "ba".
    4.  Elements that match the condition are placed into a new array called `out[]`.

* **Result Event Data**
    The new `out[]` array contains only the emails that matched the filter.
    ```
    out[0]=bar@example.com
    out[1]=baz@example.com
    ```

# array:intersection()

This function determines the set intersection of array values over all input events, computing the values that occur in all events supplied to it.

> **Important**
> This function must be enabled using the `ArrayFunctions` feature flag.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The prefix of the array to process. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_intersection` | The name of the output array that will contain the intersection. |

***

## Function Operation

This function calculates the common values found in the specified array across multiple events. Key behaviors include:
* The order of values in the output array is not defined.
* Empty arrays are ignored by the function.
* If no arrays are found in the input, the output will be empty.

***

## Example

### Find Set Intersection Within an Array

This example finds the unique email addresses within a single array by performing an intersection on its own values. Note that this example seems to misuse the function's primary purpose of finding intersections *across events* and instead uses it to deduplicate a single array.

* **Query Example**
    ```
    array:intersection("mailto[]", as=unique_mails)
    ```

* **Input Event Data**
    An event containing an array of email addresses with duplicates.
    ```
    mailto[0]=foo@example.com
    mailto[1]=bar@example.com
    mailto[2]=bar@example.com
    ```

* **Step-by-Step**
    1.  The query begins with events containing the `mailto[]` array.
    2.  `array:intersection()` processes the `mailto[]` array, finds the common or unique set of element values, and stores them in a new array called `unique_mails`.

* **Result Event Data**
    The output is a new array, `unique_mails`, containing only the unique values from the input array.
    ```
    unique_mails[0]=foo@example.com
    unique_mails[1]=bar@example.com
    ```

# array:length()

The `array:length()` function calculates the number of elements present in an array.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to measure. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_length` | The name of the field that will store the output length. |

***

## Function Operation

This function counts the elements in an array, but with a critical rule: it only counts elements that are in sequential order starting from index 0. The count stops at the first gap or missing index. For example, for an array with elements at `animals[0]`, `animals[1]`, and `animals[3]`, the function will return a length of `2` because it stops at the missing `animals[2]` index.

If the function is applied to a field that is not an array, it will return `0`.

***

## Example

### Count Array Elements After Filtering

This example first filters events to find those where the first function used is `head`, and then it counts the number of functions in the array for those specific events.

* **Query Example**
    ```
    queryParserMetrics.function[0] = "head"
    | array:length("queryParserMetrics.function[]", as="_numberOfFunctions")
    ```

* **Input Event Data**
    An event containing an array of functions.
    ```
    queryParserMetrics.function[0]="head"
    queryParserMetrics.function[1]="bucket"
    queryParserMetrics.function[2]="groupBy"
    ```

* **Step-by-Step**
    1.  The query first filters events, keeping only those where the value of the `queryParserMetrics.function[0]` field is exactly `"head"`.
    2.  For the events that pass the filter, the `array:length()` function is then applied to the `queryParserMetrics.function[]` array.
    3.  It counts the number of elements in that array and places the result in a new field named `_numberOfFunctions`.

* **Result Event Data**
    The output is the total count of functions for an event that started with the `head` function.
    ```
    _numberOfFunctions 3
    ```

# array:reduceAll()

The `array:reduceAll()` function computes a single aggregate value from all elements of a specified array across all input events.

***

## Parameters

| Parameter | Type | Required/Default |
| :--- | :--- | :--- |
| **array** | string | required |
| **function**| array of aggregate functions | required |
| **var** | string | required |

***

## Function Operation

This function applies an aggregate function (like `avg()`, `count()`, `sum()`, or `max()`) to every element within a specified array, across all events in the search result. It effectively treats all the values from all the arrays as one large collection and then performs the calculation. This is a more efficient method than using `split()` to create individual events for each array element before aggregating.

For example, if you have three events each with an array `a[]`:
* Event 1: `[1, 4, 2]`
* Event 2: `[3, 5, 2]`
* Event 3: `[5, 2, 3]`

The function would operate on the combined set of values: `{1, 4, 2, 3, 5, 2, 5, 2, 3}`.

***

## Example

### Compute an Aggregated Value of an Array on All Events

This example uses the `max()` aggregate function to find the highest value from all elements across all `values[]` arrays in a set of events.

* **Query Example**
    ```
    array:reduceAll("values[]", var=x, function=max(x))
    ```

* **Input Event Data**
    Consider multiple events each containing a `values[]` array.
    * Event 1: `values[] = [10, 25, 15]`
    * Event 2: `values[] = [5, 30, 20]`
    * Event 3: `values[] = [40, 12, 8]`

* **Step-by-Step**
    1.  The query begins with a set of events, each containing a `values[]` array.
    2.  `array:reduceAll()` gathers all elements from every `values[]` array across all events into a single conceptual list: `{10, 25, 15, 5, 30, 20, 40, 12, 8}`.
    3.  The `max()` function is applied to this combined list to find the highest value.

* **Result Event Data**
    The output is a single event with a single field containing the maximum value found.
    ```
    _max=40
    ```

# array:reduceColumn()

The `array:reduceColumn()` function computes an aggregate value for each array element that shares the same index across multiple events.

> **Important**
> This function is experimental and should not be used in production environments. It must also be enabled using the `ArrayFunctions` feature flag.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The prefix of the array to process. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_reduceColumn` | The name for the output array. |
| **function**| array of aggregate functions | required | The aggregate function(s) to apply to each column (e.g., `max()`, `avg()`). |
| **var** | string | required | A placeholder name for array elements to be used in the aggregate function. |

***

## Function Operation

This function works by grouping together all array elements that have the same index from all input events. It then applies a specified aggregate function to each of these groups (or "columns"). For instance, it would take all elements at index `[0]` from all events and calculate their average, then do the same for all elements at index `[1]`, and so on. The results are returned in a new array where each index corresponds to the result of the calculation for that column.

***

## Example

### Compute Average Value for Each Array Element With Same Index

This example calculates the average value for each column across multiple events containing an `ages[]` array.

* **Query Example**
    ```
    array:reduceColumn("ages[]", var=x, function=avg(x))
    ```

* **Input Event Data**
    Four separate events, each with an `ages[]` array of the same length.

| ages[0] | ages[1] | ages[2] |
| :--- | :--- | :--- |
| 16 | 32 | 64 |
| 15 | 30 | 45 |
| 1 | 2 | 4 |
| 89 | 57 | 67 |

* **Step-by-Step**
    1.  The query starts with a set of events, each containing an `ages[]` array.
    2.  `array:reduceColumn()` groups the values by their index:
        * Column 0: `{16, 15, 1, 89}`
        * Column 1: `{32, 30, 2, 57}`
        * Column 2: `{64, 45, 4, 67}`
    3.  The `avg()` function is applied to each of these columns independently.
    4.  The results are stored in a new array named `_reduceColumn` by default.

* **Result Event Data**
    The output is a single event with one array containing the calculated average for each column.

| _reduceColumn[0] | _reduceColumn[1] | _reduceColumn[2] |
| :--- | :--- | :--- |
| 40.3 | 40.3 | 45.0 |

# array:reduceRow()

The `array:reduceRow()` function computes an aggregated value for the array within each event individually.

> **Important**
> This function is experimental, under active development, and should not be used in production. It must be enabled using the `ArrayFunctions` feature flag.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to process. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_reduceRow` | The name for the output field. |
| **function**| function | required | The aggregate function to use (e.g., `avg()`, `max()`), which must output a single event with a single field. |
| **var** | string | required | A placeholder name for the array element to be used in the aggregate function. |

***

## Function Operation

This function operates on each event (or row) separately. For each event, it applies an aggregate function to all the elements within that event's array. It is essentially a shorthand for using `array:eval()` to process each event's array. The result is a new field added to each event, containing the calculated value for that event's array.

***

## Example

### Calculate Average of Field Values in an Array

This example calculates the average of values in an `ages[]` array for each individual event.

* **Query Example**
    ```
    array:reduceRow("ages[]", var=x, function=avg(x))
    ```

* **Input Event Data**
    A series of events, each containing its own `ages[]` array.

| ages[0] | ages[1] | ages[2] |
| :--- | :--- | :--- |
| 16 | 32 | 64 |
| 15 | 30 | 45 |
| 1 | 2 | 4 |
| 89 | 57 | 67 |

* **Step-by-Step**
    1.  The query starts with a set of events, each with an `ages[]` array.
    2.  `array:reduceRow()` processes each event one by one.
    3.  For the first event, it calculates the average of `{16, 32, 64}`.
    4.  For the second event, it calculates the average of `{15, 30, 45}`, and so on for all subsequent events.
    5.  The calculated average for each event is placed in a new `_avg` field for that same event.

* **Result Event Data**
    The original events are returned with a new field, `_avg`, showing the calculated average for each row.

| ages[0] | ages[1] | ages[2] | _avg |
| :--- | :--- | :--- | :--- |
| 16 | 32 | 64 | 37.333 |
| 15 | 30 | 45 | 30 |
| 1 | 2 | 4 | 2.67 |
| 89 | 57 | 67 | 71 |

# array:regex()

The `array:regex()` function filters events by checking if a regular expression pattern matches any of the values within a specified array. If no value in the array matches the pattern for a given event, that event is excluded from the search results.

> **Note**: It is recommended to always test regular expressions directly within LogScale to ensure compatibility, rather than relying on third-party tools.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to search within. The parameter name `array` can be omitted. |
| **flags** | string | optional | The regex modifier flags to use. Possible values are:<br>- `F`: Use LogScale Regex Engine v2.<br>- `d`: Period (`.`) also includes newline characters.<br>- `i`: Ignore case for matched values.<br>- `m`: Multi-line parsing of regular expressions. |
| **regex** | regex | required | The regular expression pattern to search for within the array values. |

***

## Example

### Find Matches in Array and Group Results

This example uses a regular expression to find events where an array element starts with "Cozy Bear" and then groups the results by host.

* **Query Example**
    ```
    array:regex("incidents[]", regex="^Cozy Bear.*")
    | groupBy(host)
    ```

* **Input Event Data**
    A set of events each containing a `host` field and an `incidents` array.

| host | incidents[0] | incidents[1] | incidents[2] |
| :--- | :--- | :--- | :--- |
| v1 | Evil Bear | Cozy Bear | |
| v15 | Fancy Fly | Tiny Cat | Cozy Bears |
| v22 | Fancy Fly | Tiny Cat | Cold Bears |
| v4 | Fancy Fly | Tiny Cat | Cozy Bearskins|
| v1 | Evil Bear | Cozy Bears | |

* **Step-by-Step**
    1.  The query starts with the source events.
    2.  `array:regex("incidents[]", regex="^Cozy Bear.*")`: This filters the events, keeping only those where at least one element in the `incidents` array starts with the string "Cozy Bear".
    3.  `groupBy(host)`: The events that pass the filter are then grouped by their `host` value, and a count is generated for each host.

* **Result Event Data**
    The output is an aggregated count of matching events for each host.

| host | _count |
| :--- | :--- |
| v1 | 2 |
| v15 | 1 |
| v4 | 1 |

# array:rename()

The `array:rename()` function takes an array and renames all of its fields.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to rename. The parameter name `array` can be omitted. |
| **asArray** | string | required | The new name for the array. |

***

## Function Operation

This function requires that the input array has continuous, sequential indexes starting from `[0]`. If the function encounters a gap in the indexes (e.g., elements exist at `[0]`, `[1]`, and `[3]` but not `[2]`), it will only rename the elements up to that first gap.

If the target array doesn't exist, or if the new name is the same as the old name, the function will do nothing. If an array with the new name already exists, it will be overwritten.

***

## Example

### Rename Existing Fields in an Array

This example renames an array from `mail[]` to `user.email[]`, which is a common task when normalizing data to a standard schema like the Elastic Common Schema (ECS).

* **Query Example**
    ```
    array:rename(array="mail[]", asArray="user.email[]")
    ```

* **Input Event Data**
    An event with an array named `mail[]`.
    ```
    mail[0]='user0@example.com'
    mail[1]='user1@example.com'
    mail[2]='user2@example.com'
    ```

* **Step-by-Step**
    1.  The query starts with events containing the `mail[]` array.
    2.  `array:rename()` takes all fields belonging to the `mail[]` array and renames them to `user.email[]`. The original `mail[]` fields are removed.

* **Result Event Data**
    The event now has the `user.email[]` array instead of the `mail[]` array.
    ```
    user.email[0]=user0@example.com
    user.email[1]=user1@example.com
    user.email[2]=user2@example.com
    ```

# array:sort()

The `array:sort()` function sorts the elements of an array. By default, it sorts elements as case-insensitive strings in ascending order and replaces the input array with the result. You can also sort elements as numbers and specify a different output array.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | array name | required | The array you want to sort. |
| **asArray** | string | optional <br> default: input array | The name of the array where the sorted output will be stored. |
| **order** | string | optional <br> default: `asc` | The sorting order. Values can be `ascending` (`asc`) or `descending` (`desc`). |
| **type** | string | optional <br> default: `string` | The data type to use for sorting. Values can be `string` for alphabetical sorting or `number` for numerical sorting. |

***

## Function Operation

The `array:sort()` function has several key behaviors:
* **Default Sort**: Without specifying `type` or `order`, it performs a case-insensitive, ascending (A-Z) string sort.
* **Numeric Sort with Non-Numbers**: When `type=number`, any elements that are not numbers are sorted separately as strings and placed *after* the sorted numbers, regardless of whether the order is ascending or descending.
* **Overwriting Arrays**: If you use `asArray` to specify an output array that already exists, the function will overwrite its elements. However, if the target array has more elements than the source array, the extra elements at the end of the target array are **not** removed.

***

## Examples

### Sorting Numbers with Non-Numbers Present

This example shows how `array:sort()` handles an array containing both numbers and strings when sorting numerically. The numbers are sorted first, and the strings are sorted alphabetically after the numbers.

* **Query Example**
    ```
    parseJson()
    | array:sort("array[]", type=number)
    ```

* **Input Event Data**
    ```json
    {"array": ["banana", 5, 2, "apple", 10, 41]}
    ```

* **Result Event Data**
    The numbers are sorted in ascending order, followed by the alphabetically sorted strings.

| array[0] | array[1] | array[2] | array[3] | array[4] | array[5] |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 2 | 5 | 10 | 41 | "apple" | "banana" |

---

### Overwriting a Larger Existing Array

This example demonstrates the behavior when sorting an array and outputting the result to a pre-existing, larger array. The sorted elements overwrite the beginning of the target array, but the extra elements in the target array remain untouched.

* **Query Example**
    ```
    parseJson()
    | array:sort("a[]", asArray="b[]")
    ```

* **Input Event Data**
    An event with two arrays, where `b[]` is larger than `a[]`.
    ```json
    {"a": [3, 1, 2], "b": [5, 5, 5, 5]}
    ```

* **Result Event Data**
    The `a[]` array is sorted numerically `[1, 2, 3]` and its elements overwrite the first three elements of `b[]`. The fourth element of `b[]` is not removed.

| a[0] | a[1] | a[2] | b[0] | b[1] | b[2] | b[3] |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 3 | 1 | 2 | 1 | 2 | 3 | 5 |

# array:union()

[cite_start]The `array:union()` function determines the set union of array values over all input events[cite: 6]. [cite_start]It gathers all unique values from a specified array across all events into a single output array[cite: 7].

> **Important**
> [cite_start]This function is considered experimental and should not be used in production[cite: 4]. [cite_start]It must be enabled using the `ArrayFunctions` feature flag[cite: 5].

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | [cite_start]The prefix of the array to process[cite: 9]. |
| **as** | string | optional <br> default: `_union` | [cite_start]The name of the output array[cite: 9]. |

***

## Function Operation

[cite_start]This function combines all values from the specified array across multiple events and removes duplicates to produce a single array containing only unique values[cite: 6, 70]. [cite_start]The order of the values in the final output array is not defined[cite: 8]. [cite_start]If no arrays are found in the input events, the output will be an empty array[cite: 8].

***

## Example

### Deduplicate Compound Field Data

This complex example demonstrates how `array:union()` can be used within a larger pipeline to parse, filter, and deduplicate information from a compound field like a `userAgent` string.

* **Query Example**
    ```
    splitString(field=userAgent, by=" ", as=agents)
    | array:filter(array="agents[]", function={bname=/\//}, var="bname")
    | array:union(array=agents, as=browsers)
    | split(browsers)
    ```

* **Input Event Data**
    An event with a single `userAgent` field containing a long string.
    ```
    Mozilla/5.0 (Macintosh Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko)
    ```

* **Step-by-Step**
    1.  [cite_start]`splitString()`: The `userAgent` string is first split by spaces into a new array called `agents`[cite: 34].
    2.  [cite_start]`array:filter()`: This function filters the `agents` array to keep only the elements that contain a forward slash (`/`), which are typically the browser name/version pairs[cite: 41].
    3.  [cite_start]`array:union()`: This aggregates the filtered `agents` arrays from all events into a single array named `browsers`, which eliminates any duplicate entries[cite: 44, 45].
    4.  [cite_start]`split()`: Finally, the `browsers` array is split into individual events, with each event containing one unique browser entry[cite: 52].

* **Result Event Data**
    The final output is a list of individual events, each containing a unique browser or toolkit string.

| _index | row[1] |
| :--- | :--- |
| 0 | Gecko/20100101 |
| 1 | Safari/537.36 |
| 2 | AppleWebKit/605.1.15 |

# asn()

The `asn()` function enriches an event by adding the Autonomous System (AS) number and organization associated with a given IP address.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: input field name | The prefix to use for the new fields created by the function. |
| **field** | string | optional <br> default: `ip` | The field containing the IP address to look up. The parameter name `field` can be omitted. |

***

## Function Operation

This function takes an IP address from a specified field and uses the MaxMind GeoLite2 database to find its associated AS number and organization. By default, it reads from the `ip` field and creates two new fields: `ip.asn` and `ip.org`. You can specify a different source field and a different prefix for the output fields. The function can process both IPv4 and IPv6 addresses.

***

## Example

### Determine AS Number with Custom Input and Output Fields

This example shows how to use `asn()` with a custom source field for the IP address (`ipaddr`) and a custom prefix for the new fields (`address`).

* **Query Example**
    ```
    asn(field=ipaddr, as=address)
    ```

* **Step-by-Step**
    1.  The query starts with events that have an `ipaddr` field containing an IP address.
    2.  The `asn()` function takes the value from the `ipaddr` field.
    3.  It then adds two new fields to the event: `address.asn` and `address.org`, containing the Autonomous System number and organization name for that IP.

# avg()

The `avg()` function calculates the average for a numeric field over a set of events. The result is returned in a field named `_avg` by default.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_avg` | The name for the output field that will contain the calculated average. |
| **field** | string | required | The field from which to extract numbers to be averaged. The parameter name `field` can be omitted. |

***

## Example

### Calculate the Mean of CPU Time

This example calculates the average CPU time from a field, converts it from nanoseconds to milliseconds, and formats the final result to two decimal places. This is useful for determining processing power or troubleshooting high CPU usage.

* **Query Example**
    ```
    avg(field=cputimeNanos)
    | cputime := (_avg/1000000)
    | format("%,.2f", field=_avg, as=_avg)
    ```

* **Step-by-Step**
    1.  `avg(field=cputimeNanos)`: First, the query calculates the average of the `cputimeNanos` field across all events.
    2.  `cputime := (_avg/1000000)`: The result, stored in `_avg`, is then divided by 1,000,000 to convert the time from nanoseconds to milliseconds, making it easier to read.
    3.  `format("%,.2f", field=_avg, as=_avg)`: Finally, the `format()` function is used to reformat the calculated average to show only two decimal points.

* **Result Event Data**
    The final output is a single field `_avg` containing the formatted average CPU time in milliseconds.
    ```
    _avg
    0.14
    ```

# base64Decode()

The `base64Decode()` function performs Base64 decoding of a field. After decoding from Base64, the value is interpreted as a binary representation of a string according to the specified character set.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_base64Decode` | The name of the field where the decoded value will be stored. |
| **charset** | string | optional <br> default: `UTF-8` | The character set to use when converting the decoded bytes into a string. Possible values are `UTF-8`, `UTF-16BE`, `UTF-16LE`, and `ISO-8859-1`. |
| **field** | string | required | The field containing the Base64 value to be decoded. The parameter name `field` can be omitted. |

***

## Function Operation

The function first decodes a Base64 string into its binary representation. It then transforms these bytes into a string using the specified `charset`. To work with raw binary data, the `ISO-8859-1` character set should be used.

### Behavior on Invalid Encoding
* Characters outside the standard Base64 alphabet are ignored.
* If the input string is not a valid Base64 encoding, the output field will not be created.
* If a decoded code point is invalid for the selected `charset`, it is replaced with a placeholder character.

***

## Example

### Perform Base64 Decoding of a Field

This example demonstrates how to decode a Base64-encoded string from a field named `encoded` and store the result in a new field named `decoded`.

* **Query Example**
    ```
    decoded := base64Decode(encoded)
    ```

* **Step-by-Step**
    1.  The query starts with events that have a field named `encoded` containing a Base64 string.
    2.  The `base64Decode()` function reads the value from the `encoded` field.
    3.  It decodes the Base64 value and stores the resulting string in a new field called `decoded`.

# base64Encode()

The `base64Encode()` function performs Base64 encoding of a field's value. This process converts the input string to UTF-8 to ensure that values with special characters are transformed into a known set of non-special characters.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_base64Encode` | The name of the field where the encoded value will be stored. |
| **field** | string | required | The field containing the value to be encoded. The parameter name `field` can be omitted. |

***

## Function Operation

This function always encodes input strings as UTF-8 and does not accept a `charset` parameter, which is a key difference from its counterpart, `base64Decode()`. It's designed to make data handling easier for systems that only accept ASCII characters. It is important to note that while Base64 encoding is useful for data transport, it is not a secure method of encryption.

***

## Example

### Perform Base64 Encoding of a Field

This example takes a simple string, "Hello, World!", and converts it to its Base64 representation.

* **Query Example**
    ```
    base64Encode(a)
    ```

* **Input Event Data**
    An event with a field `a` containing the string `Hello, World!`.

* **Step-by-Step**
    1.  The query starts with an event containing the field `a`.
    2.  `base64Encode()` takes the input string `Hello, World!` from field `a`.
    3.  It converts the string to UTF-8 and then encodes it with Base64.
    4.  The result is stored in the default output field, `_base64Encode`.

* **Result Event Data**
    The output is a new field containing the Base64-encoded string.
    ```
    _base64Encode="SGVsbG8sIFdvcmxkIQ=="
    ```

# beta:param()

The `beta:param()` function reads a specified parameter and assigns its value to a new field within an event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | required | The name of the output field that will store the parameter's value. |
| **parameter** | string | required | The name of the parameter to read. The parameter name `parameter` can be omitted. |

***

## Examples

There are no examples provided in the document for this function.

# beta:repeating()

The `beta:repeating()` function marks a live query as "repeating," which means it is implemented by making a series of repeated historical queries.

> **Important**
> This is a beta feature that may not work as expected and could be removed without warning. The `RepeatingQueries` feature must be enabled for this function to be available. If the feature is disabled, any query, alert, or dashboard using this function will fail.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | The name of the output field. |
| **interval** | long | optional <br> default: system-selected | The time interval between successive historical queries, defined using Relative Time Syntax. The parameter name `interval` can be omitted. |

***

## Function Operation

This function allows for the use of functions not normally supported in live queries (such as `selfJoin()` or certain uses of `join()`) by executing the query as a series of historical queries. When the query snapshot cache is enabled, these repeated queries can reuse previous results for better performance. This function has no effect when used in a historical query.

***

## Example

### Repeat a Query Every 5 Minutes

This example demonstrates how to set a live query to repeat every five minutes.

* **Query Example**
    ```
    beta:repeating(5m)
    ```

# bitfield:extractFlags()

The `bitfield:extractFlags()` function decodes an integer into its bit representation and extracts the bits at specified positions, creating new boolean fields for each specified bit.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | string | required | The name of the field containing the integer to be decoded. |
| **onlyTrue** | boolean | optional <br> default: `false` | If set to `true`, new fields will only be created for bits that are `true`. Bits that are `false` will be ignored. |
| **output** | array of arrays of strings | required | A list of pairs, where each pair contains a bit index (0-63) and the corresponding field name to be created. |

***

## Function Operation

This function reads an integer value from the specified `field`, converts it into its binary form, and then checks the state of specific bits as defined in the `output` parameter. For each bit index and name pair provided, a new field is created with a boolean value (`true` or `false`) representing whether that bit was set (1) or not (0). The bits are indexed from 0 and can handle integers up to 64 bits long.

***

## Example

### Decode and Extract Bit Flags

This example decodes an integer from the `MemoryDescriptionFlags` field and extracts several named flags, creating new fields only for the flags that are true.

* **Query Example**
    ```
    bitfield:extractFlags(field="MemoryDescriptionFlags", onlyTrue=true, output=[
        [0, INVALID_ADDRESS],
        [1, CURRENT_STACK],
        [2, JIT_DOTNET],
        [3, MZ],
        [4, CODE],
        [5, MODULE],
        [6, MAPPED],
        [7, REFLECIVE_PE],
        [8, JIT_FLASH],
        [9, PRIMARY_MODULE],
        [10, PRIVATE_MEMORY],
        [11, KNOWN_FUNCTION],
        [12, FREE_MEMORY]
    ])
    ```

* **Input Event Data**
    An event where the field `MemoryDescriptionFlags` has a value of `1234`.
    ```
    MemoryDescriptionFlags, 1234
    // 1234 in binary is 10011010010
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `MemoryDescriptionFlags` field.
    2.  `bitfield:extractFlags()` reads the integer `1234`.
    3.  It checks the state of bits 0 through 12 as defined in the `output` array.
    4.  Because `onlyTrue=true` is set, it creates new fields only for the bits that are set to 1 in the binary representation of 1234.

* **Result Event Data**
    The resulting event has new boolean fields corresponding to the "on" bits from the input value.

| CODE | CURRENT_STACK | FREE_MEMORY | INVALID_ADDRESS |
| :--- | :--- | :--- | :--- |
| true | true | false | false |

# bitfield:extractFlagsAsArray()

The `bitfield:extractFlagsAsArray()` function decodes an integer from a field into its bit representation and creates an array containing the names of the flags that correspond to "true" (set) bits.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **asArray** | string | optional <br> default: `_flags[]` | The name of the output array. |
| **field** | string | required | The name of the field containing the integer to be decoded. |
| **flagNames**| array of arrays of strings | required | A list of pairs, where each pair contains a bit index (0-63) and the corresponding flag name. |

***

## Function Operation

This function reads an integer value, converts it to its binary form, and then checks which bits are set to 1. For each bit that is set, the function takes the corresponding name from the `flagNames` list and adds it to the output array. The names in the output array are sorted in order from the lowest bit to the highest bit.

Important validation rules for the `flagNames` parameter:
* Bit indices must be non-negative numbers less than 64.
* Both the bit indices and the flag names must be unique within the list.

If an array with the specified output name already exists, it will be overwritten.

***

## Example

### Decode and Extract True Bits as an Array

This example decodes an integer from the `MemoryDescriptionFlags` field and creates an array containing the names of all the flags that are enabled.

* **Query Example**
    ```
    bitfield:extractFlagsAsArray(field="MemoryDescriptionFlags", flagNames=[
        [0, INVALID_ADDRESS],
        [1, CURRENT_STACK],
        [2, JIT_DOTNET],
        [3, MZ],
        [4, CODE],
        [5, MODULE],
        [6, MAPPED],
        [7, REFLECTIVE_PE],
        [8, JIT_FLASH],
        [9, PRIMARY_MODULE],
        [10, PRIVATE_MEMORY],
        [11, KNOWN_FUNCTION],
        [12, FREE_MEMORY]
    ], asArray="flagsArray[]")
    ```

* **Input Event Data**
    An event where the field `MemoryDescriptionFlags` has a value of `1234`.
    ```
    MemoryDescriptionFlags, 1234
    // 1234 in binary is 10011010010
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `MemoryDescriptionFlags` field.
    2.  `bitfield:extractFlagsAsArray()` reads the integer `1234` and converts it to its binary representation.
    3.  It then identifies which bits are set to `1` and collects the corresponding names from the `flagNames` list.
    4.  These names are placed into a new array called `flagsArray[]`.

* **Result Event Data**
    The output is a new array containing the names of all the flags that were set in the input value.

| flagsArray[0] | flagsArray[1] | flagsArray[2] | flagsArray[3] | flagsArray[4] |
| :--- | :--- | :--- | :--- | :--- |
| CURRENT_STACK | CODE | MAPPED | REFLECTIVE_PE | PRIVATE_MEMORY |

# bitfield:extractFlagsAsString()

The `bitfield:extractFlagsAsString()` function decodes an integer into its bit representation and extracts the names of the "true" (set) bits into a single, delimited string.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_flags` | The name of the output field that will store the string of flag names. |
| **field** | string | required | The name of the field containing the integer to be decoded. |
| **flagNames**| array of arrays of strings | required | A list of pairs, where each pair contains a bit index (0-63) and the corresponding flag name. |
| **separator**| string | optional <br> default: ` ` (space) | The separator to use between the names of the true flags in the output string. |

***

## Function Operation

This function reads an integer value, converts it to its binary form, and identifies which bits are set to 1. It then collects the corresponding names from the `flagNames` list and joins them into a single string, using the specified `separator`. The flag names in the output string are always listed in ascending order, from the lowest bit to the highest.

**Validation:**
* In the `flagNames` parameter, bit indices must be unique, non-negative numbers below 64.
* The flag names themselves must also be unique.

***

## Example

### Decode and Extract True Bits as a Comma-Separated String

This example decodes an integer, extracts the names of the true bits, and formats them as a comma-separated string in a custom field named `trueFlags`.

* **Query Example**
    ```
    bitfield:extractFlagsAsString(
        field="MemoryDescriptionFlags", 
        flagNames=[
            [0, INVALID_ADDRESS],
            [1, CURRENT_STACK],
            [2, JIT_DOTNET],
            [3, MZ],
            [4, CODE],
            [5, MODULE],
            [6, MAPPED],
            [7, REFLECIVE_PE],
            [8, JIT_FLASH],
            [9, PRIMARY_MODULE],
            [10, PRIVATE_MEMORY],
            [11, KNOWN_FUNCTION],
            [12, FREE_MEMORY]
        ], 
        as="trueFlags", 
        separator=", "
    )
    ```

* **Input Event Data**
    An event where the field `MemoryDescriptionFlags` has a value of `1234`.
    ```
    MemoryDescriptionFlags, 1234
    // 1234 in binary is 10011010010
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `MemoryDescriptionFlags` field.
    2.  `bitfield:extractFlagsAsString()` reads the integer `1234` and identifies the bits that are set to `1`.
    3.  It collects the names for these true bits (e.g., `CURRENT_STACK`, `CODE`, `MAPPED`, etc.).
    4.  The function then joins these names into a single string, separated by a comma and a space, and stores it in the `trueFlags` field.

* **Result Event Data**
    The resulting event has a new field, `trueFlags`, containing the comma-separated string of active flags.
    ```
    trueFlags="CURRENT_STACK, CODE, MAPPED, REFLECIVE_PE, PRIVATE_MEMORY"
    ```

# bucket()

The `bucket()` function extends `groupBy()` to group events by time, dividing the search time interval into a specified number of time-based "buckets." Each event is placed into a bucket based on its timestamp, and the function creates a new `_bucket` field containing the start time of that bucket in milliseconds.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **buckets** | number | optional | Defines the number of buckets to divide the query time interval into (1 to 1500). |
| **field** | string | optional | Specifies one or more fields to group by in addition to the time bucket. |
| **function**| array of aggregate functions | optional <br> default: `count(as=_count)` | Specifies the aggregate function(s) to perform on each group. |
| **limit** | integer | optional <br> default: 10 | The maximum number of series to produce (up to 500), prioritizing the series with the highest numerical values. |
| **minSpan** | long | optional | Sets the minimum allowed time span for each bucket, using Relative Time Syntax (e.g., `1hour`). |
| **span** | relative-time | optional <br> default: auto | Defines a fixed time span for each bucket (e.g., `1min`, `3weeks`). |
| **timezone**| string | optional | Defines the time zone for bucketing, such as `UTC` or `'+02:00'`. |
| **unit** | array of strings | optional | Applies a unit conversion to the output columns (e.g., `bytes/span` to `Kbytes/day`). |

***

## Function Operation

The `bucket()` function groups data into time-based intervals. It's important to note that you cannot use anchored time units (like `/d` for start of day) for the `span` parameter. The function calculates buckets by dividing the query time interval. Due to how time boundaries are handled, this can sometimes result in more buckets being created than specified. For example, a one-hour query with a one-minute span might produce 61 buckets to include partial data at the beginning and end of the time range.

***

## Examples

### Calculate Linear Relationship Between Two Variables

This example uses `bucket()` to prepare data for a linear regression analysis, calculating the relationship between the total size of data sent and the average server load over time.

* **Query Example**
    ```
    bucket(function=[sum(bytes_sent, as=x), avg(server_load_pct, as=y)])
    | linReg(x=x, y=y)
    ```

* **Input Event Data**
    A series of events with `bytes_sent` and `server_load_pct` values.

| @timestamp | bytes_sent | server_load_pct |
| :--- | :--- | :--- |
| 2024-01-15T09:00:00Z | 156780 | 45.2 |
| 2024-01-15T09:05:00Z | 234567 | 52.8 |
| 2024-01-15T09:10:00Z | 189234 | 48.6 |
| 2024-01-15T09:15:00Z | 345678 | 65.3 |
| ... | ... | ... |

* **Step-by-Step**
    1.  `bucket()`: This function first groups the data into time buckets. Within each bucket, it calculates two things:
        * The `sum()` of `bytes_sent`, renaming it to `x`.
        * The `avg()` of `server_load_pct`, renaming it to `y`.
    2.  `linReg()`: The results are then piped to the linear regression function, which correlates the sum of bytes sent (`x`) with the average server load (`y`).

* **Result Event Data**
    The output shows the statistical relationship between the two variables.

| _slope | _intercept | _r2 | _n |
| :--- | :--- | :--- | :--|
| 0.000106... | 28.934... | 0.991... | 10 |

---

### Calculate Linear Relationship Grouped by a Field

This more advanced example analyzes how different HTTP request types impact server load by performing a separate linear regression for each type.

* **Query Example**
    ```
    bucket(function=[avg(server_load_pct, as=y), groupBy(request_type, function=count(as=x))])
    | groupBy(request_type, function=linReg(x=x, y=y))
    ```

* **Input Event Data**
    Events containing server load and request type information.

| @timestamp | server_load_pct | request_type |
| :--- | :--- | :--- |
| 2024-01-15T09:05:00.000Z | 52.8 | GET |
| 2024-01-15T09:05:00.000Z | 52.8 | PUT |
| 2024-01-15T09:10:00.000Z | 48.6 | GET |
| ... | ... | ... |

* **Step-by-Step**
    1.  `bucket()`: Within each time bucket, the function calculates the average `server_load_pct` (as `y`) and also performs a nested `groupBy()` to count the number of requests for each `request_type` (as `x`).
    2.  `groupBy()`: The results are then grouped again by `request_type`.
    3.  `linReg()`: For each request type group, a linear regression is performed to correlate the request count (`x`) with the average server load (`y`).

* **Result Event Data**
    The output provides separate linear regression results for each request type, helping to identify which types have the strongest impact on server performance.

| request_type | _slope | _intercept | _r2 |
| :--- | :--- | :--- | :--- |
| DELETE | <no value> | <no value> | <no value> |
| GET | -13.749... | 72.799... | 0.594... |
| POST | 16.299... | 32.700... | 0.719... |
| PUT | <no value> | <no value> | <no value> |

# callFunction()

The `callFunction()` function dynamically calls a specified aggregate function on a field over a set of events. This is particularly useful for creating dynamic dashboards where the aggregation function can be changed via a parameter.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_function` | The name of the output field. |
| **field** | string | required | The field to perform the function on. |
| **function**| string | required | The name of the function to run. The parameter name `function` can be omitted. Possible values are: `avg`, `count`, `max`, `min`, `range`, `sum`. |

***

## Example

### Call a Named Function within a Time Chart

This example demonstrates how to use `callFunction()` inside a `timeChart()` to dynamically select the aggregate function using a query parameter. This allows users to change the function (e.g., from `count` to `avg`) directly from a dashboard widget.

* **Query Example**
    ```
    timeChart(function=[callFunction(?{function=count}, field=value)])
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  The `timeChart()` function is used to plot data over time.
    3.  `callFunction()` is used as the aggregation function for the time chart. It reads the function name from a query parameter called `function`. If no parameter is provided, it defaults to `count`.
    4.  The chosen function is then applied to the `value` field for each time bucket in the chart.

* **Summary and Results**
    The query counts events in the `value` field and displays the results in a time chart. The key feature is that the aggregation function itself is dynamic. By changing the `?function` parameter in a dashboard (e.g., to `?function=avg`), the same widget can be used to display the average, count, sum, etc., without altering the underlying query.

# cidr()

The `cidr()` function filters events by checking if an IPv4 or IPv6 address in a specified field belongs to one or more subnet ranges. These subnet ranges can be provided directly in the query or loaded from a lookup file.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **column** | string | optional | The column name in a lookup file that contains the subnet list. Used with the `file` parameter. |
| **field** | string | required | The field containing the IP address to check. The parameter name `field` can be omitted. |
| **file** | string | optional | The name of the lookup file (CSV or JSON) containing the subnet list. Used with the `column` parameter. |
| **negate** | boolean | optional (deprecated) <br> default: `false` | Deprecated parameter. Use the `!cidr()` syntax for negation instead. |
| **subnet** | array of strings | optional | A list of subnet ranges to match against. |

***

## Examples

### Check if a Field Contains a Valid IP Address

This example uses `cidr()` in a creative way to validate if a field contains any valid IPv4 or IPv6 address. It does this by checking against the subnets that encompass all possible addresses (`0.0.0.0/0` for IPv4 and `::/0` for IPv6).

* **Query Example**
    ```
    case {
      cidr("address", subnet=["0.0.0.0/0", "::/0"]) | ip := address;
      *
    }
    ```

* **Step-by-Step**
    1.  The query uses a `case` statement to structure the logic.
    2.  `cidr()` checks if the value in the `address` field falls within the entire possible range of IPv4 or IPv6 addresses.
    3.  If it's a valid IP, a new field `ip` is created and assigned the value from the `address` field.

* **Summary and Results**
    This query effectively acts as an IP address validator. Events with a valid IP in the `address` field will have a new `ip` field created, while events with non-IP data in that field will be ignored by the `cidr()` condition.

---

### Filter Events Using Subnets from a File

This example demonstrates how to filter events by matching an IP address against a list of subnets loaded from an external CSV file. This is a powerful way to manage large or dynamic network groups.

* **Query Example**
    ```
    cidr(field=SRC, file="cidrfile.csv", column="cidr-block")
    ```

* **Step-by-Step**
    1.  The query starts with events that have a `SRC` field containing an IP address.
    2.  The `cidr()` function is instructed to use the lookup file `cidrfile.csv`.
    3.  It reads the list of subnets from the column named `cidr-block` within that file.
    4.  The function then keeps only the events where the IP address in the `SRC` field matches one of the subnets from the file.

* **Summary and Results**
    This query allows you to maintain network groups, allowlists, or blocklists in an external file instead of hard-coding them into your queries. The search will only be performed on events where the `SRC` IP falls within one of the ranges specified in the `cidrfile.csv`.

# coalesce()

[cite_start]The `coalesce()` function accepts a list of fields or expressions and returns the first value that is not null or empty. [cite: 458] [cite_start]It is useful for normalizing data where the same type of information might exist in fields with different names. [cite: 459, 473]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_coalesce` | [cite_start]The name of the field that will contain the selected value. [cite: 462] |
| **expressions** | list of expressions | required | The prioritized list of expressions to select from. [cite_start]The first non-null result from left to right is used. [cite: 462] |
| **ignoreEmpty** | boolean | optional <br> default: `true` | [cite_start]If set to `true`, empty strings are ignored and not selected as a value. [cite: 462] |

***

## Function Operation

[cite_start]The function evaluates a list of expressions in order from left to right and returns the value of the first expression that is not null. [cite: 458] [cite_start]By default, empty strings are also treated as undefined values and are skipped. [cite: 462, 461] [cite_start]This behavior can be changed by setting `ignoreEmpty=false`. [cite: 461] [cite_start]If a field name contains unsupported characters (like a space or a hyphen), it must be wrapped in the `getField()` function to be used correctly within `coalesce()`. [cite: 486, 487]

***

## Example

### Find the First Value in a List of Fields

This example uses `coalesce()` to find a hostname from several possible fields (`host`, `server`, `host[0].name`) and provides a default value (`"example.com"`) if none of them exist. [cite_start]This is a common way to normalize data from different sources. [cite: 473, 482]

* **Query Example**
    ```
    coalesce([host, server, host[0].name, "example.com"])
    ```

* **Input Event Data**

| host | server | host[0].name | machine |
| :--- | :--- | :--- | :--- |
| | 'crowdstrike.com' | | |
| | | 'crowdstrike.com' | |
| | | | 'clienta' |

* **Step-by-Step**
    1.  [cite_start]The query starts with events that may contain different fields for the hostname. [cite: 478]
    2.  [cite_start]`coalesce()` evaluates the fields in order for each event: `host`, `server`, and then `host[0].name`. [cite: 481]
    3.  [cite_start]It returns the value of the first field in that list that is not null or empty. [cite: 458]
    4.  [cite_start]If none of the fields have a value, it returns the string literal `"example.com"` as a default. [cite: 482]
    5.  [cite_start]The result is placed in a new field named `_coalesce`. [cite: 481]

* **Result Event Data**
    The output shows the `_coalesce` field populated with the first available hostname value.

| _coalesce | host | server | host[0].name | machine |
| :--- | :--- | :--- | :--- | :--- |
| crowdstrike.com | <no value> | crowdstrike.com | | |

# collect()

The `collect()` function gathers values from one or more fields across a set of events. The collected values can be output either as a single string with a separator or as multiple rows, each with an individual value.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **fields** | array of strings | required | The name(s) of the field(s) to collect values from. The parameter name `fields` can be omitted. |
| **limit** | integer | optional <br> default: 2000 | The maximum number of distinct values to collect. |
| **multival** | boolean | optional <br> default: `true` | If `true`, outputs a single concatenated string. If `false`, outputs multiple rows with individual values. |
| **separator** | string | optional <br> default: `\n` | The separator to use between values when `multival=true`. |

***

## Function Operation

The `collect()` function has both a value limit and a memory limit. The value limit is controlled by the `limit` parameter. The memory limit is 10 MiB when `collect()` is run as a top-level function and 1 MiB when it's used within another function like `groupBy()`. If either limit is exceeded, the function returns partial results with a warning.

> **Warning**
> Collecting the `@timestamp` field directly only works when a single timestamp exists. The recommended workaround is to create a new field (e.g., `timestamp := @timestamp`) and collect that new field instead.

***

## Examples

### Collect and Group Events by Multiple Fields

This example shows how to use `collect()` within a `groupBy()` function to see all associated IPs (`aip`) for each pair of local and remote IP addresses, alongside a distinct count of those associated IPs.

* **Query Example**
    ```
    LocalAddressIP4=* RemoteAddressIP4=* aip=*
    | groupBy([LocalAddressIP4, RemoteAddressIP4], function=([count(aip, as=aipCount, distinct=true), collect([aip])]))
    ```

* **Step-by-Step**
    1.  The query first filters for events where the `LocalAddressIP4`, `RemoteAddressIP4`, and `aip` fields all exist.
    2.  `groupBy()` then groups the events by unique pairs of `LocalAddressIP4` and `RemoteAddressIP4`.
    3.  Within each group, two aggregations are performed:
        * `count()`: A distinct count of the `aip` values is calculated and stored in the `aipCount` field.
        * `collect()`: All of the `aip` values for that group are collected into a single array field named `aip`.

* **Result Event Data**
    The output shows each unique connection pair, the count of distinct associated IPs, and the full list of those IPs.

| LocalAddressIP4 | RemoteAddressIP4 | aipCount | aip |
| :--- | :--- | :--- | :--- |
| 192.168.1.100 | 203.0.113.50 | 3 | `[10.

# communityId()

The `communityId()` function computes the Community ID, a standard for hashing network flows, which can be used to easily correlate and join related network traffic across different systems.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_community_id` | The name of the field where the output Community ID will be stored. |
| **destinationip**| string | required | The name of the field containing the destination IP address (IPv4 or IPv6). |
| **destinationport**| integer | optional | The name of the field containing the destination port (0-65535). |
| **icmpcode** | integer | optional | The name of the field for the ICMP code (0-65535). Requires `icmptype` to be set. |
| **icmptype** | integer | optional | The name of the field for the ICMP type (0-65535). Requires `icmpcode` to be set. |
| **proto** | string | required | The name of the protocol field. The value must be an IANA protocol number (0-255) or a recognized protocol keyword. |
| **seed** | integer | optional <br> default: 0 | The seed value (0-65535) used when computing the hash. |
| **sourceip** | string | required | The name of the field containing the source IP address (IPv4 or IPv6). |
| **sourceport**| string | optional | The name of the field containing the source port (0-65535). |

***

## Function Operation

This function generates a consistent hash for a network flow based on its source/destination IPs and ports, protocol, and a seed value. It follows the Community ID specification. If the protocol is ICMP, it maps the `icmptype` and `icmpcode` fields to port equivalents as described in the specification. If the function receives invalid inputs, such as an invalid IP address or a port number out of range, it will assign an empty string to the output field.

***

## Example

### Compute Community ID for Netflow Logs

This example calculates the Community ID for standard netflow events by providing the necessary 5-tuple information (source/destination IP, source/destination port, and protocol).

* **Query Example**
    ```
    communityId(
        proto=flow.protocolIdentifier,
        sourceip=flow.sourceIPv4Address,
        sourceport=flow.sourceTransportPort,
        destinationip=flow.destinationIPv4Address,
        destinationport=flow.destinationTransportPort
    )
    ```

* **Step-by-Step**
    1.  The query starts with netflow events that contain fields for protocol, source IP/port, and destination IP/port.
    2.  The `communityId()` function takes the values from these fields to compute the flow hash.
    3.  The resulting Community ID is stored in a new field named `_community_id` by default.

* **Summary and Results**
    The query generates a consistent hash for each unique network flow, which allows for easy tracking and correlation of all events related to a single flow across different systems. This is highly useful for network analysis and security monitoring.

# concat()

The `concat()` function concatenates the values of a list of fields into a single value in a new field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_concat` | The name of the output field that will store the concatenated value. |
| **field** | array of strings | required | The list of fields whose values are to be concatenated. The parameter name `field` can be omitted. |

***

## Function Operation

This function is only capable of combining the values of existing fields. It **cannot** be used to combine fields with arbitrary string literals (like adding a separator). For combining fields with strings, the `format()` function should be used instead.

***

## Example

### Correlate Inbound Email URLs with Subsequent Access Attempts

This is an advanced example where `concat()` is used as a crucial pre-processing step in a security correlation search. The goal is to detect when a user clicks a malicious URL from an email. `concat()` is used to construct a standardized, full URL from multiple fields in network traffic data so it can be matched against URLs from email security data.

* **Query Example**
    ```
    ((#repo="abnormal_security" AND #event.module="email-security") OR (#repo="corelight" AND #event.module="ids"))
    | case {
        #Vendor="corelight"
        | url.prefix := "http://"
        | url.original := concat([url.prefix, "/", client.address, "/", url.path]);
        *
      }
    | correlate(
        emailInbound: {#event.module="email-security"},
        emailAccess: {#event.module="ids" | url.original <=> emailInbound.url.original},
        sequence=true, 
        within=1h
      )
    ```

* **Input Event Data**
    Events from two sources: `abnormal_security` for emails and `corelight` for network traffic (IDS).

| @timestamp | #repo | #event.module | #Vendor | client.address |
| :--- | :--- | :--- | :--- | :--- |
| 2023-06-15T10:00:00Z | abnormal_security | email-security| abnormal | <no value> |
| 2023-06-15T10:15:30Z | corelight | ids | corelight | 10.0.1.100 |
| ... | ... | ... | ... | ... |

* **Step-by-Step**
    1.  The query first filters for relevant events from `email-security` and `ids` modules.
    2.  A `case` statement is used to process only the network traffic events (`#Vendor="corelight"`).
    3.  Inside the `case` statement, `concat()` is used to build a full URL. It combines several fields and string literals (`url.prefix`, `"/"`, `client.address`, `"/"`, `url.path`) into a new field called `url.original`. **Note**: The document shows an incorrect use of `concat` here, as it includes string literals. The correct implementation would use `format()`. Assuming the intended logic is to build the URL, this step is crucial for standardization.
    4.  The `correlate()` function then matches the `url.original` field from email events (`emailInbound`) with the newly constructed `url.original` field from network traffic events (`emailAccess`) that occur within one hour.

* **Result Event Data**
    The query outputs correlated events, showing which inbound emails led to a subsequent URL access attempt.

| @timestamp | emaillnbound.email.from.address | emaillnbound.email.to |
| :--- | :--- | :--- |
| 2023-06-15T10:00:00Z | sender@external.com | user1@company.com |
| 2023-06-15T11:00:00Z | phish@bad.com | user2@company.com |
| 2023-06-15T12:30:00Z | partner@vendor.com | user3@company.com |

# concatArray()

The `concatArray()` function concatenates the values of all fields that share the same base name and have a sequential array suffix (e.g., `field[0]`, `field[1]`, `field[2]`) into a single string in a new field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_concatArray` | The name of the output field. |
| **field** | string | required | The base name for the array fields to concatenate. The parameter name `field` can be omitted. |
| **from** | number | optional <br> default: 0 | The first array index to include in the concatenation. |
| **prefix** | string | optional | A prefix to add to the beginning of the generated string. |
| **separator**| string | optional | The separator to place between the values. |
| **suffix** | string | optional | A suffix to append to the end of the generated string. |
| **to** | number | optional | The last array index to include. If omitted, it includes all sequential fields from the `from` index. |

***

## Function Operation

This function selects all array fields beginning with the specified `field` name and a sequential index. It's crucial to note that the concatenation will **stop** at the first missing index. For example, if fields `email[0]`, `email[1]`, and `email[3]` exist, only the values from `email[0]` and `email[1]` will be concatenated. If the starting index (e.g., `email[0]`) is missing, no output field is created.

***

## Examples

### Concatenate Object Arrays into a Single String

This example first uses `objectArray:eval()` to pull a specific value (`bar`) from each object in an array of objects into a new, simple array. Then, it uses `concatArray()` to join the elements of that new array into a single string.

* **Query Example**
    ```
    "a[0].foo" := "a"
    | "a[0].bar" := "b"
    | "a[1].foo" := "c"
    | "a[1].bar" := "d"
    | objectArray:eval(array="a[]", asArray="output[]", var="x", function={output := x.bar})
    | concatArray("output")
    ```

* **Input Event Data**
    ```
    a[0].foo: a
    a[0].bar: b
    a[1].foo: c
    a[1].bar: d
    ```

* **Step-by-Step**
    1.  The query first establishes an array of objects, `a[]`.
    2.  `objectArray:eval()` iterates over `a[]` and extracts the value of the `bar` key from each object, creating a new flat array called `output[]` with the values `[b, d]`.
    3.  `concatArray("output")` then takes the `output[]` array and concatenates its elements (`b` and `d`) into a single string.

* **Result Event Data**
    The final output includes the new `_concatArray` field with the value `"bd"`.

| a[0].foo | a[0].bar | a[1].foo | a[1].bar | output[0] | output[1] | _concatArray |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| a | b | c | d | b | d | bd |

---

### Concatenate Values with a Prefix, Suffix, and Separator

This example demonstrates how to format the concatenated string by adding a prefix `[` and a suffix `]` and using a comma as the separator, effectively creating a string that looks like an array.

* **Query Example**
    ```
    concatArray(server, prefix="[", separator=",", suffix="]")
    ```

* **Input Event Data**
    ```
    server[0] := "dopey"
    server[1] := "sleepy"
    server[2] := "doc"
    server[3] := "happy"
    server[4] := "sneezy"
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `server[]` array fields.
    2.  `concatArray()` takes the values from `server[0]` through `server[4]`.
    3.  It prepends the `prefix` `[`, joins the elements with the `,` `separator`, and appends the `suffix`]`.

* **Result Event Data**
    The output is a single field `_concatArray` containing the formatted string.
    ```
    _concatArray="[dopey,sleepy,doc,happy,sneezy]"
    ```

# copyEvent()

The `copyEvent()` function creates a duplicate of an event. After this function is used, both the original and the copied event proceed to the next step in the query pipeline. It is most useful within a parsing pipeline.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **type** | string | required | The value to be assigned to the `#type` field for the copied event. The parameter name `type` can be omitted. |

***

## Function Operation

When you call `copyEvent()`, you create a second, identical version of the current event, but with a new `#type` that you specify. This allows you to process the same event in two different ways. For on-premise deployments, if you intend to copy an event to a different repository, the `ALLOW_CHANGE_REPO_ON_EVENTS` environment variable must be set to `true`.

***

## Examples

### Make a Copy of an Event to Track Arrival Time

This example demonstrates how to store an event with both its original timestamp and a separate timestamp based on when it arrived in the system.

* **Query Example**
    ```
    copyEvent("arrivaltime")
    | case { 
        #type=arrivaltime | @timestamp := now(); *;
        * | parseTimestamp(field=ts) 
      }
    ```

* **Step-by-Step**
    1.  `copyEvent("arrivaltime")`: A copy of the current event is created, and its `#type` is set to `arrivaltime`. Now, two events (the original and the copy) are in the pipeline.
    2.  `case`: This statement handles the two events differently based on their `#type`.
    3.  If `#type=arrivaltime`, the function sets the `@timestamp` of the copied event to the current time (`now()`).
    4.  The original event (which does not have `#type=arrivaltime`) has its timestamp parsed from a field named `ts`.

* **Summary and Results**
    The query results in two versions of the event being stored. This allows you to track both when an event originally occurred (original timestamp) and when it was received and processed by the system (arrival time), which is useful for monitoring ingestion delays.

---

### Copy an Event to a Different Repository

This example shows how a single parser can be used to ingest data and then distribute copies of events to different repositories.

* **Query Example**
    ```
    copyEvent("cloned_event")
    | case { 
        #type="cloned_event" | #repo := "target-repo-name"; * }
    ```

* **Step-by-Step**
    1.  `copyEvent("cloned_event")`: A copy of the current event is made, and its `#type` is set to `cloned_event`.
    2.  `case`: The statement checks the `#type` field.
    3.  If the `#type` is `cloned_event`, the function sets the `#repo` field to `"target-repo-name"`, effectively directing this copied event to be stored in the target repository. The original event is not modified and remains in its original repository.

* **Summary and Results**
    This technique is useful for creating centralized parsing logic. You can send logs from a single source to one parser, which can then decide which repositories to send the events to based on their content, allowing for flexible log routing.

# count()

The `count()` function counts the number of events. It can be used to count all events, only events containing a specific field, or the number of distinct values for a specified field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_count` | The name of the output field. |
| **distinct** | boolean | optional | If `true`, counts only the distinct values of the specified `field`. |
| **field** | string | optional | If specified, the function only counts events that contain this field. |

***

## Function Operation

The `count()` function provides a simple event count. When using `distinct=true`, the function counts the number of unique values for a given `field`. For performance reasons, when there are many distinct values, the system uses an estimation algorithm. This means the result will be a very close approximation but may not be perfectly exact. The typical accuracy is less than 2%, and for fewer than 100 distinct values, the count is exact.

***

## Examples

### Alert Query for Parser Issues

This complex example uses `count()` as part of a multi-stage query to create an alert for ongoing parser problems. The goal is to trigger an alert only when a parser has been generating a high volume of errors over a significant period.

* **Query Example**
    ```
    #type=humio #kind=logs
    | loglevel=WARN
    | class = c.h.d.ParserLimitingJob
    | "Setting reject ingest for"
    | groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])
    | timeDiff := _max - _min
    | timeDiff > 300000 and _count > 10
    ```

* **Step-by-Step**
    1.  The query first filters for specific warning logs related to parsers rejecting ingested events.
    2.  `groupBy(id, ...)` groups these errors by the parser `id`. For each parser, it performs three aggregations:
        * `count()`: Counts the total number of error events (`_count`).
        * `min(@timestamp)`: Finds the time of the first error (`_min`).
        * `max(@timestamp)`: Finds the time of the last error (`_max`).
    3.  `timeDiff := _max - _min` calculates the duration of the error period for each parser.
    4.  The final line filters these results, creating an alert only for parsers where the errors have persisted for more than 5 minutes (`timeDiff > 300000` ms) AND there have been more than 10 error events (`_count > 10`).

* **Summary and Results**
    This query is used to proactively identify and alert on significant parser issues. By using `count()` in combination with time analysis, it avoids generating alerts for minor, transient problems and focuses only on persistent, high-volume failures.

---

### Calculate a Percentage of Successful Status Codes Over Time

This example shows `count()` being used inside an embedded aggregation within `timeChart()` to calculate a success rate percentage over time.

* **Query Example**
    ```
    success := if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
          [sum(success, as=success), count(as=total)]
          | pct_successful := (success / total) * 100
          | drop([success, total])
        }
      ], span=15m, limit=100)
    ```

* **Step-by-Step**
    1.  The query first creates a temporary field `success`, assigning `1` for successful HTTP status codes (less than 500) and `0` for server errors (500 or greater).
    2.  `timeChart()` creates a time series chart.
    3.  Inside the `function` parameter, an embedded aggregation `{...}` is used:
        * `sum(success, as=success)` adds up all the `1`s, giving a total of successful requests.
        * `count(as=total)` counts all requests, regardless of success.
    4.  These two values are then used to calculate `pct_successful`.
    5.  The temporary `success` and `total` fields are dropped, leaving only the final percentage for the time chart.

* **Summary and Results**
    This query generates a time chart showing the percentage of successful requests for each customer over 15-minute intervals. It demonstrates how `count()` can serve as the denominator (the total number of events) in a rate or percentage calculation within a more complex aggregation.

# counterAsRate()

The `counterAsRate()` function calculates the rate of change for a counter field. The result is returned in a new field, with the unit being events per second by default. It's typically used within `timeChart()` or `groupBy()`.

> **Important**
> This function requires at least two data points to calculate a rate. It also expects the counter field to have values that are always increasing over time (monotonically increasing). If the counter resets (e.g., due to a server restart), no result is returned for the time bucket in which the reset occurred.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_rate` | The name of the output field. |
| **field** | string | required | The counter field from which to calculate the rate. The parameter name `field` can be omitted. |

***

## Example

### Show the Rate of a Counter Per Host

This example shows the rate of change for a counter, creating a separate time series for each host and converting the final unit from events per second to events per minute.

* **Query Example**
    ```
    timeChart(host, function=counterAsRate(counter), unit="1/sec to 1/min")
    ```

* **Summary**
    This query is used to monitor the rate of a counter on a per-host basis.
    - `timeChart(host, ...)`: This creates a time chart with a separate series for each unique value in the `host` field.
    - `function=counterAsRate(counter)`: For each host's series, it calculates the rate of change of the `counter` field.
    - `unit="1/sec to 1/min"`: This converts the default output rate from "per second" to "per minute" for easier interpretation.

# createEvents()

The `createEvents()` function generates temporary events from a list of raw strings provided in the query. It is an aggregator function, which means it discards all incoming events and outputs only the newly generated ones. This is ideal for creating sample data for testing or troubleshooting queries without needing actual data.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **rawstring** | string | required | A specification of the events to create. Each event is provided as a `@rawstring`. The parameter name `rawstring` can be omitted. |

***

## Function Operation

This function takes a list of strings and creates a new temporary event for each string. The content of each string becomes the `@rawstring` for the new event. Initially, these events have no extracted fields, so `createEvents()` is almost always piped to a parsing function like `kvParse()` or `parseJson()` to structure the raw string data into fields.

***

## Example

### Generate a Temporary Event to Test Bit Flag Extraction

This is a complex example that shows how `createEvents()` can be used to generate a test event to validate a more complex parsing and extraction query involving bit flags.

* **Query Example**
    ```
    createEvents(["flags=4"])
    | kvParse()
    | bitfield:extractFlags(
        field=flags,
        output=[
            [1, ErrorFlag],
            [2, WarningFlag]
        ]
      )
    ```

* **Step-by-Step**
    1.  `createEvents(["flags=4"])`: A single temporary event is created with the raw string `"flags=4"`.
    2.  `kvParse()`: This parser processes the `@rawstring` of the new event, identifies the key-value pair, and creates a field named `flags` with the integer value `4`.
    3.  `bitfield:extractFlags()`: This function then operates on the newly created `flags` field. It converts the value `4` to its binary representation (`100`) and extracts the state of the specified bits:
        * It checks bit 1 (the "2s" place) and creates a field `ErrorFlag` set to `false`.
        * It checks bit 2 (the "4s" place) and creates a field `WarningFlag` set to `true`.

* **Result Event Data**
    The final output is a single event with two new boolean fields, `ErrorFlag` and `WarningFlag`, representing the decoded bitmask.

| ErrorFlag | WarningFlag |
| :--- | :--- |
| false | true |

# crypto:md5()

The `crypto:md5()` function computes a cryptographic MD5 hash of one or more fields. The output is a hexadecimal string. This is useful for calculating checksums or combining multiple fields into a single, fixed-length identifier.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_md5` | The name of the output field that will store the hash. |
| **field** | array of strings | required | The name of the field or fields to hash. The parameter name `field` can be omitted. |

***

## Function Operation

The function operates on the UTF-8 encoding of the provided field values. When given multiple fields, it concatenates their values before computing the hash. If a field is missing from an event or has an empty value, it is treated as an empty string for the concatenation.

***

## Example

### MD5 Hash Multiple Fields

This example shows how to hash multiple fields together to create a single, reproducible MD5 sum. This can be an effective method for creating a unique ID for a specific set of field values, which can then be used for lookups or joins.

* **Query Example**
    ```
    crypto:md5([a,b,c])
    ```

* **Step-by-Step**
    1.  The query starts with events that contain the fields `a`, `b`, and `c`.
    2.  The `crypto:md5()` function takes the values from these three fields.
    3.  It concatenates the values together into a single string.
    4.  It then computes the MD5 hash of the concatenated string and returns the result in a new field named `_md5`.

* **Summary and Results**
    The query creates a single MD5 hash from the combined values of the supplied fields. This is a reproducible process, meaning the same set of input values will always produce the same MD5 hash. This makes it a useful technique for creating unique identifiers for specific event types or for creating a common key to join two different datasets.

# table()

The `table()` function is an aggregate function that displays query results in a table, allowing you to specify which fields to include and how to sort them.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **fields** | array of strings | required | The names of the fields to select for the table columns. The parameter name `fields` can be omitted. |
| **limit** | number | optional <br> default: 200 | The maximum number of rows to include in the result. |
| **order** | string | optional <br> default: `desc` | The order to sort in. Can be `asc` (ascending) or `desc` (descending). |
| **reverse** | boolean | optional (deprecated) | Deprecated parameter. Use `order` instead. |
| **sortby** | array of strings | optional <br> default: `@timestamp`| The names of the fields to sort the results by. |
| **type** | array of strings | optional <br> default: `number` | The data type of the fields to sort by. Can be `any`, `hex`, `number`, or `string`. |

***

## Function Operation

The `table()` function aggregates events to create a tabular view. It performs three main actions:
1.  **Sorts Columns**: Arranges the columns in the table in the order specified in the `fields` parameter.
2.  **Limits Rows**: Restricts the number of events (rows) returned based on the `limit` parameter.
3.  **Sorts Results**: Sorts the rows of the table according to the `sortby` and `order` parameters.

For large data exports, the `select()` function is recommended as an alternative, as it provides similar output without the row limits and sorting constraints of `table()`.

***

## Example

### Calculate and Display Query Costs by User and Repository

This complex example demonstrates how `table()` can be used as the final step in a data analysis pipeline to present aggregated and sorted results in a clean, readable format.

* **Query Example**
    ```
    #type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"
    | repoUser := format("%s/%s", field=[dataspace, initiatingUser])
    | top(repoUser, sum(deltaTotalCost), as=cost)
    | table([cost, repoUser], sortby=cost)
    ```

* **Input Event Data**
    Internal logs from Humio that contain information about query costs.

| #type | #kind | class | message |
| :--- | :--- | :--- | :--- |
| humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query |
| humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query |
| ... | ... | ... | ... |

* **Step-by-Step**
    1.  The query first filters for specific internal logs related to high-cost queries.
    2.  `format()`: It then combines the `dataspace` (repository) and `initiatingUser` fields into a single, new field called `repoUser` (e.g., `"development/jane.smith"`).
    3.  `top()`: This function finds the most common `repoUser` values and, for each one, calculates the `sum()` of their query costs, storing it in a new field named `cost`.
    4.  `table()`: Finally, `table()` takes this aggregated data and displays it. It creates a table with two columns, `cost` and `repoUser`, and sorts the rows by the `cost` field in descending order (the default).

* **Result Event Data**
    The output is a sorted table that clearly shows the total query cost for each user and repository combination, with the highest costs listed first.

| cost | repoUser |
| :--- | :--- |
| 3200 | development/jane.smith |
| 2300 | production/john.doe |
| 1500 | staging/bob.wilson |

# tail()

The `tail()` function retrieves a specified number of the most recent events. It sorts events by `@timestamp` or `@ingesttimestamp` to determine which are the newest.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **limit** | number | optional <br> default: 200 | The maximum number of events to return. The parameter name `limit` can be omitted. |

***

## Function Operation

The `tail()` function sorts events in descending order based on their timestamp to find the most recent ones. It defaults to using the `@timestamp` field but will use `@ingesttimestamp` if `@timestamp` is not available. If neither timestamp field is present, the query will report an error.

***

## Example

### Deduplicate Events by a Specific Field

This example uses `tail()` inside a `groupBy()` function to effectively deduplicate events based on a field. For each unique value in the specified field, it keeps only the single most recent event.

* **Query Example**
    ```
    groupBy(field, function=tail(1))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  `groupBy(field, ...)`: This function groups all events by the unique values found in the specified `field`.
    3.  `function=tail(1)`: Within each of those groups, `tail(1)` is executed. This sorts the events in that group by timestamp and keeps only the single most recent one.

* **Summary and Results**
    This query is a powerful technique for creating a unique list of events based on a particular field, where "unique" is defined as the last seen event for that field's value. This is useful for getting the current state of entities when you have many historical events for each one.

# test()

The `test()` function evaluates an arbitrary expression as a boolean value (`true` or `false`) and filters events, keeping only those for which the expression returns `true`. It allows for complex comparisons between multiple fields and values.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **expression** | expression | required | The expression to evaluate. The parameter name `expression` can be omitted. |

***

## Function Operation

This function provides a flexible way to filter events based on custom logic. It's important to remember that within a `test()` expression, unquoted strings are interpreted as field names, while quoted strings are treated as literal values. For example, `test(myField == "myValue")` compares the field `myField` to the string "myValue", whereas `test(myField == myOtherField)` compares the value of the `myField` field to the value of the `myOtherField` field.

***

## Examples

### Check if Fields Have the Same Length

This example uses `test()` with the `length()` function nested inside it to find events where the values in two different fields have the same character length.

* **Query Example**
    ```
    test(length(userid) == length(method))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  For each event, the `test()` function evaluates the expression `length(userid) == length(method)`.
    3.  It calculates the length of the `userid` field and the length of the `method` field.
    4.  It then compares the two lengths. If they are equal, the expression returns `true`, and the event is kept in the result set.

* **Summary and Results**
    This query returns all events where the `userid` and `method` fields have the same length (e.g., a `userid` of "chad" and a `method` of "POST" would both have a length of 4 and would match).

---

### Filter for Events Within a Relative Time Window

This example demonstrates a more complex use case, filtering for events that occurred within the first 30 days of the query's start time.

* **Query Example**
    ```
    test(@timestamp < (start() + (30*24*60*60*1000)))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  The `test()` function evaluates the time comparison for each event.
    3.  `start()`: This function returns the start time of the query search window in milliseconds.
    4.  `(30*24*60*60*1000)`: This is a manual calculation of the number of milliseconds in 30 days.
    5.  The expression `(start() + 30_days_in_ms)` calculates a cutoff timestamp that is exactly 30 days after the query began.
    6.  The event's `@timestamp` is then compared to this cutoff. If the event's timestamp is earlier than the cutoff, the expression returns `true`, and the event is kept.

* **Summary and Results**
    This query is a practical way to filter for events that occurred within a specific time window relative to the start of the search, allowing for dynamic and precise time-based analysis.

# time:dayOfMonth()

[cite_start]The `time:dayOfMonth()` function gets the day of the month from a timestamp field[cite: 7493].

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_dayOfMonth` | [cite_start]The name of the output field[cite: 7494]. |
| **field** | string | optional <br> default: `@timestamp` | [cite_start]The name of the input field containing the timestamp[cite: 7494]. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. [cite_start]This will override the query's default timezone[cite: 7494]. |
| **timezoneField** | string | optional | The name of a field that contains the timezone to use. [cite_start]This is ignored if the `timezone` parameter is also set[cite: 7494]. |

***

## Examples

There are no examples provided in the document for this function.

# time:dayOfWeek()

The `time:dayOfWeek()` function gets the day of the week from a timestamp field, returning a number from 1 (Monday) to 7 (Sunday).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_dayOfWeek` | The name of the output field. |
| **field** | string | required <br> default: `@timestamp` | The name of the input field containing the timestamp. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:dayOfWeekName()

The `time:dayOfWeekName()` function gets the English display name of the day of the week (e.g., "Monday") from a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_dayOfWeekName` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:dayOfYear()

The `time:dayOfYear()` function gets the day of the year from a timestamp field, returning a number from 1 to 365 (or 366 in a leap year).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_dayOfYear` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:hour()

The `time:hour()` function gets the hour of the day (0-23) from a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_hour` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Example

### Summarize Data Events by Hour

This example uses `time:hour()` to extract the hour from the ingest timestamp of events and then uses `groupBy()` to count how many events occurred in each hour.

* **Query Example**
    ```
    hr := time:hour(field="@ingesttimestamp")
    | groupBy(hr)
    ```

* **Step-by-Step**
    1.  `hr := time:hour(field="@ingesttimestamp")`: This step extracts the hour (0-23) from the `@ingesttimestamp` of each event and stores it in a new field named `hr`.
    2.  `groupBy(hr)`: The events are then grouped by the `hr` field, and a count of events for each hour is calculated.

* **Summary and Results**
    The query is used to get a count of data events per hour. The final result is a table showing each hour of the day and the total number of events that were ingested during that hour, which can then be plotted on a bar chart.

# time:millisecond()

The `time:millisecond()` function gets the millisecond component of a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_millisecond` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:minute()

The `time:minute()` function gets the minute component of a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_minute` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:month()

The `time:month()` function gets the month from a timestamp field, returning a number from 1 to 12.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_month` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:monthName()

The `time:monthName()` function gets the English name of the month (e.g., "January") from a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_monthName` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:second()

The `time:second()` function gets the second component of a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_second` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. If not specified, the query's offset will be used. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# time:weekOfYear()

The `time:weekOfYear()` function gets the week number of a timestamp, returning a value from 1 to 53.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_weekOfYear` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Function Operation

This function calculates the week number according to the ISO 8601 standard, where weeks start on a Monday and are assigned to the year that contains the Thursday of that week.

***

## Examples

There are no examples provided in the document for this function.

# time:year()

The `time:year()` function gets the year of a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_year` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. If not specified, the query's offset will be used. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.

# timeChart()

The `timeChart()` function is a powerful aggregation tool that groups events into time-based buckets and creates a time-series chart. It is an extension of `groupBy()` designed specifically for visualizing data over time.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **buckets** | number | optional | Defines the number of time buckets (1 to 1500) to divide the query interval into. |
| **function**| array of aggregate functions | optional <br> default: `count()` | The aggregate function(s) to perform on each time bucket. |
| **limit** | number | optional <br> default: 10 | The maximum number of series to produce (up to 500). Prioritizes series with the highest numerical aggregate values. |
| **minSpan** | string (Relative Time) | optional | Sets the minimum allowed time span for each bucket (e.g., `5h`). |
| **series** | string | optional | A field whose unique values will each become a separate series (line) on the chart. |
| **span** | string (Relative Time) | optional <br> default: `auto` | Defines a fixed time span for each bucket (e.g., `1h`, `15m`). |
| **timezone**| string | optional | The timezone to use for bucketing (e.g., `UTC`, `+02:00`). |
| **unit** | string | optional | Applies a unit conversion to the output (e.g., `bytes/bucket to Mbytes/hour`). |

***

## Function Operation

The `timeChart()` function is the primary tool for creating time-series visualizations. It groups events into time intervals (`buckets`) and then applies one or more aggregate functions to the events in each bucket.

> **Important**
> You cannot use anchored time units (like `/d` for the start of a day) when defining the `span` length in `timeChart()`.

### Series Selection
When using the `series` and `limit` parameters, `timeChart()` prioritizes which series to display based on the numerical values produced by the aggregate `function`. It does not consider the series names themselves. When multiple functions are used, it considers the combined numerical output to determine the top series.

***

## Examples

### Calculate a Percentage of Successful Status Codes

This advanced example demonstrates how to perform a multi-step calculation directly within the `function` parameter of `timeChart()` to produce a success rate percentage.

* **Query Example**
    ```
    success := if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
          [sum(success, as=success), count(as=total)]
          | pct_successful := (success / total) * 100
          | drop([success, total])
        }
      ], span=15m, limit=100)
    ```

* **Step-by-Step**
    1.  An `if()` statement first creates a binary `success` field: `1` for successful requests (status < 500) and `0` for server errors.
    2.  `timeChart()` then creates a series for each `customer`.
    3.  Within its embedded aggregation `{...}` for each time bucket:
        * It calculates the `sum()` of the `success` field (total successes) and the `count()` of all events (total requests).
        * It uses these two temporary values to calculate the `pct_successful`.
        * Finally, it uses `drop()` to remove the temporary `success` and `total` fields, leaving only the final percentage.

---

### Compute a Cumulative Aggregation Across Buckets

This example shows how `timeChart()` can be used to prepare data for a sequence function like `accumulate()`, allowing you to calculate a running total over time.

* **Query Example**
    ```
    timeChart(span=1000ms, function=sum(value))
    | accumulate(sum(_sum), as=_accumulated_sum)
    ```

* **Input Event Data**

| @timestamp | value |
| :--- | :--- |
| 1451606301001 | 5 |
| 1451606301500 | 6 |
| 1451606301701 | 1 |
| 1451606302001 | 2 |
| 1451606302201 | 6 |

* **Step-by-Step**
    1.  `timeChart(span=1000ms, function=sum(value))`: This function groups data into 1-second buckets and calculates the `sum()` of the `value` field for each bucket, creating a `_sum` field.
    2.  `accumulate(...)`: The `accumulate()` function then takes these time-bucketed results and calculates a running total of the `_sum` field across the buckets.

* **Result Event Data**
    The output is a time series that includes both the sum for each individual bucket (`_sum`) and the cumulative running total up to that point (`_accumulated_sum`).

| _bucket | _sum | _accumulated_sum |
| :--- | :-- | :--- |
| 1451606300000 | 0 | 0 |
| 1451606301000 | 12 | 12 |
| 1451606302000 | 8 | 20 |
| 1451606303000 | 0 | 20 |

---

### Find Top N Series Using Multiple Functions

This example illustrates how the `limit` parameter selects the top series based on the combined numerical output of multiple functions, not just the single highest value.

* **Query Example**
    ```
    timeChart(series=key, function=[max(value), {foo := value % 41 | selectLast(foo)}], limit=2)
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 42 |
| b | 41 |
| c | 40 |

* **Step-by-Step**
    1.  `timeChart()` creates a series for each `key` (`a`, `b`, `c`).
    2.  For each series, it calculates two values:
        * `max(value)`: The maximum value.
        * `foo := value % 41 | selectLast(foo)`: A second value based on a modulus calculation.
    3.  The top 2 series are selected based on the *sum* of these calculated values for each key:
        * **'a'**: `max` is 42, `foo` is `42 % 41 = 1`. Total score = 43.
        * **'b'**: `max` is 41, `foo` is `41 % 41 = 0`. Total score = 41.
        * **'c'**: `max` is 40, `foo` is `40 % 41 = 40`. Total score = 80.
    4.  Because `limit=2`, the function keeps the series with the highest total scores: 'c' (80) and 'a' (43).

* **Result Event Data**
    The output shows only the data for series 'a' and 'c', demonstrating how the combined metrics determined the "top" series.

| _bucket | key | _max | foo |
| :--- | :-: | :-- | :-: |
| 1747109790000 | a | 42 | 1 |
| 1747109790000 | c | 40 | 40 |

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

# top()

The `top()` function is an efficient way to find the most common values of a field in a set of events. It is a more powerful and concise alternative to running `groupBy()`, `count()`, and `sort()` together. It can also find the top values based on the `sum` or `max` of another field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_count` or `_sum` | The name of the output field for the aggregated value. |
| **error** | number | optional <br> default: 5 | The error threshold in percentage. A warning is shown if the approximative algorithm's precision is lower than this. |
| **field** | array of strings | required | The field(s) on which to group and find the top values. |
| **limit** | number | optional <br> default: 10 | The number of top results to return. |
| **max** | string | optional | Changes the aggregation function from `count()` to find the `max()` of the specified field. |
| **percent** | boolean | optional <br> default: `false` | If `true`, adds a `_percent` column showing the percentage of the total count. |
| **rest** | string | optional | If specified, adds an extra row that aggregates all other values not included in the top results. The string provided becomes the label for this row. |
| **sum** | string | optional | Changes the aggregation function from `count()` to find the `sum()` of the specified field. |

***

## Function Operation

The `top()` function uses an approximative algorithm to efficiently compute the most frequent items, which is guaranteed to be exact for a certain number of unique items but may be an estimate for very high cardinality fields. It only returns events that are guaranteed to be in the top results (no false positives).

***

## Examples

### Calculate Query Costs by User and Repository

This example shows how to use `top()` with the `sum` parameter to find which users have the highest total query costs, rather than just the highest number of queries.

* **Query Example**
    ```
    #type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"
    | repoUser := format("%s/%s", field=[dataspace, initiatingUser])
    | top(repoUser, sum(deltaTotalCost), as=cost)
    | table([cost, repoUser], sortby=cost)
    ```

* **Step-by-Step**
    1.  The query first filters for internal logs that report on high-cost queries.
    2.  `format()`: The `dataspace` (repository) and `initiatingUser` fields are combined into a single `repoUser` field for easier grouping.
    3.  `top(repoUser, sum(deltaTotalCost), as=cost)`: This is the key step. It groups by the combined `repoUser` field. Instead of counting, it calculates the `sum()` of the `deltaTotalCost` for each user and renames the resulting sum field to `cost`. It then returns the top users with the highest total cost.
    4.  `table()`: The final results are displayed in a clean, sorted table.

* **Result Event Data**
    The output is a table showing the total query cost for each user/repository pair, sorted by the highest cost.

| cost | repoUser |
| :--- | :--- |
| 3200 | development/jane.smith |
| 2300 | production/john.doe |
| 1500 | staging/bob.wilson |

---

### Extract the Top Most Viewed Pages and Group the Rest

This example demonstrates how to extract a value with `regex`, find the top 12 most frequent values, and group all other less frequent values into a single "others" category.

* **Query Example**
    ```
    regex(regex="/.*/(?<url_page>\\S+\\.page)", field=url)
    | top(url_page, limit=12, rest=others)
    ```
* **Step-by-Step**
    1.  `regex()`: A regular expression is used to extract the page name (ending in `.page`) from a `url` field and store it in a new field called `url_page`.
    2.  `top(url_page, limit=12, rest=others)`: The `top()` function then finds the 12 most frequent values in the `url_page` field. The `rest=others` parameter instructs the function to create an additional row labeled "others" that contains the aggregated count of all pages that didn't make it into the top 12.

* **Result Event Data**
    The output is a table of the top 12 most viewed pages and their counts, plus a final row for all other page views combined.

| url_page | _count |
| :--- | :--- |
| home.page | 51 |
| index.page | 21 |
| ... | ... |
| others | 153 |

---

### Create a Frequency Count With Formatted Links

This example shows how `top()` can be used to generate data that is then piped into another function (`format()`) for enhanced presentation, such as creating clickable links in the UI.

* **Query Example**
    ```
    top(repo)
    | format("[Link](https://example.com/%s)", field=repo, as=link)
    ```

* **Input Event Data**
    A series of events with a `repo` field indicating activity in different code repositories.

| @timestamp | repo |
| :--- | :--- |
| 2023-06-15T10:00:00Z | "frontend-app" |
| 2023-06-15T10:05:00Z | "backend-api" |
| 2023-06-15T10:10:00Z | "frontend-app" |
| ... | ... |

* **Step-by-Step**
    1.  `top(repo)`: This function finds the most frequently occurring repository names and provides a count for each, sorted in descending order.
    2.  `format(...)`: The results are then piped to `format()`. For each repository name, it constructs a markdown link string, inserting the `repo` field value into the URL. This new link is stored in a field named `link`.

* **Result Event Data**
    The output is an interactive table showing each repository, its activity count, and a clickable link to that repository.

| repo | _count | link |
| :--- | :--- | :--- |
| frontend-app | 4 | `[Link](https://example.com/frontend-app)` |
| backend-api | 2 | `[Link](https://example.com/backend-api)` |
| ... | ... | ... |

# transpose()

The `transpose()` function transforms table-like data by converting columns into rows, effectively switching the orientation of the data. This is a great way to reorganize data into a more readable format or prepare it for specific visualizations.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **column** | string | optional <br> default: `"column"` | The name of the field that will contain the original column headers. |
| **header** | string | optional | The name of a field whose values should be used as the new column headers for the transposed table. |
| **limit** | integer | optional <br> default: 5 | The maximum number of rows to transpose (from 1 to 1,000). |
| **pivot** | string | optional | A shorthand to use the same field for both the `header` and `column` values. |

***

## Examples

### Create a Pivot Table

This complex example shows how to create a pivot table to summarize the last time each user performed a specific action, using a nested `groupBy` to prepare the data and `transpose` to format it.

* **Query Example**
    ```
    groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})
    | transpose(header=type)
    | drop(column)
    ```

* **Step-by-Step**
    1.  **Nested `groupBy()`**: The query first performs a nested aggregation. The inner `groupBy` finds the maximum (i.e., latest) timestamp for each `actor.user.id`. The outer `groupBy` then groups those results by `type` (the action performed). This creates a flat table with columns: `type`, `actor.user.id`, and `_max` (the timestamp).
    2.  **`transpose(header=type)`**: The `transpose()` function then pivots this table. It uses the unique values from the `type` column (e.g., "alert.create", "dashboard.create") as the new column headers. The values in the table become the `actor.user.id` and `_max` timestamp.
    3.  **`drop(column)`**: Transposing creates a leftover metadata field named `column` by default. This final step removes that field to clean up the output.

* **Result Event Data**
    The final output is a pivot table where the columns are the event types, and the rows show the last user to perform that action and the timestamp of when it occurred.

| alert.clear-error | alert.create | alert.update |
| :--- | :--- | :--- |
| 1700546666592 | 1699004139419 | 1700546666676 |
| 007WGPBX9YbvZbKOrBMd5fgH| 007WGPBX9YbvZbKOrBMd5fgH| 007WGPBX9YbvZbKOrBMd5fgH|

---

### Search Across Multiple Structured Fields

This example demonstrates an advanced technique where `transpose()` is used to enable searching for a value across all fields of an event simultaneously.

* **Query Example**
    ```
    groupBy(@id, function=transpose())
    | row[1] = /httpd/
    | groupBy(column)
    ```

* **Input Event Data**
    Standard log events with multiple fields.

| host | @rawstring |
| :--- | :--- |
| MAIL01 | `... MAIL01 httpd[61789]: ...` |
| LON-SRV01| `... LON-SRV01 httpd[60123]: ...` |

* **Step-by-Step**
    1.  `groupBy(@id, function=transpose())`: This step transposes each event individually. For each event, it creates a set of new events where each new event has two fields: `column` (containing the original field name, e.g., "host", "app") and `row[1]` (containing the original field's value, e.g., "MAIL01", "httpd[61789]:").
    2.  `row[1] = /httpd/`: Now that all values are in a single field (`row[1]`), a simple filter can search across all of them. This line keeps only the transposed rows where the value contains "httpd".
    3.  `groupBy(column)`: Finally, the query groups by the `column` field. This produces a final count of which original fields contained the search term "httpd".

* **Result Event Data**
    The final output is a table listing the original field names that contained the value "httpd" and a count of how many times it appeared in each.

| column | _count |
| :--- | :--- |
| @rawstring | 5 |
| app | 5 |

# unit:convert()

The `unit:convert()` function converts numerical values between different units, such as bytes to megabytes or milliseconds to seconds.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | The name of the output field. Defaults to overwriting the input field. |
| **binary** | boolean | optional <br> default: `false` | If `true`, standard prefixes (K, M, G, etc.) are treated as powers of 1024 (binary) instead of 1000 (decimal). |
| **field** | string | required | The name of the input field to convert. The parameter name `field` can be omitted. |
| **from** | string | optional | If the field contains a unitless number, this specifies the unit to assume for the conversion. |
| **keepUnit**| boolean | optional <br> default: `false` | If `true`, the new unit will be appended to the output value. |
| **to** | string | optional | The unit to convert the value to. If omitted, the function parses the value into its base unit (e.g., "5 MB" becomes `5000000`). |
| **unit** | string | optional | If specified, the field's value must end with this exact unit. This helps resolve ambiguity if a unit name starts with a reserved prefix (e.g., 'h' for hecto- vs. 'hits'). |

***

## Function Operation

This function can parse values that have units attached (e.g., `"5 MB"`) or unitless numbers if a `from` unit is specified. It supports a wide range of SI (decimal, powers of 1000) and binary (powers of 1024) prefixes.
* **Decimal prefixes**: `k`, `M`, `G`, `T`, etc.
* **Binary prefixes**: `Ki`, `Mi`, `Gi`, `Ti`, etc.

The `binary=true` flag is useful when data is labeled with decimal prefixes but represents binary quantities (e.g., a field has the value `"5 kB"` but it should be treated as 5 Kibibytes). The function can also normalize rates (like `hits/min`) to a per-second basis.

***

## Example

### Convert File Size and Transfer Time Units

This example demonstrates how to convert multiple fields to more readable units for analysis. It converts file sizes from bytes to megabytes and transfer times from milliseconds to seconds.

* **Query Example**
    ```
    unit:convert(field=file_size, from="B", to="MB")
    | unit:convert(field=transfer_time, from="ms", to="s")
    | table([file_name, file_size, transfer_time])
    ```

* **Input Event Data**

| file_name | file_size | transfer_time |
| :--- | :--- | :--- |
| doc1.pdf | 1048576 | 3500 |
| img1.jpg | 2097152 | 4200 |
| video1.mp4| 5242880 | 12000 |

* **Step-by-Step**
    1.  The query starts with events containing file size in bytes and transfer time in milliseconds.
    2.  The first `unit:convert()` call reads the `file_size` field, treats the number as Bytes (`from="B"`), and converts it to Megabytes (`to="MB"`).
    3.  The second `unit:convert()` call reads the `transfer_time` field, treats it as milliseconds (`from="ms"`), and converts it to seconds (`to="s"`).
    4.  Finally, `table()` displays the newly converted, human-readable values.

* **Result Event Data**
    The output is a table with standardized units, making it easy to compare the values.

| file_name | file_size | transfer_time |
| :--- | :--- | :--- |
| doc1.pdf | 1.0 | 3.5 |
| img1.jpg | 2.0 | 4.2 |
| video1.mp4| 5.0 | 12.0 |

# upper()

The `upper()` function converts text from an event field to uppercase letters. It can use either the system's default locale or a specific locale if provided.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_upper` | The name of the output field. |
| **field** | string | required | The name of the input field with the value to convert to uppercase. The parameter name `field` can be omitted. |
| **locale** | string | optional <br> default: system locale | The locale to use for conversion, specified as an ISO 639 language and ISO 3166 country (e.g., `en`, `en_US`). |

***

## Function Operation

This function reads the string value from the specified `field` and outputs an uppercase version. Specifying the correct `locale` is important for languages with non-Latin alphabets to ensure correct character conversion. For example, you can specify `en_US` for US English or `en_UK` for UK English.

***

## Examples

### Correlate Two Scheduled Task Events

This complex example uses `upper()` to standardize task names before correlating task registration and deletion events. This ensures that a match will be found even if the casing of the task name is different in the two event types.

* **Query Example**
    ```
    correlate(
      ScheduledTaskRegistered: {
        #repo="base_sensor" #event_simpleName=ScheduledTaskRegistered
        | upper(field=TaskName, as=scheduledTaskName)
      },
      ScheduledTaskDeleted: {
        #repo="base_sensor" #event_simpleName=ScheduledTaskDeleted
        | upper(field=TaskName, as=scheduledTaskName)
        | aid <=> ScheduledTaskRegistered.aid
        | scheduledTaskName <=> ScheduledTaskRegistered.scheduledTaskName
      },
      sequence=false, within=5m)
    ```

* **Input Event Data**

| #event_simpleName | aid | TaskName |
| :--- | :-- | :--- |
| ScheduledTaskRegistered | aid123| backup_task |
| ScheduledTaskDeleted | aid123| backup_task |
| ScheduledTaskRegistered | aid456| cleanup_task|
| ScheduledTaskDeleted | aid456| cleanup_task|

* **Step-by-Step**
    1.  The `correlate()` function defines two subqueries, `ScheduledTaskRegistered` and `ScheduledTaskDeleted`.
    2.  In both subqueries, `upper(field=TaskName, as=scheduledTaskName)` is used to convert the `TaskName` to uppercase and store it in a new field, `scheduledTaskName`.
    3.  The correlation then matches events from both subqueries where the `aid` and the standardized `scheduledTaskName` are identical, and the events occurred within a 5-minute window.

* **Result Event Data**
    The output shows correlated events, demonstrating the complete lifecycle of a task within the specified timeframe.

| @timestamp | ScheduledTaskRegistered.aid | ScheduledTaskRegistered.scheduledTaskName |
| :--- | :--- | :--- |
| 2023-06-15T10:00:00Z | aid123 | BACKUP_TASK |
| 2023-06-15T10:05:00Z | aid456 | CLEANUP_TASK |

---

### Standardize and Combine Fields for Analysis

This example demonstrates a full data preparation pipeline where `upper()` and `lower()` are used to standardize fields, which are then combined and aggregated to find the most common patterns.

* **Query Example**
    ```
    lower(#severity, as=severity)
    | upper(#category, as=category)
    | concat([severity, category], as=test)
    | top(test)
    ```

* **Step-by-Step**
    1.  `lower(#severity, as=severity)`: Converts the `#severity` field to all lowercase.
    2.  `upper(#category, as=category)`: Converts the `#category` field to all uppercase.
    3.  `concat([severity, category], as=test)`: Combines the newly standardized `severity` and `category` fields into a single field named `test`.
    4.  `top(test)`: Finds the most common combinations of severity and category and provides a count for each.

* **Result Event Data**
    The final output is a table showing the most frequent combinations of severity and category, with standardized casing for easy grouping and analysis.

| test | _count |
| :--- | :--- |
| infoALERT | 90005 |
| infoFILTERALERT | 36640 |
| errorALERT | 17256 |
| warningGRAPHQL | 14240 |
| ... | ... |

# urlDecode()

The `urlDecode()` function decodes a URL-encoded string field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_urldecode` | The name of the output field. |
| **field** | string | required | The name of the input field with the value to be URL-decoded. The parameter name `field` can be omitted. |

***

## Example

### URL-Decode a Field into a Custom Output Field

This example takes a URL-encoded string from a field named `Bar` and decodes it, storing the result in a new field named `Foo`.

* **Query Example**
    ```
    urlDecode("Bar", as="Foo")
    ```

* **Input Event Data**
    An event with a field `Bar` containing the URL-encoded string:
    `Bar=https%3A%2F%2Fwww.example.com`

* **Result Event Data**
    The query creates a new field named `Foo` with the decoded value.
    `Foo="https://www.example.com"`

# urlEncode()

The `urlEncode()` function URL-encodes the contents of a string field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_urlencode` | The name of the output field. |
| **field** | string | required | The name of the input field with the value to be URL-encoded. The parameter name `field` can be omitted. |
| **type**| string | optional | The type of encoding to perform. |

***

## Example

### URL-Encode a Field into a Custom Output Field

This example takes a URL string from a field named `Bar` and encodes it, storing the result in a new field named `Foo`.

* **Query Example**
    ```
    urlEncode("Bar", as="Foo")
    ```

* **Input Event Data**
    An event with a field `Bar` containing the URL string:
    `Bar=https://www.example.com`

* **Result Event Data**
    The query creates a new field named `Foo` with the URL-encoded value.
    `Foo="https%3A%2F%2Fwww.example.com"`

# wildcard()

The `wildcard()` function performs a wildcard (glob) pattern search on event fields, making it easier to find matches without using complex regular expressions.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | array of strings | optional | The field or fields to search within. If not specified, all fields of the original event are searched. |
| **ignoreCase**| boolean | optional <br> default: `false` | If `true`, the search will be case-insensitive. |
| **pattern** | string | required | The wildcard pattern to search for. The parameter name `pattern` can be omitted. |

***

## Function Operation

The behavior of `wildcard()` changes based on its parameters:
* **Case-sensitive, specific field**: `wildcard(field=log, pattern="*error*")` is equivalent to a standard wildcard search: `log="*error*"`.
* **Case-sensitive, all fields**: `wildcard(pattern="*error*")` performs a free-text search for the pattern across the entire event.
* **Case-insensitive**: When `ignoreCase=true`, the search is equivalent to a case-insensitive regular expression. For performance reasons, only set `ignoreCase=true` when necessary.

The function can be negated using `!wildcard(...)` to find all events that do *not* match the pattern.

***

## Examples

### Drop Events Based on Specific Field Values or Patterns

This example demonstrates using `wildcard()` as one of several conditions within a `case` statement to filter events based on complex logic.

* **Query Example**
    ```
    case {
      fielda = "badresult" | dropEvent();
      fieldb = "badresult" | dropEvent();
      wildcard("badip*", field=[fieldc, fieldd]) | dropEvent();
      *
    }
    ```

* **Step-by-Step**
    1.  A `case` statement is used to evaluate multiple conditions.
    2.  The first two lines drop events if `fielda` or `fieldb` contain an exact "badresult".
    3.  The third condition uses `wildcard("badip*", field=[fieldc, fieldd])`. This checks if the value in *either* `fieldc` or `fieldd` starts with the string "badip". If a match is found in either field, the event is dropped.
    4.  The final `*` ensures all other events are kept.

* **Summary and Results**
    This query uses `wildcard()` to apply a pattern match across multiple fields as part of a larger filtering strategy, allowing for flexible and powerful event processing.

---

### Find and Group Fields With a Specific Pattern

This example shows a common workflow: using `wildcard()` to find all relevant events and then piping the results to `groupBy()` for aggregation.

* **Query Example**
    ```
    wildcard(field=class, pattern="*S3Bucket*", ignoreCase=true)
    | groupBy(class)
    ```

* **Step-by-Step**
    1.  `wildcard(...)`: The query first searches the `class` field for any value that contains the string "S3Bucket", ignoring case.
    2.  `groupBy(class)`: The events that match are then piped to `groupBy()`, which groups them by the unique `class` names and provides a count for each.

* **Result Event Data**
    The output is an aggregated count of all events that have "S3Bucket" anywhere in their `class` field, regardless of capitalization.

| class | _count |
| :--- | :--- |
| c.h.b.s.S3BucketStorageCleaningJob| 197 |
| c.h.b.s.S3BucketStorageFileUploader| 2329 |
| c.h.b.s.S3BucketStorageUploadJob | 3869 |

---

### Search Multiple Fields for a Specific Pattern

This example shows how to use the `field` parameter to search for a pattern in a list of specified fields.

* **Query Example**
    ```
    wildcard(field=[plant, animal], pattern="*horse*", ignoreCase=false)
    ```
* **Input Event Data**
    Events that contain either a `plant` or an `animal` field.

| field | value |
| :--- | :--- |
| animal | "horse" |
| animal | "dancing with horses" |
| plant | "daisy" |
| plant | "horseflower" |

* **Step-by-Step**
    1.  The query starts with the source events.
    2.  `wildcard()` searches for the pattern `*horse*` (the string "horse" anywhere in the value).
    3.  The `field=[plant, animal]` parameter tells it to only check in the `plant` and `animal` fields.
    4.  The search is case-sensitive because `ignoreCase` is false (the default).

* **Summary and Results**
    The query returns a list of events where either the `animal` field or the `plant` field contains the string "horse". This is equivalent to running `animal="*horse*" OR plant="*horse*"`.

# window()

The `window()` function computes an aggregate (like an average or sum) over a sliding window of data. It can only be used as the `function` argument within a `timeChart()` or `bucket()` function.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **buckets** | integer | optional | Defines the size of the sliding window in terms of the number of `timeChart` buckets to include in the aggregate. |
| **function**| array of aggregate functions | optional <br> default: `count()` | The aggregate function(s) to perform on each window. |
| **span** | string (Relative Time) | optional | Defines the width of the sliding window as a fixed time duration (e.g., `15m`, `1h`). |

**Note**: You must define exactly one of either `span` or `buckets`.

***

## Function Operation

The `window()` function calculates a running aggregate. For each time bucket created by the parent `timeChart()` function, `window()` looks back over a specified period (defined by `span` or `buckets`) and computes an aggregate value from all the events in that window. This is useful for smoothing out data or calculating moving averages. If the query's overall time interval is less than the specified window `span`, no result will be computed.

***

## Examples

### Find Outliers Using a Dynamic Threshold

This complex example uses `window()` to calculate a running average and standard deviation, which are then used to define a dynamic threshold for identifying outliers.

* **Query Example**
    ```
    timeChart(function=[max(m1), window(function=[stddev(m1), avg(m1)], span=15m)])
    | groupBy(_bucket, function={ limit := _avg + 2 * _stddev | table([_max, limit]) })
    ```

* **Step-by-Step**
    1.  `timeChart()`: The query first creates a time chart that calculates two things for each time bucket:
        * `max(m1)`: The simple maximum value of the `m1` metric in that bucket.
        * `window(...)`: A 15-minute sliding window that calculates the running `avg(m1)` and `stddev(m1)` (standard deviation).
    2.  `groupBy(_bucket, ...)`: The results are then re-grouped by the time bucket.
    3.  `limit := _avg + 2 * _stddev`: Inside the `groupBy`, a new field `limit` is calculated. This creates a dynamic threshold that is two standard deviations above the running average.
    4.  `table([_max, limit])`: The final output is a table comparing the actual maximum value (`_max`) in a bucket to the calculated dynamic `limit`, making it easy to spot outliers.

---

### Calculate Multiple Running Aggregates per Series

This example demonstrates how to calculate multiple running aggregates (average and maximum) for different series (hosts) simultaneously.

* **Query Example**
    ```
    timeChart(host, function=window(function=[avg(cpu_load), max(cpu_load)], buckets=3))
    ```

* **Step-by-Step**
    1.  `timeChart(host, ...)`: The query creates a time chart with a separate series for each unique `host`.
    2.  `function=window(...)`: For each time bucket, the `window()` function is applied.
    3.  `buckets=3`: The window size is defined as the current bucket plus the two preceding buckets.
    4.  `function=[avg(cpu_load), max(cpu_load)]`: Within this 3-bucket window, it calculates both the `avg()` and the `max()` of the `cpu_load` field.

* **Summary and Results**
    This query produces a time chart showing the 3-bucket moving average and the 3-bucket moving maximum CPU load for each host. This is useful for monitoring performance, as it can smooth out temporary spikes and highlight sustained high CPU usage.

# worldMap()

The `worldMap()` function is a helper function that processes geographical data to make it compatible with the World Map widget. It can take either IP addresses or latitude/longitude coordinates as input and groups the data points using a geohashing algorithm.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **ip** | string | optional | The field containing the IP address to look up geo-coordinates for. |
| **lat** | string | optional | The field containing the latitude coordinate. |
| **lon** | string | optional | The field containing the longitude coordinate. |
| **magnitude**| aggregate function | optional <br> default: `count()` | A function used to calculate the weight of each geographic bucket. This value determines the size or opacity of the map markers. |
| **precision**| number | optional <br> default: 4 | The precision of the geohash calculation (from 1 to 12). A lower number is suitable for a global view, while a higher number is better for a small, zoomed-in area. |

***

## Example

### Create a World Map with Average Latency as Magnitude

This example demonstrates how to use the `magnitude` parameter to create a more insightful map. Instead of just counting the number of events per location, it calculates the average latency for each location and uses that value to determine the size of the map markers.

* **Query Example**
    ```
    worldMap(ip=myIpField, magnitude=avg(latency))
    ```

* **Step-by-Step**
    1.  The query starts with events that contain an IP address in `myIpField` and a `latency` measurement.
    2.  `worldMap()` takes the IP address from `myIpField` and converts it to a geographical location.
    3.  It then groups the events by their location using geohash buckets.
    4.  The `magnitude=avg(latency)` parameter tells the function to calculate the average `latency` for all events within each geographic bucket.
    5.  The resulting value is used to size the markers on the world map, with higher average latencies resulting in larger or more intense points.

* **Summary and Results**
    This query is useful for visualizing network performance and quickly spotting geographical patterns in latency. It allows you to identify regions with high latency, which could indicate network bottlenecks or other performance issues.

# writeJson()

The `writeJson()` function takes a list of field names, which can include dot notation for nesting and square brackets for arrays, and converts them into a structured JSON object.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_json` | The name of the output field where the JSON object will be stored. |
| **field** | array of strings | optional <br> default: `@rawstring` | The values and fields to be converted to JSON. The parameter name `field` can be omitted. |

***

## Function Operation

This function reads a list of flat field names and intelligently reconstructs a nested JSON structure from them. The field names provided are treated as prefix matches, meaning `writeJson(field=["a.b"])` would match fields like `a.b.c` and `a.b[0]`. It also supports array-globbing patterns, so `writeJson(field=["a.b[*]"])` would match all elements within the `a.b` array.

***

## Example

### Convert Flat Fields to a Nested JSON Object

This example demonstrates how to take several flat fields that represent a nested structure and convert them back into a single, structured JSON object.

* **Query Example**
    ```
    writeJson(["a.b.c", "a.b.e[0]", "a.d", "a.f.g"])
    ```

* **Input Event Data**
    Events with several flat fields that use dot notation to imply a nested structure.

| a.b.c | a.b.e[0] | a.d | a.f.g |
| :--- | :--- | :--- | :--- |
| value1 | value2 | value3 | value4 |
| test1 | test2 | test3 | test4 |
| data1 | data2 | data3 | data4 |

* **Step-by-Step**
    1.  The query starts with events containing the flattened fields.
    2.  `writeJson()` takes the array of field paths as input.
    3.  It processes the dot notation (`a.b.c`) to create nested objects and the square bracket notation (`a.b.e[0]`) to create arrays.
    4.  It combines all the specified paths into a single JSON object and stores it in the default `_json` field.

* **Result Event Data**
    The output shows the original flat fields along with the new `_json` field, which contains the reconstructed nested JSON object.

| _json | a.b.c | a.b.e[0] | a.d | a.f.g |
| :--- | :--- | :--- | :--- | :--- |
| `{"a":{"b":{"c":"value1","e":["value2"]},"d":"value3","f":{"g":"value4"}}}` | value1 | value2 | value3 | value4 |
| `{"a":{"b":{"c":"test1","e":["test2"]},"d":"test3","f":{"g":"test4"}}}` | test1 | test2 | test3 | test4 |
| `{"a":{"b":{"c":"data1","e":["data2"]},"d":"data3","f":{"g":"data4"}}}` | data1 | data2 | data3 | data4 |

# xml:prettyPrint()

The `xml:prettyPrint()` function formats an XML field to improve its readability by adding indentation and line breaks. This can be a computationally expensive operation, so it is best to use it at the end of a query after all filtering is complete.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | The name of the field to store the formatted output in. |
| **field** | string | required <br> default: `@rawstring` | The name of the field containing the XML to format. |
| **step** | integer | optional <br> default: 2 | The number of characters to use for indentation. |
| **strict** | boolean | optional <br> default: `false` | If `true`, only valid XML will be formatted; invalid XML will result in no output field. If `false`, invalid XML is copied to the output field unmodified. |
| **width** | integer | optional <br> default: 80 | The desired maximum width (in characters) of the output lines. |

***

## Examples

### Format Only Valid XML Input

This example demonstrates how to use the `strict` parameter to ensure that only well-formed XML is formatted and passed on, which is useful for validating XML integrity.

* **Query Example**
    ```
    formattedXml := xml:prettyPrint(field=message, strict=true)
    ```

* **Step-by-Step**
    1.  The query starts with events containing XML in a field named `message`.
    2.  `xml:prettyPrint()` is called on the `message` field.
    3.  Because `strict=true` is set, the function first validates the XML. If the XML in the `message` field is valid, it is formatted and stored in the `formattedXml` field.
    4.  If the XML is invalid, the `formattedXml` field is not created for that event.

---

### Format XML with Custom Line Length and Indentation

This example shows how to control the visual layout of the formatted XML by specifying the line width and indentation step.

* **Query Example**
    ```
    formattedXml := xml:prettyPrint(field=payload, step=4, width=100)
    ```

* **Step-by-Step**
    1.  The query targets a field named `payload` that contains XML data.
    2.  `xml:prettyPrint()` is called with two custom formatting parameters:
        * `step=4`: This sets the indentation for each level of the XML tree to 4 spaces.
        * `width=100`: This sets the maximum line length to 100 characters before wrapping.
    3.  The resulting, nicely formatted XML is stored in the `formattedXml` field.

---

### Format XML After Filtering Data (Recommended Practice)

This example shows the recommended best practice of applying `xml:prettyPrint()` at the end of a query, after the data has been filtered, to avoid unnecessary processing on events that will be discarded.

* **Query Example**
    ```
    #type=SOAP
    | account=123
    | xml:prettyPrint()
    ```
* **Step-by-Step**
    1.  The query first filters for all events where the `#type` is `SOAP`.
    2.  It then further filters those results to keep only events where the `account` field is `123`.
    3.  `xml:prettyPrint()` is then called on only this small, relevant subset of events. Since no `field` is specified, it formats the `@rawstring` of the filtered events.

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

# parseCsv()

The `parseCsv()` function parses a CSV-encoded field into known columns. It can parse values of the form:

  * `value 1, value 2, value 3`
  * `"value 1", "value 2", value 3` (Quoted values. Quotes are optional.)
  * `"value 1"; "value 2"; value 3` (Using `;` as delimiter. Delimiter is configurable.)

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `columns` | string or array | required | Names of columns to extract from field. |
| `delimiter` | string | optional \<br\> default: ` ,  ` | Delimiter character to split records by. |
| `excludeEmpty` | boolean | optional \<br\> default: `false` | If the value of a column is empty, exclude the field. |
| `field` | string | required \<br\> default: `@rawstring` | Field that holds the input in CSV form. |
| `trim` | boolean | optional \<br\> default: `false` | Allows to ignore whitespace before and after values. If the value is quoted, the quotes can start after the spaces. Useful for parsing data created by sources that do not adhere to the CSV standard. |

***

## Function Operation

The `parseCsv()` function takes a field containing CSV-formatted data and extracts its values into new, named fields. You must provide the expected column names using the `columns` parameter. The function can handle both unquoted and quoted values, and the delimiter character is configurable via the `delimiter` parameter (default is a comma).

The `excludeEmpty` parameter, if set to `true`, will prevent the creation of new fields for columns that have empty values. The `trim` parameter (default `false`) controls whitespace handling:

  * If `trim` is `true`, spaces around the delimiter are ignored, but spaces *within* quoted columns are retained. Quotes can appear after spaces. This is useful for parsing data that might not strictly adhere to CSV standards regarding whitespace.
  * If `trim` is `false` (default), spaces around the delimiter will be included as part of the field's value.

You can specify the input field to be parsed using the `field` parameter; if omitted, `@rawstring` is parsed by default.

***

## Examples

### Parse String as CSV

This example uses `parseCsv()` to parse a CSV-encoded field into known columns.

  * **Query Example**

    ```
    kvParse()
    parseCsv(result, columns=[count, status, completion, precision, sourcetask])
    ```

  * **Input Event Data**

    ```
    2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117, success,27%,3.14"
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs, extracting `result="117, success,27%,3.14"`.
    3.  `parseCsv(result, columns=[count, status, completion, precision, sourcetask])`: CSV parses the `result` field (extracted by `kvParse()`) and adds the following fields to the event: `count` with the value `117`, `status` with the value `success`, `completion` with the value `27%`, and `precision` with the value `3.14`. The `sourcetask` field will not be assigned a value as there were too few columns in the input for it.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to parse a string as CSV.

  * **Result Event Data**

| completion | count | precision | result | status |
| :--- | :--- | :--- | :--- | :--- |
| 27% | 117 | 3.14 | 117, success, 27%,3.14 | success |

---

### Parse String as CSV - Example 2

This example uses `parseCsv()` to parse a log line with spaces and quotes and trim the output by setting the `trim` parameter to `true`.

  * **Query Example**

    ```
    parseCsv(columns=[status, hosts, rest], trim=true)
    ```

  * **Input Event Data**
    (Assume `@rawstring` contains):
    `117, " crowdstrike.com, logscale.com ", 3.14`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseCsv(columns=[status, hosts, rest], trim=true)`: CSV parses the `@rawstring` (or implied `field` if used in a pipeline) and adds the following fields to the event: `status` with the value `117`, `hosts` with the value `" crowdstrike.com, logscale.com "`, `rest` with the value `3.14`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to parse a string as CSV. When `trim` is set to `true`, spaces around the separation character (e.g., a comma) are ignored, but spaces are retained within quoted columns. For example:

      * `117, crowdstrike.com, humio.com` would identify three columns: `117`, `crowdstrike.com`, `humio.com`.
      * `117," crowdstrike.com, humio.com ",3.14` would retain the spaces at the beginning and end of the quoted column.
        Without `trim` (`trim=false`), spaces around the character separator would be included in the values. For example, `117, "crowdstrike.com, humio.com "` would identify three columns where the comma between the two hostnames is interpreted as a separator, if the quotation mark after the space does not start a quoted value.

---

### Parse a CSV-encoded Field Into Known Columns

This example parses a CSV-encoded field into known columns and adds it to the event using the `parseCsv()` function.

  * **Query Example**

    ```
    parseCsv(result, columns=[count, status, completion, precision, sourcetask])
    ```

  * **Input Event Data**
    (Assume a log line from which a `result` field can be extracted, e.g., similar to "2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117, success,27%,3.14"")

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseCsv(result, columns=[count, status, completion, precision, sourcetask])`: Parses a CSV-encoded field into known columns. In this example, where the parsed field is from a log line, using `parseCsv(result, columns=[count, status, completion, precision, sourcetask])` will add these fields to the event: `count: 117`, `status: success`, `completion: 27%`, `precision: 3.14`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to parse a CSV-encoded field into known columns. CSV files are often used to exchange data between systems.

# parseFixedWidth()

The `parseFixedWidth()` function parses a fixed width-encoded field into known columns. It can parse values of the form:
`value 1 value 2 value 3`, widths `[8,8,8]`
`value 1value 2value 3`, widths `[7,7,7]`

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `columns` | array of strings | required | Names of columns to extract from field. |
| `field` | string | optional \<br\> default: `@rawstring` | Field that holds the input in fixed width form. |
| `trim` | boolean | optional \<br\> default: `true` | Remove leading and trailing white-space from fields after extracting. |
| `widths` | array of numbers | required | Widths of columns. |

***

## Function Operation

The `parseFixedWidth()` function is used to extract data from a field where values are arranged in fixed-width columns. It requires two main parameters: `columns`, which specifies the names for the new fields to be extracted, and `widths`, which provides an array of numbers indicating the exact width of each corresponding column in the input string.

By default, the function processes the `@rawstring` field, but a different input field can be specified using the `field` parameter. The `trim` parameter, which defaults to `true`, ensures that leading and trailing whitespace is removed from the extracted field values. If `trim` is set to `false`, whitespace will be retained.

This parsing method is particularly valuable for structured data that maintains strict positional formatting and character length requirements. If an extracted field value is longer than its specified width, the parser truncates the excess data while maintaining the overall structure of the parsed output.

***

## Examples

### Fixed width parse the result field from a log line

This example demonstrates parsing a fixed width-encoded field named `result` from a log line.

  * **Query Example**

    ```
    parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])
    ```

  * **Input Event Data**

    ```
    2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117success 27% 3.14"
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])`: Parses the fixed width-encoded `result` field from the log line.
    3.  Event Result set.

  * **Summary and Results**
    The query adds the following fields to the event: `count` with value `117`, `status` with value `success`, `completion` with value `27%`, and `precision` with value `3.14`. The `sourcetask` field will not be set as the input is too short. Values are trimmed after they have been extracted, for example, " success" will become "success".

  * **Result Event Data**
    (Implicitly added fields based on parsing)

      * `count = 117`
      * `status = success`
      * `completion = 27%`
      * `precision = 3.14`

---

### Parse Fixed Width-encoded Log Lines Fields

This example parses fixed width-encoded fields from log lines into columns using the `parseFixedWidth()` function.

  * **Query Example**

    ```
    parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])
    ```

  * **Input Event Data**
    (Example log line assumed to contain a `result` field in fixed-width format, e.g., similar to "117success 27% 3.14")

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])`: Parses the fixed width-encoded field in the access log and adds the returned values as known columns in the result field.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to parse compact numeric data consisting of fixed width-encoded fields into columns. In case a field value is longer than, for example, 10 characters, the parser handles overflow by truncating data that exceeds the specified field width while maintaining the structure of the parsed output. As an example, if the original `sourcetask` value was: `SCAN_FILES_WITH_VERY_LONG_NAME` (29 characters), then the extra characters `_WITH_VERY_LONG_NAME` would be truncated. This parsing method is particularly valuable when dealing with structured data that must maintain strict positional formatting and character length requirements.

# parseHexString()

The `parseHexString()` function parses the input from hex encoded bytes, decoding the resulting bytes as a string using the selected character set. If the input field has a prefix (other than `0x` and `16#`), then use `regex()` or `replace()` to remove the prefix before using `parseHexString()`. Any non-hex characters in the input are ignored; the decoding attempts to decode all characters in the field that match `0-9` or `A-F`. Case is ignored.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_parsehexstring` | Name of output field. |
| `charset` | string | optional \<br\> default: `UTF-8` | The charset to use when transforming bytes to string. \<br\>**Values:** `ISO-8859-1`, `UTF-16BE`, `UTF-16LE`, `UTF-8`. |
| `field` | string | required | Specifies the field containing the hex string to use as input. |

***

## Function Operation

The `parseHexString()` function converts a hexadecimal string from a specified input field into a readable string format. It decodes the hex-encoded bytes using the character set defined by the `charset` parameter (defaulting to `UTF-8`). The function is flexible in handling input, ignoring any non-hexadecimal characters and processing only those that match `0-9` or `A-F`, regardless of case. If the hexadecimal input string has prefixes other than `0x` or `16#`, these should be removed beforehand using functions like `regex()` or `replace()` for proper parsing. The resulting decoded string is then stored in a new output field, which by default is named `_parsehexstring` but can be customized with the `as` parameter.

***

## Examples

### Parses a hex string from field `foo` into field `text`

This example parses the hex string `48656c6c6f576f726c64` from the field named `foo` into the field `text`, resulting in the value "Helloworld" using `ISO-8859-1` character set.

  * **Query Example**

    ```
    foo: 48656c6c6f576f726c64
    parseHexString(foo, as=text, charset="ISO-8859-1")
    ```

  * **Step-by-Step**

    1.  The query starts by defining a field `foo` with the hexadecimal string `48656c6c6f576f726c64`.
    2.  `parseHexString(foo, as=text, charset="ISO-8859-1")` then parses the content of the `foo` field, interprets it as a hexadecimal string, and decodes it into a human-readable string using the `ISO-8859-1` character set. The result is stored in a new field named `text`.

  * **Result Event Data**
    (Implicitly, an event will contain the field `text` with the value "Helloworld")

---

### Parses a complex hex string from field `hex` into field `text`

This example parses a string containing `0x` prefix, spaces, and non-hex characters, along with a smiley emoji hex code, from the field `hex` into the field `text` using `UTF-8` charset, getting the value "Helloworld" followed by a smiley.

  * **Query Example**

    ```
    hex := "0x4 865 6c6c6f576f726c6420 plus F09F 98 80"
    | text := parseHexString(hex)
    ```

  * **Step-by-Step**

    1.  The query starts by assigning the string `"0x4 865 6c6c6f576f726c6420 plus F09F 98 80"` to a field named `hex`.
    2.  `| text := parseHexString(hex)` then parses the content of the `hex` field. The `parseHexString` function automatically handles the `0x` prefix and ignores non-hexadecimal characters like spaces and "plus ". It decodes the valid hexadecimal sequences (`48656c6c6f576f726c6420` and `F09F9880`) into a string. The result is stored in a new field named `text`.

  * **Result Event Data**
    (Implicitly, an event will contain the field `text` with the value "Helloworld 😊")
    (The smiley is the result of decoding the trailing digits `F09F9880` in UTF-8).

# parseInt()

The `parseInt()` function converts an integer from any radix (or number base), such as from hexadecimal or octal, to base 10, the decimal radix, expected as input by all other functions. For example, converting the hexadecimal FF to 255 using radix 16 or 77 to 63 using radix 8. The conversion is always unsigned. If the input field has a prefix (other than `0x` and `16#`) then use `regex()` to remove the prefix before using `parseInt()`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: (same as input field) | The output name of the field to set. |
| `endian` | string | optional \<br\> default: `big` | Input Digit-pair ordering (little, big) for hexadecimal. |
| `field` | string | required | The name of the input field. |
| `radix` | number | optional \<br\> default: `16` | Input Integer base (2 to 36). |

***

## Function Operation

The `parseInt()` function converts numerical strings from various radices (number bases) into their equivalent base 10 (decimal) integer representation. This conversion is always unsigned. It supports radices from 2 to 36. For hexadecimal inputs, you can specify the byte order (`endian`) as either `little` or `big`. If the input string has prefixes other than `0x` or `16#`, these must be removed prior to using `parseInt()` (e.g., with `regex()`). The converted decimal integer is then outputted to a new field, which by default will have the same name as the input field, but can be renamed using the `as` parameter.

***

## Examples

### Shows how to parse a hexadecimal string in little endian as an integer

This example demonstrates how to parse a hexadecimal string in little-endian format as an integer.

  * **Query Example**

    ```
    parseInt(hexval, as="centigrades", radix="16", endian="little")
    ```

  * **Input Event Data**
    An input event with the field `hexval` with the value `8001`.

  * **Result Event Data**
    The field `centigrades` having the value $(1 \* 256) + 128 = 384$.

---

### Shows how to parse a hexadecimal string in big endian as an integer

This example demonstrates how to parse a hexadecimal string in big-endian format as an integer.

  * **Query Example**

    ```
    parseInt(hexval, as="centigrades", radix="16", endian="big")
    ```

  * **Input Event Data**
    An input event with the field `hexval` with the value `8001`.

  * **Result Event Data**
    The field `centigrades` having the value $(128 \* 256) + 1 = 32769$.

---

### Shows how to parse a binary string as an integer

This example demonstrates how to parse a binary string as an integer.

  * **Query Example**

    ```
    parseInt(bitval, as="flags", radix="2")
    ```

  * **Input Event Data**
    An input event with the field `bitval` with the value `00011001`.

  * **Result Event Data**
    The field `flags` having the value $16 + 8 + 1 = 25$.

# parseJson()

The `parseJson()` function parses data or a field as JSON, converting the data into named fields and arrays within the event. Use `parseJson(field=@rawstring)` to parse the original ingested rawstring into JSON. Use the `prefix` parameter to prefix the names of the extracted fields. Furthermore, to exclude some of the extracted fields, use the `exclude` parameter to specify the JSON object structure to be excluded. For example, use `parseJson(field=input,exclude=a.b.c)` to exclude `c` and all of its descendants (a path-based exclusion) or use `parseJson(field=input,exclude="a.b[*].c")` to exclude all inside the array `b` (array-based exclusion). Note that all non-nested fields are also excluded. Use the `include` parameter if you need to keep certain descendants of an otherwise excluded path or an excluded non-nested field in the output.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `exclude` | array of strings | optional \<br\> default: `[]` | Removes specified JSON fields from the output. Use on fields that should be excluded from the result. Supports dot-pathing and array wildcards. If used with `prefix`, the excluded fields will use the specified prefix. The exclusion works as a wildcard match for the given field; for example, the value `query` will match both nested fields (like `query.string`) and similarly non-nested named fields (like `queryString`, `queryStart`, and `queryEnd`). To retain specific fields while excluding others, use the `include` parameter. |
| `excludeEmpty` | boolean | optional \<br\> default: `false` | Whether to exclude if the field is empty. \<br\>**Values:**\<br\>`false`: Do not exclude the field, even if the value is empty. \<br\>`true`: Exclude the field if the value is empty. |
| `field` | string | required \<br\> default: `@rawstring` | Fields that should be parsed as JSON. |
| `handleNull` | string | optional \<br\> default: `keep` | How null values are handled. \<br\>**Values:**\<br\>`discard`: Discard the null value and field. \<br\>`empty`: Replaces a null value with an empty string `""`. \<br\>`keep`: Converts the value to the `"null"` string. |
| `include` | array of strings | optional \<br\> default: `[]` | Retains specific descendants of excluded paths. Use on fields that should be included, even if they had been previously excluded by use of `exclude`. Supports dot-pathing and array wildcards. If used with `prefix`, the included fields will also use the specified prefix. |
| `prefix` | string | optional \<br\> default: (blank) | Prefix the name of the extracted JSON fields with the value of this parameter. |
| `removePrefixes` | array of strings | optional \<br\> default: `[]` | Prefixes that should be removed from the names of the extracted JSON fields; supports dot-pathing. If multiple prefixes are supplied, the longest matching prefix will be used. |

***

## Function Operation

The `parseJson()` function is used to convert JSON-formatted data into structured fields and arrays within a LogScale event. It primarily parses the field specified by `field` (defaulting to `@rawstring`).

When parsing JSON, the following rules apply:

  * **`excludeEmpty=true`**: Key/value pairs where the value is empty (`""`) will be completely discarded.
  * **`handleNull`**:
      * If `handleNull` is set to `discard`, any key/value pair with a `null` value will be discarded entirely, regardless of the `excludeEmpty` setting.
      * If `handleNull` is set to `empty`, `null` values are replaced with an empty string (`""`).
      * If `handleNull` is set to `keep` (default), `null` values are converted to the string `"null"`.

The `prefix` parameter allows adding a custom prefix to the names of all extracted JSON fields, which is useful for avoiding naming collisions with existing fields. Conversely, `removePrefixes` can be used to remove specified prefixes from the extracted field names; if multiple prefixes match, the longest one is removed.

The `exclude` parameter allows for the removal of specific JSON fields or structures from the output. It supports dot-pathing for nested objects and array wildcards (`[*]`) for arrays. When `exclude` is used, it acts as a wildcard match, meaning a value like `query` will exclude both nested fields (e.g., `query.string`) and similarly named non-nested fields (e.g., `queryString`). The `include` parameter can be used in conjunction with `exclude` to selectively retain certain descendants of an otherwise excluded path or a non-nested field. If both `include` and `exclude` are used with `prefix`, the `include` and `exclude` paths must also be prefixed.

***

## Examples

### Parse JSON Content With Specific Parameters

This example parses JSON content with specific parameters, excluding the actual query content while including only the `queryStart` field.

  * **Query Example**

    ```
    #type=audit-log
    | /"type": "alert.update"/
    parseJson(exclude="query", include="queryStart")
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=audit-log`: Filters for all events from the `audit-log` repository.
    3.  `|"type": "alert.update"/`: Filters for events (audit logs) where the `@rawstring` field contains the string `"type": "alert.update"`.
    4.  `parseJson(exclude="query", include="queryStart")`: Parses the JSON content. With specific parameters set, it excludes the `query` field and includes only the `queryStart` field. The exclusion works as a wildcard match for the given field; for example, the value `query` will match both nested fields (like `query.string`) and similarly non-nested named fields (like `queryString`, `queryStart`, and `queryEnd`). In this example, to retain the specific non-nested field `queryStart` while excluding the others, the `include` parameter is used.
    5.  Event Result set.

  * **Summary and Results**
    The query is used to search audit logs for alert update events, specifically looking at the non-nested `queryStart` field, while excluding the actual query content (`query.xxx`). This query is useful if, for example, you want to review alert activity without the overhead of full query contents, track temporal patterns in alert updates, or investigate alert timing issues further.

---

### Create Two Temporary Events for Troubleshooting - Example 2

This example demonstrates creating two temporary events for testing or troubleshooting using the `createEvents()` function with `parseJson()`.

  * **Query Example**

    ```
    createEvents(["{\"animal\":{\"kind\":\"dog\", \"weight\":7.0}}", "{\"animal\":{\"kind\":\"cat\", \"weight\":4.2}}"])
    | parseJson()
    ```

  * **Input Event Data**
    (Representing the raw JSON strings created by `createEvents()`):

    ```json
    [
      {"animal":{"kind":"dog", "weight":7.0}},
      {"animal":{"kind":"cat", "weight":4.2}}
    ]
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `createEvents(["{\"animal\":{\"kind\":\"dog\", \"weight\":7.0}}", "{\"animal\":{\"kind\":\"cat\", \"weight\":4.2}}"])`: Creates two temporary events, one with information about a dog and another about a cat.
    3.  `| parseJson()`: Parses the raw string content of these newly created events as JSON.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to create temporary events and parse their raw string content as JSON.

  * **Result Event Data**

| @timestamp | animal.kind | animal.weight |
| :--- | :--- | :--- |
| 1733311547717 | dog | 7.0 |
| 1733311547717 | cat | 4.2 |

# parseLEEF()

The `parseLEEF()` function parses Log Event Extended Format (LEEF) encoded messages. Only LEEF versions 1.0 and 2.0 are supported.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `delimiter` | string | optional | Delimiter to use when parsing the extension fields of a LEEF:1.0 message. When processing LEEF:2.0 messages this argument is ignored as the value is specified in a message header. The value must be a single character with unicode value in the range 0x0000-0xffff. |
| `field` | string | optional \<br\> default: `@rawstring` | Field that holds the input in LEEF form. |
| `headerprefix` | string | optional \<br\> default: `leef.` | Prefix to the field names for the header fields. |
| `keeplabels` | boolean | optional \<br\> default: `false` | Keep short form label fields (LEEF:1.0 only), which is interpreted as CEF. |
| `labelprefix` | string | optional \<br\> default: `leef.label` | Prefix to extension fields with labels (LEEF:1.0 only), which is interpreted as CEF. |
| `parsetime` | boolean | optional \<br\> default: `true` | Control if the `devTime` extension field should be parsed. Setting this to `false` can be used to disable the time parsing step. \<br\>**Values:**\<br\>`false`: Disables parsing of the `devTime` field. \<br\>`true`: Enables parsing of the `devTime` field. |
| `prefix` | string | optional \<br\> default: `leef.ext.` | Prefix to extension fields. Fields in the LEEF extension part are prefixed with this. |
| `timezone` | string | optional \<br\> default: `Z` | Time zone to use if none specified in `devTimeFormat` string. |

***

## Function Operation

The `parseLEEF()` function is used to parse Log Event Extended Format (LEEF) messages, specifically supporting versions 1.0 and 2.0. This function will automatically skip any prefix up to the `LEEF:1.0` or `LEEF:2.0` marker. This means that if LEEF messages are delivered via Syslog, the Syslog portion of the message needs to be parsed separately.

If the LEEF message contains a `devTime` (and optionally `devTimeFormat`), this function will extract the timestamp from there and assign it to `@timestamp`. You can control whether `devTime` is parsed using the `parsetime` parameter.

The `field` parameter (defaulting to `@rawstring`) specifies the input field containing the LEEF message. You can customize the prefixes added to extracted field names: `headerprefix` for header fields (default `leef.`), `labelprefix` for labeled extension fields in LEEF 1.0 (default `leef.label`), and `prefix` for general extension fields (default `leef.ext.`). The `keeplabels` parameter (LEEF 1.0 only) controls whether short-form label fields are kept. The `delimiter` parameter is used for parsing LEEF 1.0 extension fields. For LEEF 2.0, the delimiter is specified within the message header itself, so this argument is ignored. The `timezone` parameter allows specifying a default timezone if none is provided in the `devTimeFormat` string.

In general, the implementation is more permissive than the official LEEF specification to allow for common mistakes in log writers. Notably, a `LEEF:0` marker is parsed as LEEF 1.0 headers with a CEF:0-style body for legacy support.

***

## Examples

### Parse the @rawstring field to show how the LEEF format is parsed with the default field names

This example demonstrates parsing a LEEF 2.0 log line with default field naming.

  * **Query Example**

    ```
    parseLEEF(field=@rawstring)
    ```

  * **Input Event Data**
    (From a log line like this:)
    `<13>1 2019-01-18T11:07:53.520Z 192.168.1.1 LEEF:2.0|Lancope|StealthWatch|1.0|41|^|src=10.0.0.1^act=blocked an X^dst=1.1.1.1`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| leef.version | 2.0 |
| leef.device.vendor | Lancope |
| leef.device.product | StealthWatch |
| leef.device.version | 1.0 |
| leef.event id | 41 |
| leef.ext.src | 10.0.0.1 |
| leef.ext.act | "blocked an X" |
| leef.ext.dst | 1.1.1.1 |

---

### Show how the LEEF format is parsed with changed header prefix

This example demonstrates parsing a LEEF 2.0 log line with a custom header prefix.

  * **Query Example**

    ```
    parseLEEF(headerprefix="leef.header.")
    ```

  * **Input Event Data**
    (From a log line like this:)
    `LEEF:2.0 Palo Alto Networks|LF|2.0|THREAT||TimeReceived=2016-01-01T00:00:58.000000Z DeviceSN=123456789123 EventID=THREAT cat= ConfigVersion=9.1`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| leef.header.version | 2.0 |
| leef.header.device.vendor | Palo Alto Networks |
| leef.header.device.product | LF |
| leef.header.device.version | 2.0 |
| leef.ext.DeviceSN | 123456789123 |
| leef.ext.EventID | THREAT |
| leef.ext.TimeReceived | 2016-01-01T00:00:58.000000Z |
| leef.ext.cat |  |
| leef.ext.ConfigVersion | 9.1 |

---

### Show how the LEEF format is parsed with changed extension prefix

This example demonstrates parsing a LEEF 2.0 log line with a custom extension prefix.

  * **Query Example**

    ```
    parseLEEF(prefix="my.prefix.")
    ```

  * **Input Event Data**
    (From the same log line as the previous example):
    `LEEF:2.0 Palo Alto Networks|LF|2.0|THREAT||TimeReceived=2016-01-01T00:00:58.000000Z DeviceSN=123456789123 EventID=THREAT cat= ConfigVersion=9.1`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| leef.version | 2.0 |
| leef.event id | THREAT |
| leef.device.vendor | Palo Alto Networks |
| leef.device.product | 2LF |
| leef.device.version | 2.0 |
| my.prefix.DeviceSN | 2123456789123 |
| my.prefix.EventID | THREAT |
| my.prefix.TimeReceived | 2016-01-01T00:00:58.0000002 |
| my.prefix.cat |  |
| my.prefix.ConfigVersion | 9.1 |

# parseTimestamp()

The `parseTimestamp()` function parses a string into a timestamp. This function is important for creating parsers, as it is used to parse the timestamp for an incoming event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `addErrors` | boolean | optional \<br\> default: `true` | Whether to add an error field `_tc_error` to the event if it was not possible to parse the timestamp. |
| `as` | string | optional \<br\> default: `@timestamp` | Name of output field that will contain the timestamp. The timestamp is represented in milliseconds since January 1, 1970 in UTC. LogScale expects to find the timestamp in the field `@timestamp` so do not change when creating parsers. |
| `caseSensitive` | boolean | optional \<br\> default: `true` | Whether the timestamp format pattern is case sensitive. For example, the format `LLL` will accept `Feb` but not `feb` in case sensitive mode, while `feb` will be accepted in case insensitive mode. \<br\>**Values:**\<br\>`false`: Pattern is not case sensitive. \<br\>`true`: Pattern is case sensitive. |
| `field` | string | required | The field holding the timestamp to be parsed. |
| `format` | string | optional \<br\> default: `yyyy-MM-dd'T'HH:mm:ss[.SSSSSSSSS]XXXXX` | Pattern used to parse the timestamp. If a format string is not specified, it is inferred using `java.time.format.DateTimeFormatter` or one of the special format specifiers. The special format specifiers are case-insensitive. \<br\>**Values:**\<br\>`millis` or `milliseconds`: Epoch time in milliseconds. \<br\>`nanos`: Epoch time in nanoseconds. \<br\>`seconds` or `unixTimeSeconds` or `unixtime`: Epoch time in seconds (UTC). |
| `timezone` | string | optional | If the timestamp does not contain a timezone, it can be specified using this parameter. Examples are `Europe/London`, `America/New_York`, and `UTC`. See the full list of timezones supported by LogScale at Supported Time Zones. If the timestamp does not contain a timezone, and no timezone is specified here, an error is generated. If the timezone specified here and one also exists in the timestamp, then this parameter overrides the timezone in the event. |
| `timezoneAs` | string | optional \<br\> default: `@timezone` | Name of output field that will contain the timezone. LogScale expects to find the timezone in the field `@timezone` so do not change when creating parsers. |

***

## Function Operation

The `parseTimestamp()` function is crucial for converting string representations of time into standardized timestamps within LogScale. Before parsing, the timestamp part of the log should ideally be captured in a field, usually during parsing, but it can also be extracted during queries using functions like `regex()` and `parseJson()` before `parseTimestamp()`.

This function formats times using a subset of Java's `DateTimeFormatter`. It also supports special format strings such as `seconds`, `milliseconds`, `nanos`, `unixtime`, `unixtimeMillis`, and `unixTimeSeconds` for epoch time conversions. For formats specifying seconds, it can also handle milliseconds using floating-point numbers (e.g., `1690480444.589` for 2023-07-27 19:54:04 and 589 milliseconds). Nanosecond precision is also supported, and the nanosecond component will be extracted.

After parsing, a field named `@timestamp` is created, containing the parsed timestamp in UTC milliseconds, and a `@timezone` field containing the original timezone.

`parseTimestamp()` can also parse time formats that omit the year designator, often seen in Syslog. For instance, `Mar 15 07:48:13` can be parsed using the format `MMM dd HH:mm:ss`. In such cases, LogScale infers the year: if the date is less than 8 days into the future or in the past, the current year is used; otherwise, if it's more than 8 days into the future, the previous year is used.

If the input timestamp string does not contain timezone information, the `timezone` parameter **must** be specified, otherwise an error will be generated. If a timezone is specified both in the parameter and the timestamp, the parameter's value overrides the timestamp's timezone. The `addErrors` parameter controls whether an error field `_tc_error` is added if parsing fails. The `caseSensitive` parameter determines if the format pattern matching is case-sensitive.

***

## Examples

### Extract a timestamp that is using millisecond precision embedded in a JSON value

This example demonstrates how to extract and parse a timestamp with millisecond precision from a JSON field.

  * **Query Example**

    ```
    parseJson()
    parseTimestamp("millis", field=timestamp)
    ```

  * **Step-by-Step**

    1.  The `parseJson()` function is used first to parse the raw event string, assuming it contains JSON data with a `timestamp` field.
    2.  `parseTimestamp("millis", field=timestamp)` then takes the `timestamp` field (extracted by `parseJson()`) and parses it as an epoch time in milliseconds.

---

### Parse timestamps in an accesslog where the timestamp includes an explicit timezone offset

This example shows parsing a timestamp from an access log that includes an explicit timezone offset.

  * **Query Example**

    ```
    /(?<client>\S+) \[(?<@timestamp>.+)\] (?<method>\S+) (?<url>\S+)/
    parseTimestamp("dd/MMM/yyyy:HH:mm:ss Z", field=@timestamp)
    ```

  * **Input Event Data**
    `192.168.1.19 [02/Apr/2014:16:29:32 +0200] GET /hello/test/123`

  * **Step-by-Step**

    1.  `/(?<client>\S+) \[(?<@timestamp>.+)\] (?<method>\S+) (?<url>\S+)/`: This regex extracts various fields from the log line, including the timestamp into `@timestamp`.
    2.  `parseTimestamp("dd/MMM/yyyy:HH:mm:ss Z", field=@timestamp)`: Parses the extracted `@timestamp` field using the specified format string, which includes a timezone offset (`Z`).

---

### Parse an event with a timestamp not containing year

This example demonstrates parsing a timestamp that does not include the year, commonly found in Syslog entries, by specifying a timezone.

  * **Query Example**

    ```
    /(?<timestamp>\S+\s+\d+\s+\d+:\d+:\d+)/
    parseTimestamp("MMM dd HH:mm:ss", field=timestamp, timezone="Europe/London")
    ```

  * **Input Event Data**
    (Assume an event with a timestamp like): `Feb 9 12:22:44 hello world`

  * **Step-by-Step**

    1.  `/(?<timestamp>\S+\s+\d+\s+\d+:\d+:\d+)/`: This regex captures the timestamp part (e.g., `Feb 9 12:22:44`) into a field named `timestamp`.
    2.  `parseTimestamp("MMM dd HH:mm:ss", field=timestamp, timezone="Europe/London")`: Parses the `timestamp` field. Since the year is not present in the timestamp string, the `timezone` parameter is used to provide the necessary context for correct parsing. LogScale will infer the year based on the current date and the 8-day rule.

---

### Bucket Events Into Groups

This example uses `bucket()` and `parseTimestamp()` to group events into 24 buckets and then parse the generated bucket timestamps.

  * **Query Example**

    ```
    bucket(buckets=24, function=sum("count"))
    parseTimestamp(field=_bucket, format=millis)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `bucket(buckets=24, function=sum("count"))`: Buckets the events into 24 groups spanning over a period of one day, using the `sum()` function on the `count` field. The `bucket` function outputs the timestamp as an epoch value in the `_bucket` field.
    3.  `parseTimestamp(field=_bucket, format=millis)`: Extracts the timestamp from the generated `_bucket` field and converts this epoch timestamp (in milliseconds) to a date-time value.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to optimize data storage and query performance by making it easier to manage and locate data subsets when performing analytics tasks. The resulting output shows 25 buckets; the original requested 24 buckets and in addition the bucket for the extracted timestamp.

  * **Result Event Data**

| \_bucket | sum | @timestamp |
| :--- | :--- | :--- |
| 1681290000000 | 1322658945428 | 1681290000000 |

---

### Parse Timestamp Without Timezone Information

This example converts local time strings to timestamps with timezone specification using the `parseTimestamp()` function with the `timezone` parameter.

  * **Query Example**

    ```
    parseTimestamp("yyyy-MM-dd'T'HH:mm:ss", field=event_time, timezone="America/New_York")
    ```

  * **Input Event Data**

    | event\_time | action | user |
    | :--- | :--- | :--- |
    | 2023-05-02T10:30:00 | login | jsmith |
    | 2023-05-02T10:35:00 | logout | jsmith |
    | 2023-05-02T10:40:00 | login | awhite |
    | 2023-05-02T10:45:00 | update | awhite |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseTimestamp("yyyy-MM-dd'T'HH:mm:ss", field=event_time, timezone="America/New_York")`: Parses the timestamp string in `event_time` using the specified format pattern. The `timezone` parameter is set to `America/New_York` to properly interpret the local time. The result is stored in a new field named `@timestamp` (and the timezone information in `@timezone`). Note that if the timestamp string does not contain a timezone, then one must be specified using the `timezone` parameter, otherwise an error is generated.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to convert local timestamp strings into properly formatted timestamps with timezone information. This query is useful, for example, to standardize timestamp fields in logs that contain local time information without explicit timezone data.

  * **Result Event Data**

| @timezone | action | event\_time | user |
| :--- | :--- | :--- | :--- |
| America/New\_York | login | 2023-05-02T10:30:00 | jsmith |
| America/New\_York | logout | 2023-05-02T10:35:00 | jsmith |
| America/New\_York | login | 2023-05-02T10:40:00 | awhite |
| America/New\_York | update | 2023-05-02T10:45:00 | awhite |

# parseUri()

This function extracts URI components from an input field and adds them as attributes to the event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `defaultBase` | string | optional | A scheme and authority prefix or `//`, used as the base to resolve the input as a URI reference. Valid values are of the form `//` or `scheme://` where `scheme` is a URI scheme. |
| `field` | string | required | The name of the input field that contains the value to analyze. |
| `prefix` | string | optional \<br\> default: input field name | An optional prefix for the field names for the components that are added to the event. It is an error to provide an empty prefix. |

***

## Function Operation

The `parseUri()` function is particularly useful for interpreting partial URIs, such as `www.example.com/path`. Instead of requiring fully valid URIs, `parseUri()` accepts a "base" (for example, `defaultBase=http://`) which is used to extract components like hostname, port, path, query, and fragment.

If the input URI has no authority component, the event remains unchanged, similar to the behavior of `parseUrl()`. For instance, `example.com` would be interpreted as the scheme, and no hostname or path would be found, as the function attempts to interpret the input as a URI. This is where providing a `defaultBase` can override such behavior. If a `defaultBase` is used with an already full URI (e.g., `ftp://example.com/path`), the `defaultBase` is ignored, and the existing scheme (e.g., `ftp`) will be extracted.

This function is more lenient than strict URI specifications and primarily matches based on the input's structure, by design, to enhance its utility in various contexts. It performs no normalization or decoding (like percent-encoding or path normalization) on the components; it processes the parts of the input exactly as they appeared in the logs. The URI components (such as `url.scheme`, `url.hostname`, `url.path`, `url.query`, `url.fragment`) are added to the event only if the input can be parsed as a URI itself or resolved to a URI reference by providing the `defaultBase` parameter. The `url.` prefix for these attributes is replaced by the input field name or the specified `prefix` value.

***

## Examples

### Parse with `defaultBase`

This example demonstrates parsing a partial URI with a specified `defaultBase`.

  * **Input Event Data**:
    `example.com:8080/foo`

  * **Query Example**:

    ```
    parseUri(field="url", defaultBase="http://")
    ```

  * **Output**:

      * `host=example.com`
      * `path=/foo`
      * `port=8080`
      * `scheme=http`

---

### Parse without `defaultBase`

This example shows parsing a partial URI without a `defaultBase`, illustrating how it might be misinterpreted.

  * **Input Event Data**:
    `example.com:8080/foo`

  * **Query Example**:

    ```
    parseUri(field="url")
    ```

  * **Output**:

      * `scheme=example.com`
      * `path=8080/foo`

---

### Parse full URI with `defaultBase`

This example illustrates that if a full URI is provided, `defaultBase` is ignored.

  * **Input Event Data**:
    `ftp://example.com:8080/foo`

  * **Query Example**:

    ```
    parseUri(field="url", defaultBase="http://")
    ```

  * **Output**:

      * `host=example.com`
      * `path=/foo`
      * `port=8080`
      * `scheme=ftp`

# parseUrl()

This function extracts URL components from an input field and adds them as attributes to the event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional | Use a prefix for the attributes added to the event. |
| `field` | string | optional \<br\> default: `url` | The field from which to parse URL components. |

***

## Function Operation

The `parseUrl()` function uses the Java `java.net.URI` class to parse and extract components from a URL string. This includes attributes such as scheme, username, password, host, port, path, fragment, and query, which are then added as new fields to the event.

Due to the parsing mechanism, the function may misidentify URL elements if the provided URL is only partial. By default, the parser prioritizes identifying the URL scheme (the part before the first colon). This can lead to incorrect parsing for URLs that are not fully qualified. For example, if "[site.example.com:80/path/to/examp](https://www.google.com/search?q=https://site.example.com:80/path/to/examp)" is provided, "site.example.com" might be incorrectly identified as `url.scheme`. Therefore, it is important to supply fully-qualified URLs to ensure accurate parsing of all components.

The function is designed to process the actual parts of the input as they were in the logs, meaning it does not perform any normalization or decoding (like percent-encoding or path normalization) on the components.

***

## Examples

### Parsing the `uri` field to aggregate the list of paths used

This example demonstrates parsing the `uri` field to extract URL components and then grouping by the `uri.path` to aggregate paths.

  * **Query Example**

    ```
    parseUrl(uri)
    groupBy(uri.path)
    ```

  * **Step-by-Step**

    1.  `parseUrl(uri)`: Parses the `uri` field in the event, extracting various URL components like scheme, host, path, etc., and adds them as new fields (e.g., `uri.path`) to the event.
    2.  `groupBy(uri.path)`: Groups the events by their extracted `uri.path`, which allows for aggregation of similar paths.

  * **Result Event Data**

| uri | \_count |
| :--- | :--- |
| /graphql | 132 |
| /api/v1/refresh | 95 |
| /api/v1/repositories/humio/queryjobs | 30 |
| /api/v1/repositories/humio/queryjobs/P29-vxBwkHK6nxeDuC4CzOOmc0Oq | 3 |
| /api/v1/repositories/humio/queryjobs/P7-mHpupr5riWe5XEoRnY8VhicU | 2 |
| /api/v1/repositories/humio/queryjobs/P7-eaRjXUkTzse6fB3M6kQcbh4m | 2 |
| /api/v1/repositories/humio/queryjobs/P7-WjVRVoJWmdh9REgpuoGHyoOC | 2 |
| /api/v1/repositories/humio/queryjobs/P7-VadNjnjvQaeXcfJRhfRb5Uuv | 2 |
| /api/v1/repositories/humio/queryjobs/P7-GlsGrU405Dml2PPeL33PwT9n | 2 |
| /api/v1/repositories/humio/queryjobs/P7-Bwg99bEHk6NeBmSU1fSZG0Fa | 2 |
| /api/v1/repositories/humio/queryjobs/P6-6VusohlK49WisjAYttFlbpfm | 2 |

---

### Parses the field named `endpoint` and adds URL components to the event

This example shows how to parse a field named `endpoint` to extract URL components.

  * **Query Example**

    ```
    parseUrl(field=endpoint)
    ```

  * **Step-by-Step**

    1.  `parseUrl(field=endpoint)`: Parses the content of the `endpoint` field to extract URL components and add them as new fields to the event.

---

### Parses the field named `endpoint` and adds URL components with `url` as a prefix

This example demonstrates parsing the `endpoint` field and prefixing the extracted URL components with `url`.

  * **Query Example**

    ```
    url: parseUrl(field=endpoint)
    ```

  * **Step-by-Step**

    1.  `url: parseUrl(field=endpoint)`: Parses the content of the `endpoint` field and adds URL components to the event with the prefix `url`, such as `url.path`, `url.scheme`, etc.

---

### Parses the field named `endpoint` and adds URL components with `apiEndpoint` as a prefix

This example illustrates parsing the `endpoint` field and using a custom prefix `apiEndpoint` for the extracted URL components.

  * **Query Example**

    ```
    parseUrl(field=endpoint, as=apiEndpoint)
    ```

  * **Step-by-Step**

    1.  `parseUrl(field=endpoint, as=apiEndpoint)`: Parses the content of the `endpoint` field and adds URL components to the event with `apiEndpoint` as a prefix, resulting in fields like `apiEndpoint.path`, `apiEndpoint.scheme`, etc.

# parseXml()

The `parseXml()` function parses data as XML. Specify `field=@rawstring` to parse the `@rawstring` into XML. If the specified field does not exist, the event is skipped. If the specified field exists but contains non-XML data, the behavior depends on the `strict` parameter.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | required \<br\> default: `@rawstring` | Field that should be parsed as XML. |
| `prefix` | string | optional | Prefix the name of the fields extracted from XML with the value of this parameter. |
| `strict` | boolean | optional \<br\> default: `false` | Specifies if events where the field does not contain valid XML should be filtered out of the result set. \<br\>**Values:**\<br\>`false`: All events are passed, including an error flag describing why the event parsing failed. \<br\>`true`: Events that do not contain valid XML will be filtered in the result set. |

***

## Function Operation

The `parseXml()` function is used to parse XML-formatted data from a specified field into new, structured fields within an event. By default, it attempts to parse the `@rawstring` field.

The behavior of `parseXml()` depends on the validity of the XML data and the `strict` parameter:

  * If the `field` specified does not exist in an event, that event is simply skipped.
  * If the `field` exists but contains data that is not valid XML:
      * When `strict` is `false` (the default), all events are passed through. However, an internal error flag will be set on events where parsing failed, describing why the XML parsing was unsuccessful.
      * When `strict` is `true`, any events where the specified field does not contain valid XML data will be filtered out from the result set, meaning they will not appear in the output.

You can also use the `prefix` parameter to add a custom prefix to the names of all fields extracted from the XML, which helps in organizing data and avoiding naming conflicts.

***

## Examples

### Parse entire event as XML and extract timestamp

This example demonstrates parsing an entire event that is formatted as XML and then extracting and parsing a timestamp field from the XML.

  * **Query Example**

    ```
    parseXml()
    parseTimestamp(field=timestamp)
    ```

  * **Input Event Data**

    ```xml
    <log>
        <service>userService</service>
        <timestamp>2017-12-18T20:39:35Z</timestamp>
        <msg>user with id=47 logged in</msg>
    </log>
    ```

  * **Step-by-Step**

    1.  `parseXml()`: Parses the entire `@rawstring` (assuming the whole event is XML) into structured fields. This will create fields such as `log.service`, `log.timestamp`, and `log.msg`.
    2.  `parseTimestamp(field=timestamp)`: Takes the `timestamp` field (which would be `log.timestamp` after `parseXml()` if the `log` element is the root, or directly `timestamp` if no root element hierarchy is applied) and converts its string value into a proper timestamp.

---

### Parse XML embedded within a field after `kvParse()`

This example shows how to extract an XML string from a field using `kvParse()` and then parse that extracted XML.

  * **Query Example**

    ```
    /(?<timestamp>\S+)/
    parseTimestamp(field=timestamp)
    | kvParse()
    | parseXml(field=details)
    ```

  * **Input Event Data**
    (A log line that might look like this, where `details` contains XML):
    `2017-12-18T20:39:35Z user id=47 logged in details="<log><service>userService</service><timestamp>2017-12-18T20:39:35Z</timestamp><msg>user with id=47 logged in</msg></log>"`

  * **Step-by-Step**

    1.  `/(?<timestamp>\S+)/`: Extracts the initial timestamp from the raw event into a field named `timestamp`.
    2.  `parseTimestamp(field=timestamp)`: Parses this `timestamp` field into a structured timestamp.
    3.  `| kvParse()`: Parses the raw event string into key-value pairs, which would extract `details` as a field containing the XML string.
    4.  `| parseXml(field=details)`: Takes the `details` field (extracted by `kvParse()`) and parses its content as XML, converting the XML elements into new fields within the event (e.g., `log.service`, `log.timestamp`, `log.msg`).

# percentile()

`percentile()` is an estimation function that estimates percentiles over a given collection of numbers.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `accuracy` | double | optional \<br\> default: `0.01` | Provided as a relative error threshold. Can be between \> 0 and \<1: values closer to 1 means lower accuracy, values closer to 0 means higher accuracy. |
| `as` | string | optional | Prefix of output fields. |
| `field` | string | required | Specifies the field for which to calculate percentiles. The field must contain numbers. |
| `percentiles` | array of numbers | optional \<br\> default: `[50, 75, 99]` | Specifies which percentiles to calculate. |

***

## Function Operation

A percentile is a comparison value between a particular value and the values of the rest of a group. This enables the identification of scores that a particular score surpassed. For example, with a value of 75 ranked in the 85th percentile, it means that the score 75 is higher than 85% of the values of the entire group. This can be used to determine thresholds and limits for triggering events or scoring probabilities and threats.

For example, given the values 12, 25, 50, and 99, the 50th percentile would be any value between 25 and 50; in this case, the `percentile()` function will return 25.79. LogScale's `percentile()` function returns any valid value in order to reduce resource usage and not the mean of valid values, as percentile algorithms in general often return.

LogScale uses an approximate algorithm for percentiles to achieve a good balance of speed, memory usage, and accuracy. The function returns one event with a field for each of the percentiles specified in the `percentiles` parameter. Fields are named by prepending `_` to the values specified in the `percentiles` parameter. For example, the event could contain the fields `_50`, `_75`, and `_99`.

The following conditions apply when using this function:

  * The function only works on non-negative input values.
  * The `accuracy` argument specifies the accuracy of the percentile relative to the number estimated and is intended as a relative error tolerance (lower values imply better accuracy).
      * An `accuracy` of `0.001` specifies the accuracy of the percentile relative to the number estimated (note that specifying `accuracy=0.001` actually implies that the accuracy is `0.999`). The number estimated depends on the `accuracy` argument and the amount of data available. A larger amount of data returns better estimations. For example, with an original value of 1000, the value would be between 999 and 1001 (1000 - 1000/1000 and 1000 + 1000/1000).
      * An `accuracy` of `0.01` means accuracy to 1/100 of the original value. For example, with an original value of 1000, the value would be between 990 and 1010 ((1000 - 1000/100 and 1000 + 1000/100)). With an original value of 500, the value would be between 495 and 505 ((500 - 500/100 and 500 + 500/100)).
  * Any query that does not explicitly set the `accuracy` parameter may see a change in the reported percentile. Specifically, the `percentile()` function may deviate by up to one 100th of the true percentile, meaning that if a given percentile has a true value of 1000, the function may report a percentile in the range of `[990; 1010]`.

Higher accuracy implies higher memory usage. Be careful to choose the accuracy for the kind of precision needed from the expected output value. Lower percentiles are discarded if memory usage becomes too high. If your percentiles seem off, try reducing the accuracy.

***

## Examples

### Calculate the 50th, 75th, 99th and 99.9th percentiles for events with the field responsetime

This is a basic example of calculating multiple percentiles for a given field.

  * **Query Example**
    ```
    percentile(field=responsetime, percentiles=[50,75,99,99.9])
    ```

### In a timechart, calculate percentiles for both of the fields r1 and r2

This example shows how to calculate percentiles for multiple fields within a timechart.

  * **Query Example**
    ```
    timeChart(function=[percentile(field=r1, as=r1), percentile(field=r2, as=r2)])
    ```

### To calculate the median for a given value, use percentile() with percentiles set to 50

This example demonstrates how to use `percentile()` to calculate the median by setting the percentile to 50.

  * **Query Example**

    ```
    percentile(field=allocBytes, percentiles=[50], as=median)
    ```

  * **Step-by-Step**

    1.  `percentile(field=allocBytes, percentiles=[50], as=median)`: Calculates the 50th percentile of the `allocBytes` field and outputs the result in a field named `median_50`.

  * **Summary and Results**
    This query creates the field `median_50` with the 50th percentile value.

---

### Determine a Score Based on Field Value

This example calculates percentiles for `filesize` and then uses conditional logic to assign a symbolic score based on which percentile range the `filesize` falls into.

  * **Query Example**

    ```
    percentile(filesize, percentiles=[40,80], as=score)
    | symbol := if(filesize > score_80, then=":+1:", else if(filesize > score_40, then="so-so", else=":-1:"))
    ```

  * **Step-by-Step**

    1.  `percentile(filesize, percentiles=[40,80], as=score)`: Calculates the 40th and 80th percentiles for the `filesize` field, outputting them as `score_40` and `score_80`.
    2.  `| symbol := if(filesize > score_80, then=":+1:", else if(filesize > score_40, then="so-so", else=":-1:"))`: Compares whether the `filesize` is greater than `score_80`, setting `symbol` to `:+1:`. If not, it checks if `filesize` is greater than `score_40`, setting `symbol` to `so-so`. Otherwise, `symbol` is set to `:-1:`.
    3.  Event Result set.

  * **Summary and Results**
    Using `if()` is the best way to make conditional choices about values. The function has the benefit of being able to be embedded into other statements, unlike `case`.

---

### Show Percentiles Across Multiple Buckets

This example shows how to calculate response time percentiles over time by bucketing events into 1-minute intervals.

  * **Query Example**

    ```
    bucket(span=60sec, function=percentile(field=responsetime, percentiles=[50,75,99,99.9]))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `bucket(span=60sec, function=percentile(field=responsetime, percentiles=[50, 75, 99, 99.9]))`: Uses a 60-second timespan for each bucket and calculates the specified percentiles for the `responsetime` field within each bucket.
    3.  Event Result set.

  * **Summary and Results**
    The `percentile()` function quantifies values by determining whether the value is larger than a percentage of the overall values. The output provides a powerful view of the relative significance of a value. Combined in this example with `bucket()`, the query will generate buckets of data showing the comparative response time for every 60 seconds.

# range()

The `range()` function finds the numeric range between the smallest and largest numbers for the specified field over a set of events. The result is returned in a field named `_range`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `range` | Name of output field. |
| `field` | string | required | Field to extract a number from. |

***

## Function Operation

The `range()` function is an aggregation function that calculates the difference between the maximum and minimum numerical values of a specified field across a set of events. It always returns a single number representing this difference. The result is outputted into a new field, which by default is named `_range`, but can be customized using the `as` parameter. This function is particularly useful for performance analysis and identifying inconsistencies, as a small range indicates consistent performance, while a large range suggests reliability issues. It is commonly used in conjunction with `groupBy()` for comparative analysis across different categories or entities.

***

## Examples

### Find Range Between Smallest And Largest Numbers in Field

This example uses the `range()` function to find the range of values in the `responsetime` field.

  * **Query Example**

    ```
    range(responsetime)
    ```

  * **Input Event Data**

| timestamp | endpoint | responsetime |
| :--- | :--- | :--- |
| 2025-04-30T07:00:00Z | /api/users | 0.125 |
| 2025-04-30T07:00:01Z | /api/login | 2.543 |
| 2025-04-30T07:00:02Z | /api/data | 0.891 |
| 2025-04-30T07:00:03Z | /api/users | 1.234 |
| 2025-04-30T07:00:04Z | /api/search | 3.456 |
| 2025-04-30T07:00:05Z | /api/login | 0.567 |
| 2025-04-30T07:00:06Z | /api/data | 1.789 |
| 2025-04-30T07:00:07Z | /api/users | 0.234 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `range(responsetime)`: Finds the range of the values in the `responsetime` field and returns the result in a field named `_range`. The `range()` function always returns a single number (the difference between maximum and minimum).
    3.  Event Result set.

  * **Summary and Results**
    The query is used to calculate the difference between the highest and lowest values in the field `responsetime` across a set of events. Finding the range of `responsetime` in LogScale is particularly useful for performance analysis to identify performance inconsistencies. A small range indicates consistent performance, while a large range suggests reliability issues.

  * **Result Event Data**
    `_range`
    `3.331`

---

### Find Range of CPU Usage by Host

This example uses the `range()` function with `groupBy()` to find the CPU usage range for each host.

  * **Query Example**

    ```
    groupBy([host], function=range(cpu_usage))
    ```

  * **Input Event Data**

| timestamp | host | cpu\_usage |
| :--- | :--- | :--- |
| 2025-04-30T07:00:00Z | host1.com | 50 |
| 2025-04-30T07:01:00Z | host1.com | 75 |
| 2025-04-30T07:02:00Z | host1.com | 95 |
| 2025-04-30T07:03:00Z | host1.com | 65 |
| 2025-04-30T07:00:00Z | host2.com | 50 |
| 2025-04-30T07:01:00Z | host2.com | 70 |
| 2025-04-30T07:02:00Z | host2.com | 55 |
| 2025-04-30T07:03:00Z | host2.com | 65 |
| 2025-04-30T07:00:00Z | host3.com | 25 |
| 2025-04-30T07:01:00Z | host3.com | 100 |
| 2025-04-30T07:02:00Z | host3.com | 45 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy([host], function=range(cpu_usage))`: Groups events by host name (`host`), then calculates the range (the difference between the highest CPU usage value and the lowest CPU usage value) for each host, returning the results in a new field named `_range`. The `range()` function always returns a single number.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the CPU usage range by host. The smaller the range (e.g., 0-20), the more stable the system.

  * **Result Event Data**

| host | \_range |
| :--- | :--- |
| host1.com | 45 |
| host2.com | 20 |
| host3.com | 75 |

# rdns()

The `rdns()` function resolves hostnames using reverse DNS lookups.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `hostname` | Specifies the field into which the resolved value is stored. |
| `field` | string | required | Specifies the field to run the RDNS lookup against. |
| `server` | string | optional | Specifies a DNS server address. |

***

## Function Operation

The `rdns()` function performs reverse DNS lookups to resolve IP addresses into hostnames.

Key aspects of its operation include:

  * If a lookup fails, the event will be kept, but the designated output field will not be added.
  * The number of resulting events from this function is constrained by the `MAX_STATE_LIMIT` dynamic configuration parameter, which has a default limit of 20,000. If this limit is surpassed, the results will be truncated, and a warning will be issued.
  * To prevent the `rdns()` function from indefinitely blocking query execution, a timeout is applied to all RDNS requests. If a request does not yield a result within this timeout, the lookup is considered to have failed for the associated event.
  * However, if a request eventually returns after the timeout, its result is cached internally by LogScale for a certain period. Consequently, a static query using `rdns()` might fail a lookup on its initial execution but succeed in a subsequent execution.
  * In live queries, this behavior is less problematic as `rdns()` is continuously evaluated, making it preferable for use in such contexts.
  * Reverse DNS lookups are generally not considered authoritative; the owner of an IP address can change it to point to any arbitrary hostname. For an authoritative alternative without these limitations, consider using the `asn()` function instead.
  * If no RDNS server is explicitly specified, a system default is used, which can be overridden by the `server` parameter.

The function disallows reverse DNS lookups for specific IP address ranges (both IPv4 and IPv6), which are listed in the documentation. For self-hosted customers, the allowed IP addresses and servers for reverse DNS queries can be restricted using `IP_FILTER_RDNS` and `IP_FILTER_RDNS_SERVER` dynamic configuration parameters, respectively.

***

## Examples

### Resolve `ipAddress` using the server `8.8.8.8`, and store the resulting DNS name in `dnsName`

This example demonstrates resolving an IP address using a specific DNS server and storing the result in a custom field name.

  * **Query Example**

    ```
    rdns(ipAddress, server="8.8.8.8", as=dnsName)
    ```

  * **Step-by-Step**

    1.  The query targets an `ipAddress` field within events.
    2.  `rdns(ipAddress, server="8.8.8.8", as=dnsName)`: Performs a reverse DNS lookup on the `ipAddress` field. It uses the specified DNS server `8.8.8.8` to perform the lookup. The resolved hostname is then stored in a new field named `dnsName`.

  * **Summary and Results**
    This query is used to resolve an IP address to a hostname, specifying a particular DNS server and a custom output field name.

---

### Resolve `ipAddress` and store the resulting DNS name in `hostname`

This example demonstrates a basic reverse DNS lookup, storing the resolved hostname in the default output field.

  * **Query Example**

    ```
    rdns(ipAddress)
    ```

  * **Step-by-Step**

    1.  The query targets an `ipAddress` field within events.
    2.  `rdns(ipAddress)`: Performs a reverse DNS lookup on the `ipAddress` field. Since no `server` is specified, a system default DNS server is used. The resolved hostname is stored in the default output field, `hostname`.

  * **Summary and Results**
    This query performs a basic reverse DNS lookup on an IP address field, using default settings for the DNS server and output field name.

# readFile()

The `readFile()` function outputs the content of CSV lookup files or ad-hoc tables as events. This allows you to use a CSV Lookup File and ad-hoc table as data input.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `file` | array of file/table names | required | The names of the input files or input tables. You can use `table` as the parameter name instead of `file`. |
| `include` | array of strings | optional | Specifies the column names to read in the lookup file. If no argument is given, all columns are included. |
| `limit` | number | optional \<br\> minimum: `1` | Limits the number of rows returned. Use `limit=N` to preview the first N rows of the files and tables. The files or tables will be outputted in the specified order, until the limit has been reached. |

***

## Function Operation

The `readFile()` function is used to convert the content of CSV lookup files or ad-hoc tables into events, making them available for further query manipulation.

This function requires one of the following sources to be available:

  * An uploaded file.
  * An ad-hoc table defined in the query.
  * An installed package file.

When `readFile()` is used, it outputs each row of the specified file or table as a separate event. The order of output is as follows:

  * Files or tables are outputted in the order they are listed in the `file` or `table` parameter.
  * For each file or table, rows are outputted as events in their original order within the file or table.

It is recommended to use `readFile()` at the beginning of a query, as using it later will discard any events preceding it, returning only the content of the files or tables.

When reading a file from a package, the package name must be specified in addition to the filename (e.g., `readFile("falcon/investigate/logoninfo.csv")`).

For previewing large files, it is recommended to include the `limit` parameter for optimal UI performance (e.g., `readFile("host_names.csv", limit=5)`). However, if the file is intended as data input for further manipulation, the `limit` parameter can be omitted.

***

## Examples

### Perform a Right Join Query to Combine Two Datasets

This example uses the `defineTable()` function to create ad-hoc tables and then `readFile()` along with `match()` to perform a right join query, combining information from two different datasets.

  * **Query Example**

    ```
    defineTable(name="users", query={orgId=1}, include=[username, name])
    | defineTable(name="operations", query={*}, include=[username, operation])
    readFile(users)
    match(operations, field=username, strict=false)
    | select([username, operation])
    ```

  * **Input Event Data**
    **Users Table:**

    | username | name | orgId |
    | :--- | :--- | :--- |
    | user1 | John Doe | 1 |
    | user2 | Jane Doe | 1 |
    | user3 | Bob Smith | 2 |

    **Operations Table:**

    | username | operation |
    | :--- | :--- |
    | user1 | createdFile |
    | user3 | createdFile |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `defineTable(name="users", query={orgId=1}, include=[username, name])`: Generates an ad-hoc table named `users` that has the fields `username` and `name`, and includes users where the `orgId` field equals 1.
    3.  `| defineTable(name="operations", query={*}, include=[username, operation])`: Defines a new ad-hoc table named `operations` that uses all the fields (`username` and `operation`).
    4.  `| readFile(users)`: Reads the `users` ad-hoc table as events.
    5.  `| match(operations, field=username, strict=false)`: Matches the events from the `users` table with the `operations` table using `username` as the common field. Events are not filtered if they do not match (implying a right join) due to `strict=false`.
    6.  `| select([username, operation])`: Selects the `username` and `operation` fields to be displayed from the event set.
    7.  Event Result set.

  * **Summary and Results**
    The result will output two events. In the event set, all operations have been included even when there is no match between the `username` field, resulting in "no value" for `user2`. If `strict=true` had been used in the `match()` function, then the event for `user2` would not have been outputted.

  * **Result Event Data**

    | username | operation |
    | :--- | :--- |
    | user1 | createdFile |
    | user2 | no value |

---

### Preview Content in a Lookup File With `readFile()`

This example uses the `readFile()` function to preview the content of a `host_names.csv` file without needing to match it against existing data.

  * **Query Example**

    ```
    readFile("host_names.csv")
    ```

  * **Input Event Data**

    ```
    host_name, host_id
    DESKTOP-VSKPBK8, 1
    FINANCE, 2
    homer-xubuntu, 3
    logger, 4
    DESKTOP-1, 5
    DESKTOP-2, 6
    DESKTOP-3, 7
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `readFile("host_names.csv")`: Displays the content of the `.csv` file.
          * If you aim to preview the content of large files, it is recommended to always include the `limit` parameter for optimal UI performance (e.g., `readFile("host_names.csv", limit=5)`). However, if the file is used as data input for further manipulation, the `limit` parameter can be omitted.
          * If reading a file from a package, the package name should be specified in addition to the filename (e.g., `readFile("falcon/investigate/logoninfo.csv")`).
    3.  Event Result set.

  * **Summary and Results**
    The query is used to preview content in CSV Lookup Files. After previewing the content with `readFile()`, the data can be used for further manipulation, such as combining it with `count()` to count rows, `select()` to filter data, or `join()` to match data. The `readFile()` function can also read tables defined with `defineTable()`.

  * **Result Event Data (Full Content)**

    | host\_id | host\_name |
    | :--- | :--- |
    | 1 | DESKTOP-VSKPBK8 |
    | 2 | FINANCE |
    | 3 | homer-xubuntu |
    | 4 | logger |
    | 5 | DESKTOP-1 |
    | 6 | DESKTOP-2 |
    | 7 | DESKTOP-3 |

  * **Result Event Data (with `limit=5`)**

    | host\_id | host\_name |
    | :--- | :--- |
    | 1 | DESKTOP-VSKPBK8 |
    | 2 | FINANCE |
    | 3 | homer-xubuntu |
    | 4 | logger |
    | 5 | DESKTOP-1 |

---

### Preview Content in a Lookup File With `readFile()` and Filter With `join()`

This example previews content in a lookup file and then filters for specific data using the `join()` function with negation to find host names that do not send any logs.

  * **Query Example**

    ```
    readFile("host_names.csv")
    | !join(query={groupBy(host_name)}, field=host_name, key=host_name, include=[host_name, id])
    ```

  * **Input Event Data**

    ```
    host_name, host_id
    DESKTOP-VSKPBK8, 1
    FINANCE, 2
    homer-xubuntu, 3
    logger, 4
    DESKTOP-1, 5
    DESKTOP-2, 6
    DESKTOP-3, 7
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `readFile("host_names.csv")`: Displays the content of the `.csv` file.
    3.  `| !join(query={groupBy(host_name)}, field=host_name, key=host_name, include=[host_name, id])`: Filters for host names that do not send any logs by performing a negated join.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to preview content in CSV Lookup Files and then filter for host names that do not send any logs.

  * **Result Event Data**

    | host\_id | host\_name |
    | :--- | :--- |
    | 5 | DESKTOP-1 |
    | 6 | DESKTOP-2 |
    | 7 | DESKTOP-3 |

# regex()

The `regex()` function works both as a filter and can extract new fields using a regular expression. The regular expression can contain one or more named capturing groups. Fields with the names of the groups will be added to the events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | optional \<br\> default: `@rawstring` | Specifies the field to run the regular expression against. |
| `flags` | string | optional \<br\> default: `m` | Specifies regex modifier flags. \<br\>**Values:**\<br\>`d`: Period (`.`) also includes newline characters. \<br\>`i`: Ignore case for matched values. \<br\>`m`: Multi-line parsing of regular expressions. |
| `limit` | integer | optional \<br\> default: `100` | Defines the maximum number of events to produce. A warning is produced if this limit is exceeded, unless the parameter is specified explicitly. |
| `regex` | string | required | Specifies a regular expression. The regular expression can contain one or more named capturing groups. Fields with the names of the groups will be added to the events. |
| `repeat` | boolean | optional \<br\> default: `false` | If set to `true`, multiple matches yield multiple events. \<br\>**Values:**\<br\>`false`: Match at most one event. \<br\>`true`: Match multiple events. |
| `strict` | boolean | optional \<br\> default: `true` | Specifies if events not matching the regular expression should be filtered out of the result set. \<br\>**Values:**\<br\>`false`: Events not matching the regular expression are not filtered out when the regex matches. \<br\>`true`: Events not matching the regular expression are filtered out of the result set. |

***

## Function Operation

The `regex()` function serves a dual purpose: filtering events and extracting new fields based on regular expression patterns. Regular expressions in LogScale, using JitRex (similar to Java's regular expressions), are a fundamental part of the language for searching and extracting information.

When using `regex()`, care must be taken with escaping characters. A backslash character used for escaping must be double-escaped within the `regex()` function itself (e.g., `\\` for a literal backslash) because it needs to be escaped once for string definition and again for regex parsing. Alternatives to this double-escaping include using ASCII character codes (e.g., `\x5c` for backslash) or the `/regex/` syntax, which only requires single escaping as it is parsed once.

The `regex()` function, by default, searches only the specified `field` (or `@rawstring` if not specified), which can be significantly more performant than the `/regex/` syntax (free-text search across all fields).

The `flags` parameter allows for modifying regex behavior, such as `d` for dotall mode (dot matches newlines), `i` for case-insensitivity, and `m` for multi-line mode. The `g` (global) option is not supported within `flags`; for multiple matches, the `repeat` parameter should be set to `true`.

The `limit` parameter controls the maximum number of events produced. The `strict` parameter determines whether events that do not match the regular expression are filtered out (`true` by default) or kept in the result set with a non-match (`false`).

***

## Examples

### Extract the Top Most Viewed Pages of a Website

This example extracts the page names from URLs ending with `.page` and then lists the top most viewed pages.

  * **Query Example**

    ```
    regex(regex="/.*/(?<url_page>\S+\.page)", field=url)
    | top(url_page, limit=12, rest=others)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `regex(regex="/.*/(?<url_page>\S+\.page)", field=url)`: Extracts the page viewed by users by returning the name of the file from the `url` field and storing that result in a field labeled `url_page`.
    3.  `| top(url_page, limit=12, rest=others)`: Lists the top most viewed pages. The first parameter given is the `url_page` field. The second parameter limits the results to the top twelve instead of the default limit of ten. The `rest` parameter is specified with the label `others` to show how many pages were viewed that were not listed in the top twelve.
    4.  Event Result set.

  * **Summary and Results**
    The table displays the matches from the most viewed pages during the selected period to the least, limited to the top twelve.

  * **Result Event Data**

| url\_page | \_count |
| :--- | :--- |
| home.page | 51 |
| index.page | 21 |
| home-studio.page | 10 |
| a-better-digital-camera.page | 7 |
| is-film-better.page | 6 |
| leica-q-customized.page | 6 |
| student-kit.page | 4 |
| focusing-screens.page | 4 |
| changing-images-identity.page | 2 |
| others | 27 |

---

### Filter Out Based on a Non-Matching Regular Expression (Function Format)

This example searches weblog data for large log entries that are larger than a specified size but are not in a specific directory.

  * **Query Example**

    ```
    responsesize > 2000
    not regex("/falcon-logscale-.*/", field=url)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `responsesize > 2000`: Filters for events where the `responsesize` is greater than 2000.
    3.  `| not regex("/falcon-logscale-.*/", field=url)`: Negates the regular expression match, filtering out any filename that contains the prefix `falcon-logscale`, but returning all other matching URLs.
    4.  Event Result set.

  * **Summary and Results**
    This query can be used to identify large log entries that are not associated with a specific internal directory, potentially highlighting unusual or unmanaged data.

  * **Result Event Data**

| @timestamp | \#repo | \#type | @id |
| :--- | :--- | :--- | :--- |
| 2024-07-03T04:59:03 | weblogs | httpsimp | MqHKxw2QoBPZyNqbJRRs4ECC\_2\_654 |

---

### Filter Out Based on a Non-Matching Regular Expression (Syntax)

This example searches weblog data for events where the method does not match a specified value using the `/regex/` syntax with negation.

  * **Query Example**

    ```
    method != /(PUT|POST)/
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `method != /(PUT|POST)/`: This line performs a negative regular expression match, returning only the events where the `method` does not match either `PUT` or `POST`.
    3.  Event Result set.

  * **Summary and Results**
    This format of the query can be a simple way to perform a negative regular expression match, or more specifically, returning a list of the events that do not match the given regular expression.

---

### Get Integer Part of Number

This example uses regex pattern matching with a named capturing group to extract the integer part of a number from a field.

  * **Query Example**

    ```
    regex("(?<b>\\d+)\\..*",field=a)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `regex("(?<b>\\d+)\\..*",field=a)`: Looks for a sequence of characters in a capturing group and extracts the integer part of a number. Specifically, `(?<b>\\d+)` captures one or more digits into a named group `b`. `\\.` matches a literal dot, and `.*` matches any remaining characters. The original field `a` remains unchanged.
    3.  Event Result set.

  * **Summary and Results**
    The query with regex pattern matching and named capturing group is used to get the integer part of a number, storing the replacement (the matched value) automatically in a new field named `b`. This is useful when searching for specific filenames. The query using the `regex()` function is primarily used for pattern matching and extraction as regex is generally very concise for simple extraction tasks.
    An alternative method is to use the `replace()` function, e.g., `replace("(\\d+)\\..*", with="$1", field=a, as=b)`. This uses numbered references for substitution.

---

### Replace Word or Substring With Another

This example uses the `replace()` function with a regular expression to correct a spelling mistake in an event.

  * **Query Example**

    ```
    replace(regex=propperties, with properties)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace(regex=propperties, with properties)`: Replaces the word `propperties` with the word `properties` in the event.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to correct spelling mistakes in an event set. Changing words or other substrings like this with a regular expression is useful in many situations where it is necessary to make quick changes of field values.

---

### Search for Command Line String

This example searches for command line strings containing any characters after `/` and before `@` using a regular expression.

  * **Query Example**

    ```
    #event_simpleName=ProcessRollup2
    | CommandLine=/@/
    | CommandLine=/\/.*@/
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#event_simpleName=ProcessRollup2`: Filters for events of the type `ProcessRollup2` in the `#event_simpleName` field, performing as much initial filtering as possible.
    3.  `| CommandLine=/@/`: Filters for any command line containing the `@` symbol.
    4.  `| CommandLine=/\/.*@/`: Uses a regular expression to search the returned results for command lines that contain a forward slash (`/`) followed by any number of characters, and then an `@` symbol.
    5.  Event Result set.

  * **Summary and Results**
    The query is used to search for command line strings that contain any characters after `/` and before `@`. This query could, for example, be used to help security analysts identify potentially suspicious processes that might be interacting with email addresses or using email-like syntax in their command lines.

---

### Truncate a String or Message

This example uses the `replace()` function together with a regex capturing group to truncate a string or message to exactly 100 characters.

  * **Query Example**

    ```
    replace("^(.{100}).*", with="$1", field=message, as="truncated_message")
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace("^(.{100}).*", with="$1", field=message, as="truncated_message")`: Captures a group that matches exactly 100 characters of any type starting from the beginning of the line. `with="$1"` means that it replaces the entire match with the content of the first capturing group (the first 100 characters). The truncated version is returned in a new field named `truncated_message`. The original `message` field remains unchanged.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to truncate strings. Truncation can, for example, be used to speed up download times and complete searches faster. In file systems, the truncate operation is used to reduce the size of a file by removing data from the end. This can be helpful when you need to reclaim storage space or when dealing with log files that need to be periodically truncated. Another advantage of truncation is that it allows you to search for a word that could have multiple endings. This way it will broaden the results and look for variations of words. Truncation of numbers is also useful to shorten digits past a certain point in the number.

# rename()

This function renames one or more fields.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional | The new name of the field; it is used when a single field name is given in `field`. |
| `field` | string or array, array of arrays of strings | required | The field to rename, if a new field name is given in `as`. From v1.106.0, multiple fields can be given using an array of old/new field name pairs: `[[oldName1,newName1], [oldName2, newName2]]`. |

***

## Function Operation

The `rename()` function allows you to change the name of one or more fields within your event stream.

When a field is renamed to a name that already exists, the existing field and its content are overwritten by the new field and its content. This also occurs when renaming through field aliasing. Old fields are removed from the event stream after renaming, which can add overhead during processing. For more efficiency, especially if you want to retain the old field, copying to a new field using `newfield := oldfield` is more efficient, but it does retain the old field in the event set.

For single field renames, you can specify the `field` parameter (the original name) and the `as` parameter (the new name). For renaming multiple fields in a single operation, you can provide an array of arrays in the `field` parameter, where each inner array contains an `[oldName, newName]` pair.

***

## Examples

### Rename Fields

This example renames multiple fields to more readable names using the `rename()` function.

  * **Query Example**

    ```
    rename(field=[[src_ip, source_address], [dst_ip, destination_address], [src_port, source_port], [dst_port, destination_port]])
    ```

  * **Input Event Data**

    | timestamp | src\_ip | dst\_ip | src\_port | dst\_port | pro |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-04-01T07:00:00Z | 192.168.1.100 | 10.0.0.50 | 52431 | 443 | TCH |
    | 2025-04-01T07:00:01Z | 172.16.0.25 | 8.8.8.8 | 33221 | 53 | UD |
    | 2025-04-01T07-00-037 | 192.168.1.150 | 172.16.0.100 | 49223 | 80 | TCF |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `rename(field=[[src_ip, source_address], [dst_ip, destination_address], [src_port, source_port], [dst_port, destination_port]])`: Renames the fields `src_ip`, `dst_ip`, `src_port`, and `dst_port` to more readable field names. The original field names are replaced with the new field names. Since `field` is the unnamed parameter, the query could also be written as `rename([[src_ip, source_address], [dst_ip, destination_address], [src_port, source_port], [dst_port, destination_port]])`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to rename multiple fields in one single operation. Renaming of fields is used for standardization, normalization, and readability. Normalizing field names across different data sources is, for example, useful for joins. The `rename()` function is often used with the `table()` function. For renaming existing fields in arrays, refer to the documentation on "Rename Existing Fields in Array".

  * **Result Event Data (only showing renamed fields)**

    | destination\_address | destination\_port | source\_address | source\_port |
    | :--- | :--- | :--- | :--- |
    | 10.0.0.50 | 443 | 192.168.1.100 | 52431 |
    | 8.8.8.8 | 53 | 172.16.0.25 | 33221 |
    | 172.16.0.100 | 80 | 192.168.1.150 | 49223 |
    | 192.168.1.1 | 22 | 10.0.0.75 | 55678 |
    | 1.1.1.1 | 53 | 192.168.1.200 | 44556 |
    | 192.168.1.25 | 3389 | 172.16.0.50 | 51234 |
    | 10.0.0.100 | 445 | 192.168.1.75 | 48751 |
    | 172.16.0.75 | 8080 | 10.0.0.25 | 53992 |
    | 8.8.4.4 | 53 | 192.168.1.125 | 35667 |
    | 192.168.1.50 | 21 | 172.16.0.100 | 47891 |

---

### Rename a Single Field - Example 1

This example renames a single field using the `rename()` function with the `as` parameter to define the new name of the field.

  * **Query Example**

    ```
    rename(field=badName, as=goodName)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `rename(field=badName, as=goodName)`: Renames the field `badName` to `goodName`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to quickly rename single fields.

---

### Rename a Single Field - Example 2

This example renames a single field using the `rename()` function with assignment syntax to define the new name of the field.

  * **Query Example**

    ```
    goodName := rename(badName)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `goodName := rename(badName)`: Renames the `badName` field to `goodName` by assigning the new value (variable name) to the field. The value on the right side of the assignment operator is set equal to the value on the left side of it.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to quickly rename single fields.

# replace()

The `replace()` function replaces each substring of the specified field's value that matches the given regular expression with the given replacement. LogScale uses JitRex, which closely follows the syntax of re2j regular expressions, which has a syntax very close to Java's regular expressions.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: input field | Specifies the field to store the replaced string as. |
| `field` | string | optional \<br\> default: `@rawstring` | Specifies the field to run the replacement on. |
| `flags` | string | optional \<br\> default: `m` | Specifies other regex flags. \<br\>**Values:**\<br\>`d`: Period (`.`) includes newline characters. \<br\>`i`: Ignore case for matched values. \<br\>`m`: Multi-line parsing of regular expressions. |
| `regex` | string | required | The regular expression to match. |
| `replacement` | string | optional | The string to substitute for each match (same as `with`). |
| `with` | string | optional | The string to substitute for each match. |

***

## Function Operation

The `replace()` function is used to find and substitute substrings within a specified field's value. It operates by matching parts of the string against a provided regular expression (`regex`) and then replacing those matched parts with a `replacement` string (or `with` string, which is an alias). The function uses JitRex regular expressions, which are similar to Java's regex syntax.

Key aspects of its operation include:

  * The `field` parameter specifies which field to apply the replacement to; by default, it operates on `@rawstring`.
  * The `as` parameter dictates the name of the output field where the modified string will be stored. If not specified, the original `field` will be overwritten.
  * The `flags` parameter allows for modifying the regex behavior, such as `d` (dotall mode, where `.` matches newlines), `i` (case-insensitive matching), and `m` (multi-line mode).

This function is highly versatile for string manipulation and transformation tasks, such as correcting spelling mistakes, truncating strings, or extracting specific patterns and reformatting them.

***

## Examples

### Get Integer Part of Number

This example uses the `regex()` function (mentioned as an alternative in the `replace()` documentation) and regex capturing groups to extract the integer part of a number. The documentation also provides an alternative using `replace()` which is more relevant to this function's scope.

  * **Query Example (using `replace()` as suggested alternative)**

    ```
    replace("(\\d+)\\..*", with="$1", field=a, as=b)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace("(\\d+)\\..*", with="$1", field=a, as=b)`: This query uses `replace()` to capture digits before a decimal point in an unnamed group (`(\d+)`) and replaces the entire matched string with the content of this captured group (`$1`). The result is stored in a new field named `b`, leaving the original field `a` unchanged.

  * **Summary and Results**
    This query, using `replace()`, is particularly suited for string manipulation and transformation. It captures the integer part of a number from a field and stores it in a new field.

---

### Replace Word or Substring With Another

This example uses the `replace()` function with a regular expression to correct a spelling mistake.

  * **Query Example**

    ```
    replace(regex=propperties, with properties)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace(regex=propperties, with properties)`: Replaces all occurrences of the word "propperties" with "properties" in the target field.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to correct spelling mistakes in an event set. Changing words or other substrings like this with a regular expression is useful in situations where quick modifications of field values are needed.

---

### Truncate a String or Message

This example uses the `replace()` function together with regex capturing groups to truncate a string or message to exactly 100 characters.

  * **Query Example**

    ```
    replace("^(.{100}).*", with="$1", field=message, as="truncated_message")
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace("^(.{100}).*", with="$1", field=message, as="truncated_message")`: This captures a group that matches exactly 100 characters of any type starting from the beginning of the line (`^(.{100})`). The `.*` matches any remaining characters. `with="$1"` means that the entire matched string (the whole original message) is replaced with the content of the first capturing group (the first 100 characters). The truncated version is returned in a new field named `truncated_message`. The original `message` field remains unchanged.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to truncate strings. Truncation can be beneficial for speeding up download times and completing searches faster. In file systems, truncation reduces file size by removing data from the end, useful for reclaiming storage space or managing log files. Additionally, it can broaden search results by allowing searches for variations of words (e.g., via wildcards on truncated terms) and can shorten numerical digits past a certain point.

# reverseDns()

The `reverseDns()` function performs reverse DNS lookups to find hostnames associated with IP addresses. It adds hostname information to events by querying DNS servers.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `hostname` | Specifies the field into which the resolved value is stored. |
| `field` | string | required | Specifies the field to run the RDNS lookup against. |
| `limit` | number | optional \<br\> default: `ReverseDnsDefaultLimit` | Maximum number of unique IPs to process. If the input events contain more unique IP addresses than the limit allows, a warning is issued and the rest of the events are not annotated with hostname information. The default is the value of the dynamic configuration `ReverseDnsDefaultLimit` (or 5000 if not set). |
| `server` | string | optional | Name or address of the DNS server to use. The default is the value of the configuration value `RDNS_DEFAULT_SERVER` or as otherwise configured in the system. |

***

## Function Operation

The `reverseDns()` function is used to perform reverse DNS lookups, resolving IP addresses to their corresponding hostnames.

Key operational characteristics and limitations include:

  * It functions only in the result phase of a query (after the first aggregator) and only at the top-level.
  * It cannot be used in subqueries.
  * It processes a limited number of results.
  * It cannot run in filter alerts and aggregate alerts; for alerts, scheduled searches with appropriate actions should be used instead.
  * The function attempts to resolve as many IP addresses as possible within specified limits, such as the maximum number of addresses or a timeout period.
  * If resolution fails for any addresses (e.g., no response before timeout, too many addresses), it emits a warning and returns events without hostname information.
  * The `reverseDns()` function uses an external network service for resolution, which means it can be affected by DNS server or network unreliability, or by DNS server throttling/blocking of the cluster's IP range. Therefore, LogScale does not recommend using the annotated hostname for filtering.
  * There is a cluster-wide rate limit (default 1000 requests per second, distributed among nodes) on the number of requests, which can be controlled by `ReverseDnsRequestsPerSecond`.
  * The number of concurrent requests a node can make is also limited (default 10), controlled by `ReverseDnsConcurrentRequests`.
  * If a lookup fails, the event is kept without modification. To exclude such events, a field test like `hostname=*` can be used after the function.
  * A timeout is applied to prevent indefinite blocking of query execution. If a result isn't received from the DNS server before the timeout (default 5 seconds), the hostname is not annotated. This timeout is controlled by `ReverseDnsDefaultTimeoutInMs`.
  * The number of unique IP addresses resolved per query is limited by the `limit` parameter, the timeout, and the node-wide rate limit.
  * Reverse DNS is generally informational, not authoritative, as IP owners can change hostname mappings. For an authoritative alternative, `asn()` is suggested.
  * If no DNS server is specified, the `RDNS_DEFAULT_SERVER` environment variable is used; otherwise, the system default DNS server is used.
  * The function caches resolved hostnames for a limited time to reduce external calls and provide faster results.
  * The function has a default list of disallowed IP address ranges (IPv4 and IPv6) that it will not resolve.
  * For self-hosted customers, allowed IP addresses and servers for reverse DNS queries can be restricted by `IP_FILTER_RDNS` and `IP_FILTER_RDNS_SERVER`.

***

## Examples

### Resolve the hostname for the IP address in the field `ip`, using the default DNS server. The aggregator function `tail()` is used to restrict the number of events.

  * **Query Example**

    ```
    tail(100)
    | reverseDns(ip)
    ```

  * **Step-by-Step**

    1.  `tail(100)`: Restricts the number of events to the last 100, ensuring `reverseDns()` operates on a limited dataset.
    2.  `| reverseDns(ip)`: Performs a reverse DNS lookup on the `ip` field for the limited set of events, using the default DNS server.

---

### Using `groupBy()` instead of `tail()` to restrict events for `reverseDns()`

  * **Query Example**

    ```
    groupBy(ip, function=[], limit=100)
    reverseDns(ip)
    ```

  * **Step-by-Step**

    1.  `groupBy(ip, function=[], limit=100)`: Groups events by `ip` and limits the number of groups to 100, effectively restricting the unique IPs processed by `reverseDns()`.
    2.  `reverseDns(ip)`: Performs the reverse DNS lookup on the unique `ip` values.

---

### Specifying a custom DNS server when using `groupBy()`

  * **Query Example**

    ```
    groupBy(ip, function=[], limit=100)
    reverseDns(ip, server=8.8.8.8)
    ```

  * **Step-by-Step**

    1.  `groupBy(ip, function=[], limit=100)`: Groups events by `ip` and limits the number of groups.
    2.  `reverseDns(ip, server=8.8.8.8)`: Performs the reverse DNS lookup on `ip` using `8.8.8.8` as the specified DNS server.

---

### Limiting the number of unique IPs processed by `reverseDns()`

  * **Query Example**

    ```
    tail(100)
    reverseDns(ip, limit=10)
    ```

  * **Step-by-Step**

    1.  `tail(100)`: Restricts the input events to the last 100.
    2.  `reverseDns(ip, limit=10)`: Performs reverse DNS lookups on the `ip` field, but only processes a maximum of 10 unique IP addresses from the input.

# round()

The `round()` function rounds a numeric input field to the nearest integer, with an optional method to set the rounding type.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: Same name as input field | The output name of the field to round. |
| `field` | string | required | The name of the field to round. |
| `how` | string | optional \<br\> default: `round` | Method used to round the number. \<br\>**Values:**\<br\>`ceil`: Round up to the nearest whole number. \<br\>`floor`: Round down to the nearest whole number. \<br\>`round`: Standard rules; for example, \<0.5 rounds down, \>0.5 rounds up. |

***

## Function Operation

The `round()` function is used to convert a numerical value in a specified field to the nearest integer. It offers three methods for rounding, controlled by the `how` parameter:

  * **`round` (default)**: Applies standard mathematical rounding rules. Values with a fractional part of 0.5 or greater are rounded up, while those less than 0.5 are rounded down.
  * **`ceil`**: Always rounds the number up to the smallest integer that is greater than or equal to the original value.
  * **`floor`**: Always rounds the number down to the largest integer that is less than or equal to the original value.

The result of the rounding operation is outputted to a new field. By default, this output field retains the same name as the input field. However, a custom name can be assigned using the `as` parameter. This function simplifies numbers, making them easier to work with, especially for display or comparison purposes. To round to a specific decimal accuracy (rather than a whole number), the `format()` function should be used instead.

***

## Examples

### Basic Rounding

This example uses the `round()` function to round a number to the nearest integer using standard math rules.

  * **Query Example**

    ```
    round(myvalue)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `round(myvalue)`: Rounds the number in `myvalue` using standard rounding rules (numbers greater than 0.5 are rounded up, numbers lower than 0.5 are rounded down).
    3.  Event Result set.

  * **Summary and Results**
    The query is used to round a floating-point number to the nearest integer. Rounding is used to simplify numbers, making them easier to work with. To format a number or round to a specific decimal accuracy, use `format()` (e.g., `format("%.2f", field=value)`).

---

### Rounding Within a Timechart

This example uses the `round()` function with the `floor` parameter to round down a field value to an integer and display information within a timechart.

  * **Query Example**

    ```
    timeChart(function={max(value) | round(_max, how=floor)})
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `timeChart(function={max(value) | round(_max, how=floor)})`: Creates a time chart using `max()` as the aggregate function for a field named `value` to find the highest value in each time bucket, and returns the result in a field named `_max`. It then rounds the implied field `_max` from the aggregate `max()` using the `floor` option to round down the value. For example, if original `_max` values were 10.8, 15.3, 20.7, after floor rounding they would be 10, 15, 20.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to round down maximum values over time to the nearest integer (whole value). This is useful when displaying information in a time chart. Rounding to the nearest integer will make it easier to distinguish the differences between values when used on a graph for time-based visualization. The query simplifies the data presentation. To round to a specific decimal accuracy, the `format()` function must be used.

---

### Rounding to n Decimal Places

This example (from the document but explicitly stating `format()` is used instead of `round()`) shows how to round a number to a specific number of decimal places.

  * **Query Example**

    ```
    format("%.2f", field=value)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `format("%.2f", field=value)`: Rounds the `field` value to two decimal places.
    3.  Event Result set.

  * **Summary and Results**
    When using `format()`, rounding is performed using standard math rules. The `format()` function rounds a number to a specific decimal accuracy. To round a number to the nearest integer, `round()` should be used.

# sample()

The `sample()` function samples the event stream. Events that do not have the field being sampled are discarded.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | optional \<br\> default: `@timestamp` | The name of the field to use for sampling events. |
| `percentage` | double | optional \<br\> default: `1` \<br\> maximum: `100` | Keep this percentage of the events. |

***

## Function Operation

The `sample()` function is used to randomly select and retain a specified percentage of events from the event stream. This is particularly useful for analyzing characteristics of large datasets without processing every single event, thereby improving query performance and reducing resource usage.

Key aspects of its operation include:

  * Events that do not possess the field specified for sampling are discarded from the stream.
  * The `percentage` parameter (defaulting to 1%) determines what proportion of events to keep; it accepts values between \>0 and 100.
  * The `field` parameter (defaulting to `@timestamp`) indicates which field to use as the basis for sampling.

When used as part of a query, the randomly selected events are passed to the next stage of the query pipeline. Sampling can be applied before aggregations (like `groupBy()`) to analyze common patterns efficiently without hitting aggregation limits. It is useful for survey analysis (drawing conclusions without surveying all events) and for filtering both frequently and infrequently occurring events.

***

## Examples

### Sample Event Streams - Example 1

This example uses the `sample()` function to keep only a specified percentage of events.

  * **Query Example**

    ```
    sample(percentage=2)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `sample(percentage=2)`: Samples events, keeping only 2% of them. These randomly selected events are passed to the next stage of the query.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to sample events, keeping only a specified percentage of them. Event sampling can be used to determine the characteristics of a large set of data without processing every event. Sampling is useful in, for example, survey analysis, making it possible to draw conclusions without surveying all events. Sampling can also be used to filter on both frequently and infrequently occurring events.

---

### Sample Event Streams - Example 2

This example uses the `sample()` function to keep a specified percentage of events and then sorts them by host using `groupBy()` and `sort()`.

  * **Query Example**

    ```
    sample(percentage=0.1)
    | groupBy(host)
    | sort()
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `sample(percentage=0.1)`: Samples events, keeping only 0.1% of them. These randomly selected events are passed to the next stage of the query.
    3.  `| groupBy(host)`: Groups the sampled events by the `host` field. The advantage of sampling events before grouping them is that it allows for analysis of common patterns without hitting `groupBy()` limits.
    4.  `| sort()`: Sorts the returned results (by default, in descending order of count) to find the most common hosts.
    5.  Event Result set.

  * **Summary and Results**
    The query is used to sample events, keeping only a specified percentage of them, and then find the most common host among the sampled events. Event sampling can be used to determine the characteristics of a large set of data without processing every event. Sampling is useful in, for example, survey analysis, making it possible to draw conclusions without surveying all events. Sampling can also be used to filter on both frequently and infrequently occurring events.

# sankey()

A companion function to the Sankey that produces data compatible with the widget. For more information about Sankey widgets, see Sankey Diagram Widget.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `source` | string | required | The field containing the source node ID. |
| `target` | string | required | The field containing the target node ID. |
| `weight` | aggregate | optional \<br\> default: `count(as=_count)` | A function used to calculate the weight of the edges. Good candidates are functions like, for example, `sum()`, `count()` or `max()`. |

***

## Function Operation

The `sankey()` function is designed to prepare data in a format suitable for rendering Sankey diagrams in a widget. Sankey diagrams are powerful data visualization tools used to represent the flow of values between stages, offering valuable insights into data movement. They are particularly useful for visualizing complex processes, highlighting a single aspect or resource, and revealing data inconsistencies.

To produce Sankey-compatible data, the `sankey()` function requires at least two fields:

  * `source`: Specifies the field that contains the source node IDs.
  * `target`: Specifies the field that contains the target node IDs.

These two fields define the "flow" or connection between different entities.

Additionally, an optional `weight` parameter can be provided. This parameter takes an aggregate function (such as `sum()`, `count()`, or `max()`) to calculate the magnitude or quantity of the flow between the source and target nodes. This calculated weight determines the thickness of the edges in the Sankey diagram. If no `weight` function is specified, `count(as=_count)` is used by default.

***

## Examples

### Create Data Compatible With Sankey Diagram Widget - Example 1

This example uses the `sankey()` function to produce Sankey-compatible data for a web server log, showing which URLs match the HTTP methods.

  * **Query Example**:

    ```
    type=accesslog
    sankey(source=method, target=url)
    ```

  * **Input Event Data**:

    | type | timestamp | method | url |
    | :--- | :--- | :--- | :--- |
    | accesslog | 2025-05-02 11:48:15 | GET | /api/cart |
    | accesslog | 2025-05-02 11:48:45 | POST | /api/login |
    | accesslog | 2025-05-02 11:49:01 | GET | /api/logout |
    | accesslog | 2025-05-02 11:49:30 | GET | /api/users |
    | accesslog | 2025-05-02 11:50:00 | POST | /api/products |
    | accesslog | 2025-05-02 11:50:23 | PUT | /api/products |
    | accesslog | 2025-05-02 11:50:45 | GET | /api/users |
    | accesslog | 2025-05-02 11:51:12 | POST | /api/cart |
    | accesslog | 2025-05-02 11:51:33 | GET | /api/users |
    | accesslog | 2025-05-02 11:52:01 | GET | /api/products |
    | accesslog | 2025-05-02 11:52:30 | POST | /api/orders |
    | accesslog | 2025-05-02 11:53:00 | DELETE | /api/products |
    | accesslog | 2025-05-02 11:53:15 | GET | /api/dashboard |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `type=accesslog`: Filters for all web server logs. Web server logs provide valuable business insight as they list all requests for individual files that users have made from a website.
    3.  `sankey(source=method, target=url)`: Produces Sankey-compatible data for a web server log, showing which URLs match the HTTP methods. It shows how traffic is distributed across different endpoints. Note that `source` and `target` are required fields to be able to produce a Sankey Diagram.
    4.  Event Result set.

  * **Summary and Results**:
    The query is used to produce data compatible with Sankey diagram widgets, in this example visualizing the relationship between HTTP methods and URLs. Sankey diagram widgets are a powerful data visualization tool used to represent the flow of values between stages, offering valuable insights into the flow of data. When used with well-structured data, they can help identify patterns, bottlenecks, and significant resource allocation trends. Sankey diagram widgets are useful if you want to show complex processes visually, with a focus on a single aspect or resource required to be highlighted. Sankey diagram widgets can also be used to reveal inconsistent data, as the visualization of data inconsistencies makes detection easier.

---

### Create Data Compatible With Sankey Diagram Widget - Example 2

This example uses the `sankey()` function to produce Sankey-compatible data for a web server log, showing the URL address (the referrer) of the website that sends users to another website using a link.

  * **Query Example**:

    ```
    Type=accesslog referrer!="-"
    sankey(source=referrer, target=url)
    ```

  * **Input Event Data**:

    | Type | timestamp | method | url | referrer |
    | :--- | :--- | :--- | :--- | :--- |
    | accesslog | 2025-05-02 11:45:23 | GET | /api/users | `http://example.com/login` |
    | accesslog | 2025-05-02 11:45:30 | POST | /api/users | `http://example.org/signup` |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `Type=accesslog referrer!="-"`: Filters for all web server logs where the `referrer` is not equal to `-`. This returns all results where the value is not `-`. The value `-` is often used by analytics to define an empty value. In this example, we therefore filter for web server logs where there is referrer information.
    3.  `sankey(source=referrer, target=url)`: Produces Sankey-compatible data for a web server log, showing which target URLs match the referrer URL. If one URL links to the same website twice, the count of referring pages for that website will be one and the count of backlinks will be two. Note that `source` and `target` are required fields to be able to produce a Sankey Diagram.
    4.  Event Result set.

  * **Summary and Results**:
    The query is used to produce data compatible with Sankey diagram widgets, showing the referrer URL. The query is useful to get an overview of the web page that a user was on right before they landed on your page, the target (it shows how users navigate to different URLs on your site).

---

### Create Sankey Diagram Calculating Edge Thickness

This example creates a Sankey diagram calculating the edge thickness using the `sankey()` function with an aggregator.

  * **Query Example**:

    ```
    sankey(source="src", target="dst", weight=(sum(cnt)))
    ```

  * **Input Event Data**:

    | cnt | dst | src |
    | :--- | :--- | :--- |
    | 12 | apples | john |
    | 1 | bananas | john |
    | 1 | apples | joe |
    | 1 | apples | sarah |
    | 1 | apples | sarah |
    | 1 | apples | sarah |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `sankey(source="src", target="dst", weight=(sum(cnt)))`: Creates a Sankey diagram showing the sources on the left side (john, joe, sarah), and the targets on the right side (apples, bananas), and then provides the edge thickness based on the sum of `cnt`. The default `weight` parameter is `count()`.
    3.  Event Result set.

  * **Summary and Results**:
    The query is used to create a Sankey diagram showing the sources on the left side (john, joe, sarah), and the targets on the right side (apples, bananas), and then display the edge thickness using the `weight` parameter. In this example, the thickest edge is `john -> apples` (12), the medium edge is `sarah -> apples` (3), and the thin edges are `joe -> apples` (1) and `john -> bananas` (1). The query is useful for rendering results as a two-level Sankey diagram and visualizing flow relationships between nodes. It shows proportional relationships between data categories.

  * **Result Event Data**:

    | source | target | weight |
    | :--- | :--- | :--- |
    | joe | apples | 1 |
    | john | apples | 12 |
    | john | bananas | 1 |
    | sarah | apples | 3 |

# select()

The `select()` function specifies a set of fields to select from each event and include in the resulting event set. It is possible that an aggregate function, such as `table()` or `groupBy()` may be more suitable for summarizing and selecting the fields that you want to be displayed. A use-case for `select()` is when you want to export a few fields from a large number of events into a CSV file without aggregating the values. Because an implicit `tail(200)` function is appended in non-aggregating queries, only 200 events might be shown in those cases; however, when exporting the result, you get all matching events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `fields` | array of strings | required | The names of the fields to keep. |

***

## Function Operation

The `select()` function is used to narrow down the fields included in each event within the result set. It allows you to specify exactly which fields you want to retain, discarding all others. While aggregate functions like `table()` or `groupBy()` might be more appropriate for summarizing and displaying specific fields, `select()` is particularly useful for scenarios where you need to export a subset of fields from a large number of events to a CSV file without performing any aggregation.

When using `select()` in non-aggregating queries within the LogScale UI, an implicit `tail(200)` function is appended, meaning only the last 200 events might be displayed. However, when exporting the results, all matching events with the selected fields will be included in the CSV file, regardless of the UI's display limit. The `fields` parameter, which can be omitted if the context clearly defines the fields (e.g., in a pipeline where fields are directly assigned), is required to specify the names of the fields to keep.

***

## Examples

### Reduce Large Event Sets to Essential Fields

This example reduces a large dataset to essential fields using the `select()` function, focusing on `statuscode` and `responsetime` for `GET` requests.

  * **Query Example**

    ```
    method=GET
    select([statuscode, responsetime])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `method=GET`: Filters for all HTTP request methods of the type `GET`.
    3.  `select([statuscode, responsetime])`: Creates an unsorted table showing only the `statuscode` field and the `responsetime` field for the filtered events.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to filter specific fields from an event set and create a table showing these focused fields. In this example, the table shows the HTTP response status and the time taken to respond to the request, which is useful for analyzing HTTP performance, monitoring response codes, and identifying slow requests. The `select()` function is useful when you want to export a few fields from a large number of events into a CSV file without aggregating the values. Note that while the LogScale UI may only show up to 200 events, the exported CSV file will contain all results.

---

### Select Fields to Export

This example demonstrates how to select specific fields (`@timestamp` and `@rawstring`) for export as a CSV file.

  * **Query Example**

    ```
    select([@timestamp, @rawstring])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `select([@timestamp, @rawstring])`: Creates an unsorted table showing the `@timestamp` field and the `@rawstring` field for each event.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to filter specific fields from an event set and create a table showing these fields (focused event set). In this example, the table shows the timestamp of the events and the complete raw log entry, which is useful for full log analysis and data backup. The `select()` function is useful when you want to export a few fields from a large number of events into a CSV file without aggregating the values. Note that while the LogScale UI can only show up to 200 events, an exported CSV file contains all results.

# selectFromMax()

The `selectFromMax()` function identifies the event with the maximum value in a specified field and returns selected fields from that event. The resulting event contains only the fields specified in the `include` parameter. If multiple events share the same maximum value, the `selectFromMax()` function returns one of those events randomly (non-deterministic way).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | required | The name of the field that is used to find the maximum value. |
| `include` | array of strings | required | The names of the fields to include in the generated event. |

***

## Function Operation

The `selectFromMax()` function is designed to efficiently find and return specific fields from the event that contains the maximum value of a designated field. It processes events to identify the single event (or one of them if there are ties) with the highest value in the `field` parameter. The function then constructs a new event containing only the fields specified in its `include` parameter.

This function is particularly efficient for finding the "latest" or "newest" value of a field when using a timestamp field (like `@timestamp`) as the basis for finding the maximum. It avoids the need for sorting all results or using other aggregation functions, as it directly selects the most recent matching event. Timestamps are typically stored as numerical values (often in Unix epoch format), where larger numbers represent more recent times.

If multiple events share the same maximum value for the specified `field`, `selectFromMax()` will return one of those events randomly, meaning the selection is non-deterministic.

***

## Examples

### Find Most Recent (Latest) Value of Field X

This example uses the `selectFromMax()` function to find the most recent (latest) value of a field `x` and return the timestamp when that value was recorded.

  * **Query Example**

    ```
    selectFromMax(@timestamp, include=[x, @timestamp])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selectFromMax(@timestamp, include=[x, @timestamp])`: Sorts all events by timestamp, then selects the event in field `x` with the highest (most recent) timestamp, returning only the specified fields `x` and `@timestamp`. In this example, `selectFromMax()` filters for the "maximum value" of `@timestamp`, and finds the event with the newest/latest timestamp in the event set that also contains the specified field `x`. The `include` parameter is used to specify which fields to include in the output.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the most recent value of field `x` by selecting the event with the highest (most recent) timestamp value. Using this query is an efficient way to find the latest value since it does not require sorting all results or using other aggregation functions - the query directly selects the most recent matching event.

# selectFromMin()

The `selectFromMin()` function identifies the event with the minimum value in a specified field and returns selected fields from that event. The resulting event contains only the fields specified in the `include` parameter. If multiple events share the same minimum value, the `selectFromMin()` function returns one of those events randomly (non-deterministic way).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | required | The name of the field that is used to find the minimum value. |
| `include` | array of strings | required | The names of the fields to include in the generated event. |

***

## Function Operation

The `selectFromMin()` function efficiently identifies and extracts specific fields from the event that possesses the minimum value of a designated field. It processes events to find the single event (or one of them if there are multiple events with the same minimum value) that has the lowest value in the `field` parameter. A new event is then constructed containing only the fields specified in the `include` parameter.

This function is particularly efficient for retrieving the "oldest" or "first" value of a field, especially when the `@timestamp` field is used as the basis for finding the minimum. It eliminates the need for sorting all results or using other aggregation functions, as it directly selects the event with the earliest timestamp that also contains the specified field. Timestamps are typically stored as numerical values (often in Unix epoch format), where lower numbers indicate older times.

If multiple events share the same minimum value for the specified `field`, `selectFromMin()` will return one of those events randomly, making its selection non-deterministic in such cases.

***

## Examples

### Find Oldest (First) Value of Field X

This example uses the `selectFromMin()` function to find the oldest (first) value of a field `x` and return the timestamp when that value was recorded.

  * **Query Example**

    ```
    selectFromMin(@timestamp, include=[x, @timestamp])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selectFromMin(@timestamp, include=[x, @timestamp])`: Sorts all events by timestamp, then selects the event in field `x` with the oldest (lowest) timestamp, returning only the specified fields `x` and `@timestamp`. In this example, `selectFromMin()` filters for the "minimum value" of `@timestamp`, and finds the event with the oldest/first timestamp in the event set that also contains the specified field `x`. Timestamps are typically stored as numerical values (often in Unix epoch format), where lower numbers represent older times. The `include` parameter is used to specify which fields to include in the output.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the oldest value of field `x` by selecting the event with the lowest (oldest) timestamp value. Using this query is an efficient way to find the first value since it does not require sorting all results or using other aggregation functions - the query directly selects the oldest matching event.

# selectLast()

The `selectLast()` function specifies a set of fields to select from events; for each field it will keep the field value of the most recent event with that field. This can be used to collect field values across a range of events, where each event contributes one or more fields to the output event. It is usually most useful in combination with `groupBy()`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `fields` | array of strings | required | The names of the fields to keep. |

***

## Function Operation

The `selectLast()` function identifies and extracts the values of specified fields from the most recent event (based on timestamp) within a group or across the entire event stream. Its primary purpose is to consolidate information by retaining only the latest state or value for each unique combination of grouping keys.

When used, `selectLast()` processes events and, for each field listed in its `fields` parameter, it takes the value from the event with the latest timestamp where that field is present. If multiple events share the same latest timestamp, one is chosen non-deterministically. This function is particularly effective when combined with `groupBy()`, as it allows you to retrieve the most up-to-date attributes for each distinct group (e.g., the latest status for each server ID).

It's important to note the equivalences:

  * `selectLast([@timestamp])` is equivalent to `max(@timestamp)`.
  * The logical opposite (selecting the first matching event) for `@timestamp` can be achieved using `min(@timestamp)`. For arbitrary arrays of values, achieving the opposite of `selectLast([from, to])` would require a more complex query involving `head(1)` for each field.

***

## Examples

### Find the first value of a field `x` (and when that value was from)

This example demonstrates how `selectLast()` works with `@timestamp` to find the most recent value and its timestamp.

  * **Query Example**

    ```
    selectLast(@timestamp, include=[x, @timestamp])
    ```

  * **Step-by-Step**

    1.  This query uses `@timestamp` as the field to find the maximum value, effectively sorting by time to get the most recent event.
    2.  It then returns the `x` field and the `@timestamp` of that most recent event.

  * **Summary and Results**
    This selects the event with the minimum value of `@timestamp` that also contains the specified field `x`, and returns an event with fields `@timestamp` and `x` only.

---

### List All EC2 Hosts With FirstSeen Data Within 14 Days

This example retrieves the latest information about AWS EC2 instances running CrowdStrike sensors, showing their platform, hostname, agent version, and when they were first seen, with a 14-day reference point for age comparison.

  * **Query Example**

    ```
    #repo=sensor_metadata #data_source_name=aidmaster cloud.provider = "AWS_EC2_V2"
    | groupBy([aid], function=(selectLast([event_platform, aid, ComputerName, AgentVersion, FirstSeen])), limit=max)
    | FirstSeen := formatTime("%FT%T%z", field=FirstSeen)
    | TimeDelta := now() - duration("14d")
    ```

  * **Input Event Data**

    | @timestamp | aid | cloud.provider | event\_platform | ComputerName | AgentVersion | FirstSeen |
    | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-05-20T10:00:00Z | 1234abcd | AWS\_EC2\_V2 | Windows | ec2-web-01 | 6.45.15679 | 2025-05-20T10:00:00Z |
    | 2025-05-21T11:00:00Z | 1234abcd | AWS\_EC2\_V2 | Windows | ec2-web-01 | 6.45.15679 | 2025-05-20T10:00:00Z |
    | 2025-05-22T12:00:00Z | 5678efgh | AWS\_EC2\_V2 | Linux | ec2-app-02 | 6.45.15679 | 2025-05-22T12:00:00Z |
    | 2025-05-23T13:00:00Z | 5678efgh | AWS\_EC2\_V2 | Linux | ec2-app-02 | 6.45.15679 | 2025-05-22T12:00:00Z |
    | 2025-05-24T14:00:00Z | 90123ijk | AWS\_EC2\_V2 | Windows | ec2-db-03 | 6.45.15679 | 2025-05-24T14:00:00Z |
    | 2025-05-25T15:00:00Z | 90123ijk | AWS\_EC2\_V2 | Windows | ec2-db-03 | 6.45.15679 | 2025-05-24T14:00:00Z |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#repo=sensor_metadata #data_source_name=aidmaster cloud.provider = "AWS_EC2_V2"`: Searches in the `SENSOR_METADATA` repository and filters for `#data_source_name` fields containing the value `aidmaster`, looking for `cloud.provider` of the type `AWS_EC2_V2` only.
    3.  `| groupBy([aid], function=(selectLast([event_platform, aid, ComputerName, AgentVersion, FirstSeen])), limit=max)`: Groups results by the field `aid` (Agent ID). Then, for each unique group, it selects the most recent values for the fields: `event_platform`, `aid`, `ComputerName`, `AgentVersion`, `FirstSeen`. Using `selectLast()` within `groupBy()` is what actually selects the most recent record for each group.
    4.  `| FirstSeen := formatTime("%FT%T%z", field=FirstSeen)`: Formats the timestamp in the `FirstSeen` field into ISO 8601 format. The result is stored back in the `FirstSeen` field.
    5.  `| TimeDelta := now() - duration("14d")`: Calculates a timestamp from 14 days ago and returns the results into a new field named `TimeDelta`. The calculation is done by subtracting a 14-day duration from the current time using `duration()`. This new `TimeDelta` field represents a timestamp from 14 days ago and can be used for filtering or comparing against the `FirstSeen` timestamps.
    6.  Event Result set.

  * **Summary and Results**
    The query is used to retrieve the latest information about AWS EC2 instances running CrowdStrike sensors, showing their platform, hostname, agent version, and when they were first seen, with a 14-day reference point for age comparison. The query is useful, for example, for auditing EC2 instance coverage, identifying newly added EC2 instances within the last two weeks, monitoring sensor versions, or identifying aging or outdated installations. Each `aid` appears only once with its most recent values.

  * **Result Event Data**

    | aid | event\_platform | ComputerName | AgentVersion | FirstSeen |
    | :--- | :--- | :--- | :--- | :--- |
    | 1234abcd | Windows | ec2-web-01 | 6.45.15679 | 2025-05-21T11:00:00Z |
    | 5678efgh | Linux | ec2-app-02 | 6.45.15679 | 2025-05-23T13:00:00Z |
    | 90123ijk | Windows | ec2-db-03 | 6.45.15679 | 2025-05-25T15:00:00Z |

# selfJoin()

This function is used to collate data from events that share a key. Often the `groupBy()` function can be used for this, but if there are too many keys (defaulting to 100,000) then the result is imprecise since some random subset of keys is left out of the result once the limit is reached.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `collect` | array of strings | optional | Specifies columns to include. |
| `field` | array of strings | required | Specifies which field in the event (log line) that must match the given column value. |
| `limit` | number | optional \<br\> default: `20000` \<br\> minimum: `1` \<br\> maximum: `MAX_STATE_LIMIT` | Specifies the maximum number of rows in the subquery. |
| `postfilter` | boolean | optional \<br\> default: `false` | Re-run the AND of the WHERE clauses after collating results. If all fields needed for satisfying the WHERE clauses are provided as values for `collect`, this will eliminate false positives in the output. |
| `prefilter` | boolean | optional \<br\> default: `false` | Only pass values matching at least one of the WHERE clauses into the embedded `groupBy()`. |
| `select` | array of strings | optional | Specifies columns to include. |
| `where` | `[filter]` | required | The subquery to execute producing the values to join with. |

***

## Function Operation

The `selfJoin()` function is designed to correlate data from events that share a common key, especially in scenarios where the number of unique keys exceeds the `groupBy()` function's default limit (100,000), which can lead to imprecise results by omitting a random subset of keys.

`selfJoin()` operates by taking a `field` (the join key) and a series of tests defined in `where` clauses (as a subquery). It performs a two-phase operation to ensure that only log lines with a join key satisfying all the collective requirements of the `where` clauses are passed on for further processing. This is achieved in a probabilistic fashion using a bloom filter.

The `limit` parameter (default 20,000, max `MAX_STATE_LIMIT`) controls the maximum number of rows in the internal subquery and the number of matching join keys that satisfy the `where` clauses' requirements. The `GroupMaxLimit` dynamic configuration also affects the number of matching join keys, defaulting to 1,000,000.

An important note is that `selfJoin()` is **probabilistic**, and the result can contain false positives, particularly as the number of matches increases. For example, for 25,000 matching IDs, there might be an average of 1.1 false positives.

`selfJoin()` performs two passes over the data and, therefore, cannot run as a truly live query.

The embedded aggregation within `selfJoin()` uses either `collect()` or `selectLast()` on the specified fields, depending on the value of the `select` or `collect` parameters. The `postfilter` parameter, if true, re-runs the AND of the `where` clauses after results are collated, which can eliminate false positives if all necessary fields are included in `collect`. The `prefilter` parameter ensures only values matching at least one `where` clause are passed into the embedded `groupBy()`.

***

## Examples

### Filter and Collect Values in Same Table

This example retrieves all emails sent from one given person to another given person using the `selfJoin()` function.

  * **Query Example**

    ```
    selfJoin(email_id, where=[{from=*peter*}, {to=*anders*}], collect=[from, to])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selfJoin(email_id, where=[{from=*peter*}, {to=*anders*}], collect=[from, to])`: Finds and collects all the values in the `email_id` field that correspond to mails from "Peter" to "Anders". The `where` clause specifies that `from` field should contain "peter" and `to` field should contain "anders". The `collect` parameter ensures that `from` and `to` fields are included in the output.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find and collect all emails sent from one given person to another person. In general, the `selfJoin()` function is useful for narrowing down a set of events in a fairly efficient manner, in cases where the total set of events is too voluminous. Note that this query does two passes over the data and, therefore, cannot be used in a live query.

# selfJoinFilter()

This function is a filter query that runs in two phases:

1.  Runs a query to determine a set of IDs (specified using the `field` parameter), for which there exists an event with that field ID which satisfy all the `where` clauses. Each `where` clause can be satisfied by distinct events (but they must all have the same ID).
2.  Runs as a filter function that lets all events that have one of the determined IDs pass through. In the secondary run, the events need only match the ID, not any of the `where` clauses; unless `prefilter=true` is set.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | array of strings | required | Specifies which field in the event (log line) is the join key identifier. |
| `prefilter` | boolean | optional \<br\> default: `false` | Defines whether to include events that match the `where` clauses. \<br\>**Values:**\<br\>`false`: Passes all events with matching IDs and includes events that do not match the `where` clauses. \<br\>`true`: Only passes events that directly match `where` clauses. |
| `where` | `[filter]` | required | The subquery to execute producing the values to join with. |

***

## Function Operation

The `selfJoinFilter()` function operates as a two-phase filter query, primarily designed to efficiently narrow down a set of events that share a common key, especially when dealing with a large volume of data where other aggregate functions might hit key limits.

**Phase 1: ID Determination**
In the first phase, `selfJoinFilter()` executes a subquery defined by the `where` parameter. This subquery identifies a set of unique IDs (specified by the `field` parameter, which acts as the join key). An ID is included in this set if there exists at least one event with that ID that satisfies *all* of the conditions specified in the `where` clauses. Importantly, each `where` clause can be satisfied by distinct events, as long as those events share the same common ID. This process uses a compact and fast, but imprecise, summary of relevant keys, often employing a probabilistic bloom filter.

**Phase 2: Event Filtering**
In the second phase, `selfJoinFilter()` acts as a filter on the main event stream. It allows all events that possess one of the IDs determined in Phase 1 to pass through. By default, with `prefilter=false`, these events are passed regardless of whether they themselves directly satisfy the original `where` clauses. However, if `prefilter=true` is set, only events that *directly* match one of the `where` clauses are passed through.

**Important Considerations:**

  * **Probabilistic Nature**: `selfJoinFilter()` is probabilistic, meaning the result can contain false positives. The documentation provides a table illustrating the false positive rate based on the number of matches. For example, with 25,000 matching IDs, there could be an average of 1.1 false positives.
  * **Two-Pass Operation**: This function performs two passes over the data, which means it cannot be used in a truly live query unless combined with `beta:repeating()`.
  * **Multiple Fields in `field`**: If multiple fields are specified in the `field` parameter, all of them must exist in an event for that event to be considered valid for `selfJoinFilter()`.

A typical workflow when using this function involves:

1.  Filtering the initial event set to define a base set of events.
2.  Using `selfJoinFilter()` to identify events sharing common keys based on specified conditions.
3.  Correlating the content of the filtered events, often by using `groupBy()` for aggregation.
4.  (Optionally) Filtering the results further to exclude any correlated data not required in the final output.

***

## Examples

### Compare and Filter Values in Same Table

This example retrieves all emails with attachments sent from one given person to another given person using the `selfJoinFilter()` function, matching only by the email ID.

  * **Query Example**

    ```
    selfJoinFilter(field=email_id, where=[{from=peter}, {to=paul}])
    attachment=*
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selfJoinFilter(field=email_id, where=[{from=peter}, {to=paul}])`: This is the first phase. It finds all `email_id` values for which there exists an event where the `from` field is "peter" AND an event where the `to` field is "paul" (both sharing the same `email_id`). The subsequent events will then be filtered based on these `email_id`s (Phase 2).
    3.  `| attachment=*`: This is applied to the events passed from `selfJoinFilter()`. It further filters these events, retaining only those that also have an `attachment` field present (i.e., `attachment` exists and has any value).
    4.  Event Result set.

  * **Summary and Results**
    The query is used to find all emails with attachments sent from one given person to another person. In general, the `selfJoinFilter()` function is useful for narrowing down a set of events in a fairly efficient manner, especially in cases where the total set of events is too voluminous. Note that this query performs two passes over the data and, therefore, cannot be used in a live query.

# series()

The `series()` function collects a series of values for the selected fields from multiple events into one (or more) events. Combined with `groupBy()`, this can be used to gather data from transactions by some identity field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `collect` | array of strings | required | Names of the fields to keep. |
| `endmatch` | filter | optional | A filter query inside `{}` to match the end of a transaction (applied to the event as a whole), for example, `{/session end:/}`. Even with this parameter specified, "partial" sessions which do not include an end event are output, unlike what happens with the `startmatch` parameter, `endmatch` does not cause any event to be ignored. |
| `maxduration` | relative-time | optional | Maximum duration of a transaction (for example, `5min`), specified as a Relative Time Syntax. |
| `maxpause` | relative-time | optional | Maximum time between events in a transaction (for example, `10s`), specified as a Relative Time Syntax. |
| `memlimit` | string | optional \<br\> default: `1KiB` \<br\> minimum: `1` \<br\> maximum: `1024` | Limit on number of bytes of memory consumed by each series invocation (defaults to 1KiB). When used with the parameters `startmatch`, `endmatch`, `maxpause` and `maxduration` to produce multiple sub-series, this parameter controls the memory usage of the entire sequence of series, not each individual one. When `series` is used inside a `groupBy()`, this parameter only limits the memory consumption per group. So if the `groupBy()` is limited to 50,000 groups each using 1KB, the combined upper limit would be 50MB. |
| `separator` | string | optional \<br\> default: `\n` | String used to separate multiple values. |
| `startmatch` | filter | optional | A filter query inside `{}` to match the start of a transaction (applied to the event as a whole), for example, `{/session start:/}`. With this parameter specified, any event coming before the first start event, or in between an end event and the start event that follows, is not part of any session and is therefore ignored - all sessions include exactly one start event. |

***

## Function Operation

The `series()` function is used to gather values of specified fields from a sequence of events and consolidate them into one or more resultant events. This is particularly useful for reconstructing transaction flows or user sessions by collecting related data points that are spread across multiple individual log entries. It works effectively when combined with `groupBy()` to analyze data based on a common identity (e.g., `userID`, `client_ip`).

The function allows for flexible definition of a "series" or "session" using several optional parameters:

  * **`collect`**: Defines which fields' values to accumulate into the output series.
  * **`startmatch`**: A filter that specifies the event that marks the beginning of a new series. Events occurring before the first `startmatch` or between an `endmatch` and a subsequent `startmatch` are ignored. Each series will contain exactly one start event.
  * **`endmatch`**: A filter that specifies the event that marks the end of a series. Unlike `startmatch`, `endmatch` does not cause partial sessions (those without an end event) to be ignored; they are still outputted.
  * **`maxduration`**: Sets an upper time limit for a series. If a series exceeds this duration, it is terminated.
  * **`maxpause`**: Defines the maximum allowed time gap between consecutive events within a single series. If the pause between events exceeds this limit, the current series is considered ended, and a new one may begin with the next event.
  * **`separator`**: The string used to join the collected values within a field (default is a newline character).
  * **`memlimit`**: Controls the memory consumption for each series invocation, or for the entire sequence of sub-series when `startmatch`, `endmatch`, `maxpause`, and `maxduration` are used. When `series()` is nested within `groupBy()`, `memlimit` applies per group.

The output event(s) from `series()` typically include `@timestamp` (timestamp of the first event in the series), `_duration` (timespan of the series), and the collected fields as time-ordered series of values.

***

## Examples

### In an access log, collect the series of methods used for a given URL

This example shows how to collect a time-ordered series of HTTP methods for events related to a specific URL.

  * **Query Example**:

    ```
    url="/some/url"
    | series([method], separator=";")
    ```

  * **Input Event Data**:
    (Assume events related to `/some/url` with `method` fields, e.g., "GET", "POST", "GET", "DELETE")

  * **Result Event Data**:

    | Field | Example | Description |
    | :--- | :--- | :--- |
    | `@timestamp` | 145245466 | Timestamp of the first event arriving. |
    | `_duration` | 1245 | Timespan (in milliseconds) of the series of events included in this series. If the series contains just one field value, the value of `_duration` will be 0. |
    | `method` | GET;POST;GET; GET; DELETE | Time-ordered series of values for the method field. |

---

### Aggregate series of website visits, each visitor defined as non-active after 1 minute

This example uses `groupBy()` with `series()` to aggregate website visit data for each client IP, defining a new visit session if there's a pause of more than 1 minute between events.

  * **Query Example**:

    ```
    groupBy(client_ip, function=series(maxpause=1m, collect=[url], memlimit=1KB))
    ```

  * **Step-by-Step**:

    1.  `groupBy(client_ip, function=series(...))`: Groups events by `client_ip`.
    2.  `series(maxpause=1m, collect=[url], memlimit=1KB)`: For each `client_ip` group, it collects the `url` field values. A new series (visit) starts if there's a gap of more than 1 minute between events (`maxpause=1m`). The `memlimit=1KB` controls memory usage per series.

  * **Summary and Results**:
    This query effectively segments user activity into distinct "visits" based on inactivity. It can produce multiple events per `client_ip` if their activity has breaks longer than 1 minute, with each event representing a single visit series containing the URLs accessed during that visit.

---

### Aggregate series of auth logs, starting a new series for each login attempt

This example aggregates authentication logs into series, where each series begins with a "Login attempt" message.

  * **Query Example**:

    ```
    groupBy(userID, function=series(collect=[@rawstring], startmatch={ /Login attempt:/ }))
    ```

  * **Step-by-Step**:

    1.  `groupBy(userID, function=series(...))`: Groups events by `userID`.
    2.  `series(collect=[@rawstring], startmatch={ /Login attempt:/ })`: For each `userID`, it collects the `@rawstring` content into a series. A new series is initiated every time an event matching the filter `/Login attempt:/` is encountered. Events before the first `startmatch` or between an `endmatch` (if used) and a subsequent `startmatch` are ignored for session purposes.

  * **Summary and Results**:
    This query allows for tracking user activity starting precisely from a login attempt, providing a view of all subsequent raw log messages within that "session" until implicitly ended (or by `endmatch`, `maxpause`, etc.).

---

### Aggregate series of auth logs, ending each series with a failed login attempt

This example aggregates authentication logs, with each series terminating upon a "Failed Login" message.

  * **Query Example**:

    ```
    groupBy(userID, function=series(collect=[@rawstring], endmatch={ /Failed Login/ }))
    ```

  * **Step-by-Step**:

    1.  `groupBy(userID, function=series(...))`: Groups events by `userID`.
    2.  `series(collect=[@rawstring], endmatch={ /Failed Login/ })`: For each `userID`, it collects the `@rawstring` content into a series. A series is marked as ended upon encountering an event matching the filter `/Failed Login/`. Note that unlike `startmatch`, the `endmatch` parameter still outputs "partial" sessions that do not explicitly end with a matched event.

  * **Summary and Results**:
    This query is useful for analyzing sequences of events leading up to a failed login, providing context for troubleshooting or security investigations.

# session()

The `session()` function collects events into sessions, which are series of events that are no further than `maxpause` apart (defaults to 15m), and then performs an aggregate operation across the events that make up the session.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `function` | array of aggregate functions | optional \<br\> default: `count(as=_count)` | Specifies which aggregate functions to perform on each session. If several aggregators are listed for the `function` parameter, then their outputs are combined using the rules described for `stats()`. |
| `maxpause` | string | optional \<br\> default: `15m` | Defines the maximum pause between sessions; for example, events more than this far apart will become separate sessions. |

***

## Function Operation

The `session()` function groups related events into "sessions" based on a maximum inactivity period defined by the `maxpause` parameter. By default, events are grouped into the same session if they occur no more than 15 minutes apart. If the time between two consecutive events exceeds this `maxpause`, a new session is started for the subsequent event.

Once events are grouped into sessions, `session()` applies one or more aggregate functions (specified by the `function` parameter) across all events within each session. If multiple aggregate functions are provided, their outputs are combined following the rules similar to `stats()`. By default, `session()` performs a `count(as=_count)` on each session.

This function is typically used in conjunction with `groupBy()` to analyze sessions for unique identifiers (e.g., `cookie_id`, `client_ip`). It is valuable for understanding user engagement, identifying activity patterns, and detecting anomalies by providing a consolidated view of actions within a defined period of continuous activity.

***

## Examples

### Analyze User Sessions Based on Click Activity

This example uses the `session()` function to analyze user sessions based on their click activity, grouping events by `cookie_id` and counting clicks within sessions.

  * **Query Example**

    ```
    groupBy(cookie_id, function=session(maxpause=15m, count(as=clicks)))
    sort(clicks)
    ```

  * **Input Event Data**

    | timestamp | cookie\_id | action\_type | page\_url | user\_agent |
    | :--- | :--- | :--- | :--- | :--- |
    | 2025-05-15 05:30:00 | user123 | pageview | /home | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:30:15 | user123 | click | /products | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:30:30 | user123 | click | /product/item1 | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:31:00 | user123 | click | /cart | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:31:30 | user123 | click | /checkout | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:35:00 | user456 | pageview | /home | Mozilla/5.0 (Macintosh; Mac OS X) |
    | 2025-05-15 05:35:30 | user456 | click | /about | Mozilla/5.0 (Macintosh; Mac OS X) |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy(cookie_id, function=session(maxpause=15m, count(as=clicks)))`: Groups events by the `cookie_id` field (unique user identifier). It then creates sessions with a 15-minute inactivity timeout (the default value of the `maxpause` parameter). For each session, it counts the number of events (clicks) and returns the result in a new field named `clicks`.
    3.  `sort(clicks)`: Sorts the results by the number of clicks (by default, in descending order).
    4.  Event Result set.

  * **Summary and Results**
    The query is used to analyze user sessions based on their click activity. This is useful, for example, to identify the most/least active user sessions, detect potential automated behavior, or simply to understand user engagement levels.

  * **Result Event Data**

    | cookie\_id | clicks |
    | :--- | :--- |
    | user123 | 5 |
    | user456 | 3 |
    | user789 | 2 |

---

### Count Unique Visitors Based on Client IP Addresses

This example uses the `session()` function to count unique visitors (each visitor defined as non-active for 15 minutes) of a site based on client IP addresses.

  * **Query Example**

    ```
    groupBy(client_ip, function=session(maxpause=15m))
    | count()
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy(client_ip, function=session(maxpause=15m))`: Groups events by the `client_ip` field into sessions, with events more than 15 minutes apart becoming separate sessions. For example, if the same user returns to a site within 15 minutes, it will be considered the same session. By default, `session()` performs `count(as=_count)` on each session, so this step implicitly counts sessions per IP.
    3.  `| count()`: Makes a total count of all the unique sessions generated across all IP addresses.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to group events by client IP addresses into 15-minute sessions, and then makes a count of the total number of unique sessions (returns the total count of sessions across all IP addresses). This query is useful, for example, for measuring unique website/application visitors and understanding real user engagement patterns. It is also useful for security monitoring and detection of unusual spikes in unique visitors.
    If you instead use `count(client_ip)` after the `groupBy` for `client_ip` and `session`, the query will return a more detailed result showing the session count per IP address. For example, if an IP address has activity that spans beyond the 15-minute session timeout, it would create two distinct sessions.

  * **Result Event Data (Total count)**
    `_count`
    `4`

  * **Result Event Data (Count per IP if `count(client_ip)` used)**

    | client\_ip | \_count |
    | :--- | :--- |
    | 192.168.1.100 | 2 |
    | 192.168.1.101 | 1 |
    | 192.168.1.102 | 1 |

---

### Find Minimum And Maximum Values of any Numerical Field in Session

This example uses the `session()` function to find the minimum and maximum values of the `bet` field within each session.

  * **Query Example**

    ```
    groupBy(cookie_id, function=session([max(bet), min(bet)]))
    ```

  * **Input Event Data**

    | timestamp | cookie\_id | bet | action\_type | category |
    | :--- | :--- | :--- | :--- | :--- |
    | 2025-05-15 05:30:00 | user123 | 25.99 | purchase | electronics |
    | 2025-05-15 05:32:00 | user123 | 49.99 | purchase | electronics |
    | 2025-05-15 05:34:00 | user123 | 15.99 | purchase | accessories |
    | 2025-05-15 05:48:00 | user123 | 99.99 | purchase | appliances |
    | 2025-05-15 05:49:00 | user123 | 150.00 | purchase | furniture |
    | 2025-05-15 05:35:00 | user456 | 75.50 | purchase | clothing |
    | 2025-05-15 05:37:00 | user456 | 199.99 | purchase | appliances |
    | 2025-05-15 05:40:00 | user456 | 89.99 | purchase | electronics |
    | 2025-05-15 05:30:00 | user789 | 10.99 | purchase | books |
    | 2025-05-15 05:55:00 | user789 | 20.99 | purchase | books |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy(cookie_id, function=session([max(bet), min(bet)]))`: Groups events by the `cookie_id` field (unique user identifier) and creates sessions with a 15-minute timeout (the default value of the `maxpause` parameter). Then, for each session, it calculates the maximum and minimum values of the `bet` field, returning the results in new fields named `_max` and `_min`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to analyze the likelihood (the "bet") of behavior within user sessions. This query is useful, for example, for identifying if an event was an attempt to hack the system. Each session shows its own min/max values, highlighting that `user123` and `user789` had two sessions each due to a pause exceeding 15 minutes.

  * **Result Event Data**

    | cookie\_id | \_max | \_min |
    | :--- | :--- | :--- |
    | user123 | 49.99 | 15.99 |
    | user123 | 150.00 | 99.99 |
    | user456 | 199.99 | 75.50 |
    | user789 | 10.99 | 10.99 |
    | user789 | 20.99 | 20.99 |

# setField()

The `setField()` function takes two expressions, `target` and `value`, and sets the field named by the result of the `target` expression to the result of the `value` expression. It can be used to manipulate fields whose names are not statically known, but computed at runtime. This function is available from v1.127.0.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `target` | expression | required \<br\> default: `setField` | Evaluates the name of the field to set. |
| `value` | expression | required | An expression whose result becomes the value for the field. |

***

## Function Operation

The `setField()` function is used to dynamically create or modify fields within an event, where the name of the target field can be determined at runtime based on an expression. This distinguishes it from direct field assignments (`fieldName := value`), which require the field name to be static.

The function takes two primary arguments:

  * `target`: An expression that, when evaluated, produces the name of the field to be set. This allows for flexible field naming.
  * `value`: An expression whose evaluation result becomes the content of the target field. This expression can be a simple literal, another field's value, or the result of a more complex calculation, including the use of other functions like `if()`.

`setField()` enables advanced field manipulation scenarios where field names are not known beforehand or need to be constructed based on event data.

***

## Examples

### Set Values for Multiple Fields

This example demonstrates how to set values for multiple fields, including one where the target field name is derived from another field's content, and another where the value is conditionally determined by an `if()` function.

  * **Query Example**

    ```
    item := 4
    | bar := "baz"
    | setField(target=bar, value=item + 10)
    | setField(target="foo", value=item + 20)
    | setField(target="baaz", value=if(item == 4, then="OK", else="not OK"))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `item := 4`: In a test event, the field `item` is initialized with the value `4`.
    3.  `| bar := "baz"`: A field `bar` is created and assigned the string value `"baz"`.
    4.  `| setField(target=bar, value=item + 10)`: This `setField()` call takes the *value* of `bar` (which is "baz") as its `target` field name. It then sets the field named `baz` to the result of `item + 10` (which is `4 + 10 = 14`).
    5.  `| setField(target="foo", value=item + 20)`: This call sets a new field explicitly named `"foo"` to the result of `item + 20` (which is `4 + 20 = 24`).
    6.  `| setField(target="baaz", value=if(item == 4, then="OK", else="not OK"))`: This adds an `if()` function to determine the value of the new target field `baaz`. If `item` is equal to `4` (which it is), then `baaz`'s value is "OK"; otherwise, it would be "not OK".
    7.  Event Result set.

  * **Summary and Results**
    This query demonstrates setting values for different target fields as the result of given expressions. It highlights how functions can be embedded within the `value` parameter to determine the content of dynamically named fields.

  * **Result Event Data**

| baaz | bar | baz | foo | item |
| :--- | :--- | :--- | :--- | :--- |
| OK | baz | 14 | 24 | 4 |

---

### Set the Value of a Field

This example sets the value of a target field based on the result of an expression. This is equivalent to direct field assignment.

  * **Query Example**

    ```
    item := 4
    setField(target="foo", value=item + 10)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `item := 4`: In a test event, the field `item` is set to `4`.
    3.  `setField(target="foo", value=item + 10)`: Sets the value of the target field `foo` as the result of the expression `item + 10` (which is `4 + 10 = 14`). This is equivalent to `foo := item + 10`.
    4.  Event Result set.

  * **Summary and Results**
    This query uses `setField()` to assign a calculated value to a specific field.

  * **Result Event Data**

| foo | item |
| :--- | :--- |
| 14 | 4 |

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

# sort()

The `sort()` query function is used to sort events based on a given field or fields. Events can be sorted by multiple fields by setting the `field` parameter to an array of field names (for example, between square-brackets in a comma-separated list). Likewise, the `order` and `type` of each field can be specified by setting the `order` and `type` parameters to arrays.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | array of strings | optional | The fields by which to sort events. |
| `limit` | integer | optional \<br\> default: `200` | The argument given to this parameter determines the limit on the number of events included in the result of the function. The default argument is `default`. The maximum is controlled by the `StateRowLimit` dynamic configuration, which is `StateRowLimit` by default. If the argument is `max` (`limit=max`), then the value of `StateRowLimit` is used. |
| `order` | array of strings | optional \<br\> default: `desc` | The order in which to sort. \<br\>**Values:**\<br\>`asc`: Sort ascending. \<br\>`desc`: Sort descending. |
| `reverse` | boolean | optional \<br\> default: `false` | Whether to sort in descending order. This parameter is deprecated: use `order` instead. \<br\>**Values:**\<br\>`false`: Sort ascending. \<br\>`true`: Sort descending. |
| `type` | array of strings | optional \<br\> default: `number` | Set the data type of the fields to influence the sorting order. \<br\>**Values:**\<br\>`any`: Any fields. \<br\>`hex`: Treat the field as a hexadecimal value and sort in numerical order. \<br\>`number`: Treat the field as numerical and sort in numerical order. \<br\>`string`: Treat the field as string values and sort alphabetical. |

***

## Function Operation

The `sort()` function is used to arrange events in a specified order based on the values of one or more fields.

Key aspects of its operation:

  * **Field Specification**: Events can be sorted by a single field or multiple fields by providing an array of field names to the `field` parameter.
  * **Sorting Order**: The `order` parameter dictates the sort direction, either `asc` (ascending) or `desc` (descending, which is the default). When sorting by multiple fields, you can provide a corresponding array to `order` to specify a different direction for each field. The `reverse` parameter is deprecated in favor of `order`.
  * **Data Type for Sorting**: The `type` parameter informs `sort()` how to compare values. Options include `string` (lexicographical order), `number` (numerical magnitude), and `hex` (treating as hexadecimal values for numerical sorting). If `order` or `type` is a single value, it applies to all specified fields.
  * **Limiting Results**: The `limit` parameter controls the maximum number of events returned *after* the sorting is complete. The default limit is 200, but can be set to `max` to use the `StateRowLimit` dynamic configuration value.
  * **Performance Considerations**: Sorting is an in-memory operation and can be resource-intensive for large numbers of events. For optimal performance, it is recommended to:
      * Perform filtering prior to sorting to reduce the number of events.
      * Use aggregation functions to simplify the event list before sorting.
      * Place `sort()` as the last function in a query, especially after an aggregating function, for best performance.

***

## Examples

### Get a list of referring domains and sub-domains, counted and sorted alphabetically in reverse order

This example extracts referring domains from URLs, groups them, counts them, and then sorts them alphabetically in descending order.

  * **Query Example**

    ```
    regex(regex="/.*/(?<ref_url>.+?)(/|$)", field=referrer)
    groupBy(ref_url)
    | sort(ref_url, type=string, order=desc, limit=12)
    ```

  * **Step-by-Step**

    1.  The `regex()` function extracts just the domain and sub-domain portion of the URL from the `referrer` field into a new field called `ref_url`, dropping everything after the domain and first slash.
    2.  `groupBy(ref_url)`: Groups events by each unique `ref_url` and implicitly counts their occurrences.
    3.  `| sort(ref_url, type=string, order=desc, limit=12)`: Sorts the grouped results by `ref_url` as a `string` (alphabetically) in `descending` order, and limits the output to the top 12 entries.

  * **Result Event Data**

| ref\_url | \_count |
| :--- | :--- |
| www.zabotkin.ru | 14 |
| www.tangblog.top | 1 |
| [www.skidn.com](https://www.google.com/search?q=https://www.skidn.com) | 1 |
| www.klfd.net | 1 |
| www.iopt.pro | 42 |
| [www.google.com](https://www.google.com).au | 1 |
| [www.google.com](https://www.google.com) | 11 |
| [www.bing.com](https://www.bing.com) | 3 |

---

### Sort metrics by repository name and then output the top entry

This example shows how to use `sort()` with `groupBy()` to find the single top entry after sorting.

  * **Query Example**

    ```
    name=datasource-count
    groupBy([repo])
    | sort(field=_count, type=number, order=desc, limit=1)
    ```

  * **Step-by-Step**

    1.  `name=datasource-count`: Filters events where the `name` field is `datasource-count`.
    2.  `groupBy([repo])`: Groups the events by repository (`repo`) and implicitly counts them.
    3.  `| sort(field=_count, type=number, order=desc, limit=1)`: Sorts the grouped counts (in the `_count` field) numerically in descending order, and then limits the result to only the top 1 entry.

---

### Sort events by multiple fields with different orders

This example demonstrates sorting by multiple fields where each field has a different sorting order (one descending, one ascending).

  * **Query Example**

    ```
    groupBy([ref_url, method])
    | sort([ref_url, method], type=string, order=[desc, asc], limit=12)
    ```

  * **Step-by-Step**

    1.  `groupBy([ref_url, method])`: Groups events by both `ref_url` and `method`.
    2.  `| sort([ref_url, method], type=string, order=[desc, asc], limit=12)`: Sorts the results. `ref_url` will be sorted as a string in descending order, while `method` will be sorted as a string in ascending order. The output is limited to 12 results.

---

### Analyze User Sessions Based on Click Activity

This example analyzes user sessions, groups them by cookie ID, counts clicks within each session, and then sorts the sessions by the number of clicks.

  * **Query Example**

    ```
    groupBy(cookie_id, function=session(maxpause=15m, count(as=clicks)))
    sort(clicks)
    ```

  * **Input Event Data**

    | timestamp | cookie\_id | action\_type | page\_url | user\_agent |
    | :--- | :--- | :--- | :--- | :--- |
    | 2025-05-15 05:30:00 | user123 | pageview | /home | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:30:15 | user123 | click | /products | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:30:30 | user123 | click | /product/item1 | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:31:00 | user123 | click | /cart | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:31:30 | user123 | click | /checkout | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:35:00 | user456 | pageview | /home | Mozilla/5.0 (Macintosh; Mac OS X) |
    | 2025-05-15 05:35:30 | user456 | click | /about | Mozilla/5.0 (Macintosh; Mac OS X) |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy(cookie_id, function=session(maxpause=15m, count(as=clicks)))`: Groups events by `cookie_id` and creates sessions with a 15-minute inactivity timeout. It then counts the number of events (clicks) within each session, storing the result in a field named `clicks`.
    3.  `sort(clicks)`: Sorts the results by the `clicks` field (by default, in descending order).
    4.  Event Result set.

  * **Summary and Results**
    The query analyzes user sessions based on click activity, useful for identifying most/least active user sessions, detecting potential automated behavior, or understanding user engagement levels.

  * **Result Event Data**

| cookie\_id | clicks |
| :--- | :--- |
| user123 | 5 |
| user456 | 3 |
| user789 | 2 |

---

### Calculate Query Cost for All Users by Repository

This example searches across multiple repositories to calculate query costs for all users by repository, then sorts the results by live and static costs.

  * **Query Example**

    ```
    #type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"
    groupBy(repo, initiatingUser, totalLiveCost, totalStaticCost)
    | sort([totalLiveCost, totalStaticCost])
    ```

  * **Input Event Data**

    | \#type | \#kind | class | message | timestamp |
    | :--- | :--- | :--- | :--- | :--- |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:30:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:31:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:32:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:33:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:34:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:35:00Z |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"`: Filters for internal LogScale logs containing `c.h.j.RunningQueriesLoggerJob` in the `class` field and where the `message` field is "Highest Cost query".
    3.  `groupBy(repo, initiatingUser, totalLiveCost, totalStaticCost)`: Groups the results by `repo`, `initiatingUser`, `totalLiveCost`, and `totalStaticCost`, and returns a count in a field named `_count`.
    4.  `| sort([totalLiveCost, totalStaticCost])`: Sorts the results by both `totalLiveCost` and `totalStaticCost` fields (by default, in descending order).
    5.  Event Result set.

  * **Summary and Results**
    The query searches across multiple repositories and outputs query costs for all users by repository. The query returns the count in a field named `_count`. Use this query to focus on live and static costs separately.

  * **Result Event Data**

| repo | initiatingUser | totalLiveCost | totalStaticCost | \_count |
| :--- | :--- | :--- | :--- | :--- |
| app-logs | jane.smith | 2000 | 1200 | 1 |
| security-logs | john.doe | 1500 | 800 | 1 |
| infra-logs | bob.wilson | 1000 | 500 | 1 |

---

### Calculate Total Network Bandwidth Per Host

This example calculates total inbound and outbound network traffic per host, then calculates total bandwidth consumption, and finally sorts by total traffic.

  * **Query Example**

    ```
    event_simpleName="NetworkConnectStats"
    groupBy([ComputerName], function=[
        sum(field="BytesReceived", as=InboundTraffic),
        sum(field="BytesSent", as=OutboundTraffic)
    ])
    TotalTraffic := InboundTraffic + OutboundTraffic
    sort(field="TotalTraffic", order="desc")
    ```

  * **Input Event Data**

    | @timestamp | event\_simpleName | ComputerName | BytesReceived | BytesSent |
    | :--- | :--- | :--- | :--- | :--- |
    | 1686837825000 | NetworkConnectStats | DESKTOP-A1 | 15000000 | 8000000 |
    | 1686837825000 | NetworkConnectStats | DESKTOP-A1 | 8900000 | 4600000 |
    | 1686837825000 | NetworkConnectStats | LAPTOP-B2 | 25000000 | 12500000 |
    | 1686837826000 | NetworkConnectStats | SERVER-C3 | 95000000 | 47000000 |
    | 1686837826000 | NetworkConnectStats | DESKTOP-A1 | 12000000 | 6000000 |
    | 1686837826000 | NetworkConnectStats | LAPTOP-B2 | 18000000 | 9000000 |
    | 1686837827000 | NetworkConnectStats | SERVER-C3 | 85000000 | 42000000 |
    | 1686837827000 | NetworkConnectStats | DESKTOP-D4 | 5000000 | 2500000 |
    | 1686837827000 | NetworkConnectStats | LAPTOP-B2 | 22000000 | 11000000 |
    | 1686837828000 | NetworkConnectStats | SERVER-C3 | 105000000 | 50000000 |
    | 1686837828000 | NetworkConnectStats | DESKTOP-D4 | 6500000 | 3500000 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `event_simpleName="NetworkConnectStats"`: Filters events to include only those where `event_simpleName` equals "NetworkConnectStats".
    3.  `groupBy([ComputerName], function=[sum(field="BytesReceived", as=InboundTraffic), sum(field="BytesSent", as=OutboundTraffic)])`: Groups the data by `ComputerName` and calculates two aggregate values: the sum of `BytesReceived` (stored as `InboundTraffic`) and the sum of `BytesSent` (stored as `OutboundTraffic`).
    4.  `TotalTraffic := InboundTraffic + OutboundTraffic`: Creates a new field named `TotalTraffic` by summing the `InboundTraffic` and `OutboundTraffic` values.
    5.  `sort(field="TotalTraffic", order="desc")`: Sorts the results based on the `TotalTraffic` field in descending order, showing hosts with the highest bandwidth consumption first.
    6.  Event Result set.

  * **Summary and Results**
    The query analyzes network bandwidth consumption patterns across different hosts. This is useful for identifying hosts consuming excessive bandwidth, monitoring network usage, or detecting network-intensive applications or anomalies. The traffic values are in bytes, and each row represents aggregated traffic for a unique host.

  * **Result Event Data**

| ComputerName | InboundTraffic | OutboundTraffic | TotalTraffic |
| :--- | :--- | :--- | :--- |
| SERVER-C3 | 285000000 | 139000000 | 424000000 |
| LAPTOP-B2 | 65000000 | 32500000 | 97500000 |
| DESKTOP-A1 | 45700000 | 24600000 | 70300000 |
| DESKTOP-D4 | 11500000 | 6000000 | 17500000 |

---

### Filter Out Fields With No Value

This example demonstrates how to filter out fields with no values from search results.

  * **Query Example**

    ```
    method=GET
    groupBy(field=[method, statuscode], function=count(as=method_total))
    sort([method, statuscode], order=asc)
    FieldName!=""
    ```

  * **Input Event Data**
    (Example output after `groupBy` and `sort` before final filter)

    | method | statuscode | method\_total |
    | :--- | :--- | :--- |
    | GET | \<no value\> | 10 |
    | GET | 200 | 32492 |
    | GET | 301 | 1 |
    | GET | 304 | 113 |
    | GET | 403 | 9 |
    | GET | 404 | 132 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `method=GET`: Filters for all events with methods of the type `GET`.
    3.  `groupBy(field=[method, statuscode], function=count(as=method_total))`: Groups the returned results into `method` and `statuscode` fields and counts the events into a new field named `method_total`.
    4.  `sort([method, statuscode], order=asc)`: Sorts the returned results in ascending order.
    5.  `FieldName!=""`: Excludes all events where one of the fields in the result set (`method` or `statuscode`) does not contain a value. (Note: The `FieldName!=""` syntax typically checks if a *specific* field is not empty. If the intent is to check all output fields, a different approach might be needed or `FieldName` should represent the actual field to check, e.g. `statuscode!=""`).
    6.  Event Result set.

  * **Summary and Results**
    The query filters out fields not containing any values from the returned search result.

  * **Result Event Data**

| method | statuscode | method\_total |
| :--- | :--- | :--- |
| GET | 200 | 32492 |
| GET | 301 | 1 |
| GET | 304 | 113 |
| GET | 403 | 9 |
| GET | 404 | 132 |

---

### Find Least Common Values of a Field

This example finds the least common values of a field by sorting aggregated counts in ascending order. This is the opposite of the `top()` function.

  * **Query Example**

    ```
    groupby([type, kind], limit=max, function=count())
    sort(_count, order=asc)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupby([type, kind], limit=max, function=count())`: Groups the event set by the `type` and `kind` fields. For each unique combination, it counts the number of events and returns the results in a field named `_count`. The `limit=max` parameter is set to return all counts, as the purpose of this example is to find the least common values of a field.
    3.  `sort(_count, order=asc)`: Sorts the returned results in ascending order from the least common combinations to the most common combinations of `type` and `kind`.
    4.  Event Result set.

  * **Summary and Results**
    The query finds the least common values of a field in a set of events, in this example, identifying the least and most common type/kind combinations in an event set. Identifying rare combinations of type and kind could indicate unusual events or potential security issues. This type of query is useful in various scenarios, particularly for data analysis and understanding patterns in event sets.

---

### Sort Timestamps With `groupBy()`

This example demonstrates how to sort fields based on aggregated field values, specifically sorting timestamps within a `groupBy()` aggregation.

  * **Query Example**

    ```
    timestamp: formatTime(format="%H:%M")
    groupBy([thread], function=[{sort("timestamp") | collect("timestamp")}])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `timestamp: formatTime(format="%H:%M")`: Creates a new field, `timestamp`, formatted as `HH:MM`.
    3.  `groupBy([thread], function=[{sort("timestamp") | collect("timestamp")}])`: Groups the events by the name of the `thread`. As an embedded expression for the function, it first sorts the events on the `timestamp` field (the newly formatted `HH:MM` timestamp) and then uses `collect("timestamp")` to retrieve and include that sorted `timestamp` field's value in the aggregated event set, as it would normally be removed during aggregation.
    4.  Event Result set.

  * **Summary and Results**
    The result set will contain a list of the aggregated thread names, with their associated timestamps sorted.

  * **Result Event Data (Example)**

    ```
    10:09
    LogCollector ManifestUpdate
    TransientChatter event loop
    10:10
    ```

# split()

The `split()` function splits an event structure that includes an array into multiple distinct events with each array element.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | optional \<br\> default: `events` | Field to split by. |
| `strip` | boolean | optional \<br\> default: `false` | Strip the field prefix when splitting (default is `false`). |

***

## Function Operation

When LogScale ingests data into arrays, each array entry is turned into separate attributes named like `[0]`, `[1]`, etc. The `split()` function takes such an event and divides it into multiple distinct events, where each new event corresponds to an individual element of the original array. This is done based on the prefix of these `[N]` attributes, enabling aggregate functions to be applied across array values.

When `split()` is called, each generated split event is assigned a unique index ID in the `_index` field, which can be used to identify the individual event. If the original event data includes an `@id` field, then the `@id` field is also modified for each new event by appending `_` and the index number (e.g., `1_0`, `1_1`, `1_2`).

**Important Note on Efficiency and Memory Usage:**
The `split()` function is not very efficient and should only be used after aggressive filtering. When `split()` creates new events from an array, it copies *all* fields from the original event into each new split event by default, with the exception of the array that is being split. This includes large fields like `@rawstring`. Splitting an event containing a large `@rawstring` can consume a significant amount of memory (e.g., a 1MB `@rawstring` split into 100 events would consume 100MB of memory). This memory impact is particularly problematic when `split()` is used in parsers where events are stored for longer periods.

**Best practice:** Drop any unnecessary large fields, especially `@rawstring`, before using `split()`. For example:

```
drop(@rawstring)
| split(array_to_split)
```

***

## Examples

### Splitting an event containing an array

Given an input record: `@id=1, a=[1,2,3]`

  * **Query Example**

    ```
    split(a)
    ```

  * **Result Event Data**

| @id | \_index | a |
| :--- | :--- | :--- |
| 1\_0 | 0 | 1 |
| 1\_1 | 1 | 2 |
| 1\_2 | 2 | 3 |

---

### Expanding GitHub PushEvent commits

In GitHub events, a `PushEvent` contains an array of commits, and each commit gets expanded into sub-attributes of `payload.commit0`, `payload.commit1`, etc. LogScale cannot sum/count across such attributes directly. This example shows how `split()` expands each `PushEvent` into one `PushEvent` for each commit so they can be counted.

  * **Query Example**

    ```
    type=PushEvent
    split(payload.commits)
    groupBy(payload.commits.author.email)
    | sort()
    ```

  * **Step-by-Step**

    1.  `type=PushEvent`: Filters for events of type `PushEvent`.
    2.  `split(payload.commits)`: Splits each `PushEvent` event into multiple events, one for each commit within the `payload.commits` array. Each new event will contain the data for a single commit.
    3.  `groupBy(payload.commits.author.email)`: Groups the new events by the email of the commit author.
    4.  `| sort()`: Sorts the results (likely by count, as it's a grouped aggregation).

---

### Splitting events from a JSON array in `@rawstring`

If your parser receives JSON events in a JSON array, and each record in the array is an event in itself, you would like to split them out. First, you need to call `parseJson()`. When `@rawstring` contains an array, the `parseJson()` function doesn't assign names to the fields automatically; it only assigns indexes (e.g., `[0].exampleField`, `[1].exampleField`). Since `split()` needs a field name before it reads indexes, we can tell `split()` to look for the empty field name by calling `split(field="")`.

  * **Input Event Data**

    ```json
    [
      {"exampleField": "value"},
      {"exampleField": "value2"}
    ]
    ```

  * **Input Event Data (after `parseJson()`):**

| @rawstring | @timestamp | @timestamp.nanos | \[0].exampleField | \[1].exampleField |
| :--- | :--- | :--- | :--- | :--- |
| `[ {"exampleField": "value"}, {"exampleField": "value2"} ]` | 2023-10-27 10:33:42.748 | 0 | value | value2 |

  * **Query Example**

    ```
    parseJson()
    | split(field="")
    ```

  * **Result Event Data**

| .exampleField | @rawstring | @timestamp | @timestamp.nanos | \_index |
| :--- | :--- | :--- | :--- | :--- |
| value | `{"exampleField": "value"}, {"exampleField": "value2"}` | 2023-10-27 | 10:33:18.944 | 0 |
| value2 | `{"exampleField": "value"}, {"exampleField": "value2"}` | 2023-10-27 | 10:33:18.944 | 1 |

---

### Alternative for splitting events from a JSON array with `prefix`

Alternatively, you can tell `parseJson()` to add a prefix to all the fields, which can then be used as the field name to split on.

  * **Query Example**

    ```
    parseJson(prefix="example")
    | split(field="example")
    ```

  * **Summary and Results**
    This method adds the `example` prefix to all fields on the new event, so `split(field="")` might be preferred to avoid that.

---

### Deduplicate Compound Field Data With `array:union()` and `split()`

This example demonstrates deduplicating fields of information where there are multiple occurrences of a value in a single field, possibly separated by a single character. This solution uses `array:union()` and `split()` to create a unique array and then split the content out to a unique list.

  * **Query Example**

    ```
    splitString(field=userAgent, by="", as=agents)
    array:filter(array="agents[]", function={bname=/\//}, var="bname")
    array:union(array=agents, as=browsers)
    split(browsers)
    ```

  * **Input Event Data**
    (Example `userAgent` raw event data):
    `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome`
    (The actual names are the Name/Version pairs showing compatibility with different browser standards. Resolving this into a simplified list requires splitting up the list, simplifying (to remove duplicates), filtering, and then summarizing the final list.)

  * **Step-by-Step**

    1.  Starting with the source repository events.

    2.  `splitString(field=userAgent, by="", as=agents)`: Splits the `userAgent` field into individual array entries within the `agents` array for each event.

          * **Result Event Data (after splitString)**:

        | agents\[0] | agents\[1] | agents\[2] | agents\[3] | agents\[4] |
        | :--- | :--- | :--- | :--- | :--- |
        | Mozilla/5.0 | (Macintosh; | Intel | Mac | OS |

    3.  `array:filter(array="agents[]", function={bname=/\//}, var="bname")`: Filters the `agents` array. (The provided example for this step is incomplete in the source).

    4.  `array:union(array=agents, as=browsers)`: Aggregates the list of user agents across all events to create a list of unique entries, eliminating duplicates where the value of the user agent is the same.

          * **Result Event Data (after array:union)**:

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

    5.  `split(browsers)`: Splits the `browsers` array into individual events, turning the single event with an array into multiple events, each representing a browser.

          * **Input Event Data (for split)**: (Implicitly, the event after `array:union` as shown above)

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

          * **Result Event Data (after split)**:

        | \_index | browsers |
        | :--- | :--- |
        | 0 | Gecko/20100101 |
        | 1 | Safari/537.36 |
        | 2 | AppleWebKit/605.1.15 |

    6.  Event Result set.

  * **Summary and Results**
    The resulting output from the query is a list of events with each event containing a matching index and browser. This can be useful if you want to perform further processing on a list of events rather than an array of values.

# splitString()

The `splitString()` function splits a string using a regular expression into an array of values.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_splitstring` | Emit selected attribute using this name. |
| `by` | string | required | String or regular expression to split by. |
| `field` | string | optional \<br\> default: `@rawstring` | Field that needs splitting. |
| `index` | number | optional | Emit only this index after splitting. Can be negative; -1 designates the last element. |

***

## Function Operation

The `splitString()` function takes a string from a specified field and divides it into an array of substrings based on a provided regular expression or string delimiter.

Key aspects of its operation include:

  * The `by` parameter defines the delimiter, which can be a literal string or a regular expression.
  * The `field` parameter specifies the input string to be split; if omitted, `@rawstring` is used by default.
  * The `index` parameter allows you to extract only a specific element from the resulting array by its numerical position (0-based index). A negative index, like `-1`, refers to elements from the end of the array (e.g., `-1` is the last element).
  * The `as` parameter names the output field that will contain the resulting array or the single extracted element if `index` is specified.
  * When splitting by a multi-character separator using a regular expression, ensure the regular expression group is used as the value for `by`. Special characters (including asterisk `*` and pipe `|`) also need to be escaped.

***

## Examples

### Splitting `2007-01-01 test bar` into `part[0]`, `part[1]`, `part[2]`

This example splits a string by whitespace into multiple indexed fields.

  * **Input Event Data**: `@rawstring="2007-01-01 test bar"`

  * **Query Example**:

    ```
    | part := splitString(field=@rawstring, by=" ")
    ```

  * **Result Event Data**: (Implicitly creates fields `part[0]`, `part[1]`, `part[2]`)

---

### Picking out the date part using `splitString()`

This example extracts a specific indexed element (the date) after splitting a string.

  * **Input Event Data**: `@rawstring: 2007-01-01 test bar`

  * **Query Example**:

    ```
    | date: splitString(field=@rawstring, by=" ", index=0)
    ```

  * **Result Event Data**: (Implicitly creates `date` field with value `2007-01-01`)

---

### Splitting a string using `<` `>` or `;` as delimiters

This example splits a string using a regular expression that defines multiple possible delimiters.

  * **Input Event Data**: `@rawstring: <2007-01-01>test; bar`

  * **Query Example**:

    ```
    | part := splitString(field=@rawstring, by="[<>;]")
    ```

  * **Result Event Data**: (Implicitly creates fields `part[0]`, `part[1]`, `part[2]`)

---

### Splitting an event into multiple events by newlines

This example combines `splitString()` with `split()` to transform a multi-line string within an event into separate events, each representing a line.

  * **Query Example**:

    ```
    | splitString(by="\n", as=@rawstring)
    | split(@rawstring)
    ```

  * **Step-by-Step**:

    1.  `splitString(by="\n", as=@rawstring)`: Splits the original `@rawstring` by newline characters (`\n`) and stores the resulting array back into `@rawstring`.
    2.  `| split(@rawstring)`: Takes the array now in `@rawstring` and creates a new event for each element in that array.

---

### Split the value of a string field into individual characters

This example uses a regex to split a string into an array of its individual characters.

  * **Query Example**:

    ```
    characters := splitString(my_field, by="(?<!^)(?=.)")
    ```

  * **Result Event Data**: (Implicitly creates `characters` array with each element being a single character from `my_field`)

---

### Split the value of a string using case-insensitive regex

This example demonstrates splitting a string using a regular expression that performs case-insensitive matching.

  * **Query Example**:
    ```
    characters := splitString(my_field, by="(?i)(e encoded|enc)")
    ```

---

### Split the string using a multi-character separator

This example shows how to split incoming data by a multi-character separator `*|*` and correctly extract the parts.

  * **Input Event Data**: `image.png*|*PNG*|*0755*|*john`

  * **Query Example**:

    ```
    splitString(by="(\\*\\|\\*)")
    ```

  * **Result Event Data**:

| Field | Value |
| :--- | :--- |
| \_splitstring\[0] | image.png |
| \_splitstring\[1] | PNG |
| \_splitstring\[2] | 0755 |
| \_splitstring\[3] | john |

---

### Deduplicate Compound Field Data With `array:union()`, and `split()`

This example combines `splitString()`, `array:filter()`, `array:union()`, and `split()` to deduplicate and normalize user agent data.

  * **Query Example**:

    ```
    splitString(field=userAgent, by="", as=agents)
    array:filter(array="agents[]", function={bname=/\//}, var="bname")
    array:union(array=agents, as=browsers)
    split(browsers)
    ```

  * **Input Event Data**: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome`

  * **Step-by-Step**:

    1.  Starting with the source repository events.

    2.  `splitString(field=userAgent, by="", as=agents)`: Splits the `userAgent` field into individual array entries within the `agents` array for each event.

          * **Result Event Data (after splitString)**:

        | agents\[0] | agents\[1] | agents\[2] | agents\[3] | agents\[4] |
        | :--- | :--- | :--- | :--- | :--- |
        | Mozilla/5.0 | (Macintosh; | Intel | Mac | OS |

    3.  `array:filter(array="agents[]", function={bname=/\//}, var="bname")`: Filters the `agents` array based on a function checking for `/`.

    4.  `array:union(array=agents, as=browsers)`: Aggregates the list of user agents across all events to create a list of unique entries, eliminating duplicates where the value of the user agent is the same.

          * **Result Event Data (after array:union)**:

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

    5.  `split(browsers)`: Splits the `browsers` array into individual events, turning the single event with an array into multiple events, each representing a browser.

          * **Result Event Data (for split)**: (Implicitly, the event after `array:union` as shown above)

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

          * **Result Event Data (after split)**:

        | \_index | browsers |
        | :--- | :--- |
        | 0 | Gecko/20100101 |
        | 1 | Safari/537.36 |
        | 2 | AppleWebKit/605.1.15 |

    6.  Event Result set.

  * **Summary and Results**: The resulting output from the query is a list of events with each event containing a matching index and browser. This can be useful if you want to perform further processing on a list of events rather than an array of values.

# start()

The `start()` function assigns a timestamp to an output field specified by the `as` parameter. This timestamp represents the beginning of the search time interval in milliseconds since January 1, 1970 (UTC). For live queries (where the search time interval is forever moving), `start()` equals the current time minus the search interval. For subqueries in `defineTable()` or joins, `start()` equals the start time of the subquery's search interval.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `start` | Name of output field. |

***

## Function Operation

The `start()` function retrieves and assigns the beginning timestamp of the current search interval to a specified output field. This timestamp is represented in milliseconds since January 1, 1970 (UTC).

The value returned by `start()` varies depending on the query context:

  * **Historical Queries**: It represents the exact start time of the time range chosen for the query.
  * **Live Queries**: Since the search interval is continuously moving, `start()` returns a dynamically updated timestamp that equals the current time minus the defined live search interval.
  * **Subqueries (e.g., within `defineTable()` or joins)**: It reflects the specific start time of that subquery's search interval, which might differ from the main query's interval.

It's important to note that the `start()` function is **not compatible with parser operations** because parsers do not operate with search intervals. The `as` parameter can be used to customize the name of the output field (defaulting to `start`).

***

## Examples

### Assign the value of `start()` to a field `s`

This is a basic example of assigning the search interval's start time to a new field.

  * **Query Example**
    ```
    s := start()
    ```

### Use `start()` in an assignment for a comparison

This example uses `start()` to calculate if an event's timestamp is "old" relative to the beginning of the search interval.

  * **Query Example**

    ```
    isold := (@timestamp - start()) < 1000
    ```

  * **Step-by-Step**

    1.  `(@timestamp - start())`: Calculates the difference between the event's `@timestamp` and the start time of the query interval.
    2.  `< 1000`: Checks if this difference is less than 1000 milliseconds (1 second).
    3.  `isold := ...`: Assigns a boolean result (`true` or `false`) to the field `isold`.

### Search Relative Time to Query Execution

This example demonstrates writing a query that filters events based on a time range relative to when the query is executed using the `start()` function.

  * **Query Example**

    ```
    test(@timestamp < (start() + (30 * 24 * 60 * 60 * 1000)))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `test(@timestamp < (start() + (30 * 24 * 60 * 60 * 1000)))`: Tests whether the `@timestamp` for an event is less than (earlier than) the query's start time plus 30 days.
          * `start()`: Returns the start time of the query.
          * `(30 * 24 * 60 * 60 * 1000)`: This calculation determines the number of milliseconds in 30 days:
              * 30 (days) \* 24 (hours) \* 60 (minutes) \* 60 (seconds) \* 1000 (milliseconds) = 2,592,000,000 milliseconds.
          * The condition checks if `@timestamp` is earlier than the query's start time plus 30 days.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to filter events that occurred within the first 30 days after the query's start time. This is a practical way of querying with a relative time from the query execution. The 30 days (and calculation) used in the example could be updated with any time calculation to achieve the required result.

# stats()

The `stats()` function is used to compute multiple aggregate functions over the input.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `function` | array of aggregate functions | optional \<br\> default: `count(as=_count)` | Specifies which aggregate functions to perform on each group. |

***

## Function Operation

The `stats()` function is a versatile tool for applying multiple aggregate functions simultaneously over a set of input events. It is implicitly used in various contexts where a list of sub-aggregators is provided (e.g., `groupBy(x, function=[min(y), max(y)])` is equivalent to `groupBy(x, function=stats([min(y), max(y)]))`). This mechanism enables the combination of results from different aggregators.

The output behavior of `stats()` depends on the sub-aggregators:

  * **Single-Row Output**: If all sub-aggregators produce at most one row of output (common for most numerical aggregators like `sum()`, `avg()`, `min()`, `max()`), the result will be a single combined row containing all the aggregated values.
  * **Multi-Row Output (Cartesian Product)**: If one or more sub-aggregators emit more than one result row (e.g., `table()`), the total output will be the Cartesian product of all the sub-aggregators' outputs. This means every row from one sub-aggregator is combined with every row from another. If any sub-aggregator outputs zero rows, it is treated as outputting a single empty row for the purpose of the Cartesian product.
  * **Field Name Collisions**: The output combination checks for field name collisions. It is considered an error if a field is present in multiple outputs with conflicting values.

`stats()` is also available as a shorthand syntax by simply listing aggregators within square brackets (e.g., `| [min(), max()]` is equivalent to `stats(function=[min(), max()])`). A simple `stats(function=count())` is equivalent to just `count()`.

***

## Examples

### Annotate Events With Aggregation - Example 1

This example demonstrates using the `stats()` function with aggregation on field `x` where one sub-aggregator outputs multiple rows.

  * **Query Example**

    ```
    kvParse()
    | stats([
        avg(x),
        table([x])
    ])
    ```

  * **Input Event Data**
    (Events containing `x` field values):
    `x=1`
    `x=2`
    `x=9`
    `x=10`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs (e.g., extracts `x=1` as a field `x` with value `1`).
    3.  `| stats([avg(x), table([x])])`: Computes two aggregate functions over the field `x`:
          * `avg(x)`: Calculates the average of `x` values, returning a single row with a field named `_avg`.
          * `table([x])`: Returns a row for each `x` value, producing multiple rows.
            Since `table([x])` outputs multiple rows, the final output is the Cartesian product of the `_avg` result and the `x` values from `table([x])`.
    4.  Event Result set.

  * **Summary and Results**
    The query computes multiple aggregate functions over an input. The result shows the average `_avg` repeated for each `x` value due to the Cartesian product.

  * **Result Event Data**

| \_avg | x |
| :--- | :--- |
| 5.5 | 1 |
| 5.5 | 2 |
| 5.5 | 9 |
| 5.5 | 10 |

---

### Annotate Events With Aggregation - Example 2

This example illustrates using the `stats()` function with aggregation where one of the sub-aggregators (`avg(y)`) outputs zero rows.

  * **Query Example**

    ```
    kvParse()
    | stats([
        sum(x,as=sumX),
        avg(y, as=avgY),
        table([x,y])
    ])
    ```

  * **Input Event Data**
    `"x=1 y=N/A"`
    `"x=2 y=N/A"`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs.
    3.  `| stats([sum(x, as=sumX), avg(y, as=avgY), table([x, y])])`: Computes aggregate functions over fields `x` and `y`:
          * `sum(x, as=sumX)`: Sums `x` values.
          * `avg(y, as=avgY)`: Attempts to average `y` values. Since `y=N/A` (which is likely not a number), this aggregator will produce zero rows.
          * `table([x, y])`: Returns rows for each combination of `x` and `y`.
            Since `avg(y)` outputs zero rows, the `stats()` function treats it as outputting a single empty row for the Cartesian product.
    4.  Event Result set.

  * **Summary and Results**
    The query computes multiple aggregate functions over an input. Despite `avg(y)` not producing a meaningful numerical result, its presence in `stats()` influences the Cartesian product.

  * **Result Event Data (example shown in document)**
    (Header): `sumX`, `x`, `y`
    (Row 1): `3`, `1`, `N/A`
    (Row 2): `3`, `2`, `N/A`

---

### Annotate Events With Aggregation - Example 3

This example demonstrates using the `stats()` function with aggregation where all of the sub-aggregators output multiple rows, resulting in a Cartesian product.

  * **Query Example**

    ```
    kvParse()
    | stats([
        table([x,y]),
        table([z])
    ])
    ```

  * **Input Event Data**
    `"x=1 y=10 z=100"`
    `"x=2 y=20 z=200"`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs.
    3.  `| stats([table([x,y]), table([z])])`: Computes aggregate functions using `table()` over the fields `x`, `y`, and `z`. Since both `table([x,y])` and `table([z])` sub-aggregators output multiple rows, the returned result is the Cartesian product, containing all combinations of the results from the sub-aggregators.
    4.  Event Result set.

  * **Summary and Results**
    The query computes multiple aggregate functions over an input. The result is a combination of all outputs (the Cartesian product) in fields named `x`, `y`, and `z`.

  * **Result Event Data**

| x | y | z |
| :--- | :--- | :--- |
| 1 | 10 | 100 |
| 1 | 10 | 200 |
| 2 | 20 | 100 |
| 2 | 20 | 200 |

---

### Calculate Minimum and Maximum Response Times

This example calculates minimum and maximum response times using multiple aggregate functions in square brackets, which is shorthand for `stats()`.

  * **Query Example**

    ```
    [min_response:= min(responsetime), max_response:= max(responsetime)]
    ```

  * **Input Event Data**

    | @timestamp | endpoint | responsetime | status\_code |
    | :--- | :--- | :--- | :--- |
    | 1686837825000 | /api/users | 145 | 200 |
    | 1686837826000 | /api/products | 892 | 200 |
    | 1686837827000 | /api/orders | 167 | 200 |
    | 1686837828000 | /api/payment | 1290 | 500 |
    | 1686837829000 | /api/users | 156 | 200 |
    | 1686837830000 | /api/items | 78 | 200 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `[min_response:= min(responsetime), max_response = max(responsetime)]`: In a single operation, calculates the minimum value from the `responsetime` field and returns the result in a field named `min_response`. It also calculates the maximum value from the `responsetime` field and returns the result in a field named `max_response`. Square brackets allow multiple aggregations to be performed in a single operation (shorthand for `stats()`).
    3.  Event Result set.

  * **Summary and Results**
    The query finds the range of response times by calculating both the minimum and maximum values. The results are returned in fields with names specified in the field assignments. This query is useful for monitoring service performance, identifying outliers in response times, or establishing performance baselines. Only one row is returned containing both calculated values.

  * **Result Event Data**

| min\_response | max\_response |
| :--- | :--- |
| 78 | 1290 |

---

### Count Total Events

This example counts the total number of events in the result set using the `stats()` function.

  * **Query Example**

    ```
    stats(function=count())
    ```

  * **Input Event Data**

    | @timestamp | status\_code | endpoint | response\_time |
    | :--- | :--- | :--- | :--- |
    | 1686837825000 | 200 | /api/users | 145 |
    | 1686837826000 | 404 | /api/products | 89 |
    | 1686837827000 | 200 | /api/orders | 167 |
    | 1686837828000 | 500 | /api/payment | 890 |
    | 1686837829000 | 200 | /api/users | 156 |
    | 1686837830000 | 404 | /api/items | 78 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `stats(function=count())`: Counts the total number of events in the result set. The `count()` function is passed as an argument to `stats()` and returns the count in a field named `_count`. This query is actually equivalent to just `count()`.
    3.  Event Result set.

  * **Summary and Results**
    The query gets a simple count of the total number of events matching the query. This is useful for monitoring event volumes, verifying data ingestion, or getting quick counts of specific event types when combined with filters. Only one row is returned containing the total count.

  * **Result Event Data**

| \_count |
| :--- |
| 6 |

# stdDev()

The `stdDev()` function calculates the standard deviation for a field over a set of events. The result is returned in a field named `stddev`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `stddev` | Name of output field. |
| `field` | string | required | Field to extract a number from and calculate standard deviation over. |

***

## Function Operation

The `stdDev()` function computes the standard deviation of numerical values within a specified field across a set of events. Standard deviation is a measure of how widely values are dispersed from their average value. A lower standard deviation indicates that the data points tend to be close to the mean (average) of the set, while a higher standard deviation indicates that the data points are spread out over a wider range of values.

The function returns a single numerical value representing the calculated standard deviation. This result is outputted into a new field, which by default is named `stddev`, but a custom name can be assigned using the `as` parameter. The unit of measurement for the standard deviation will be the same as the input field. This function is particularly useful in statistical analysis for understanding the variability and distribution of data, such as identifying unusual patterns or establishing normal ranges for metrics.

***

## Examples

### Find the standard deviation of bytes sent in http responses

This is a basic example of using `stdDev()` to calculate the standard deviation of `bytes_sent`.

  * **Query Example**
    ```
    stdDevBytes := stdDev(field=bytes_sent)
    ```

### Calculate Standard Deviation of Bytes Sent

This example uses the `stdDev()` function to calculate how much the number of bytes sent varies from the mean value.

  * **Query Example**

    ```
    stdDevBytes := stdDev(field=bytes_sent)
    ```

  * **Input Event Data**

| @timestamp | endpoint | bytes\_sent | status\_code |
| :--- | :--- | :--- | :--- |
| 1686837825000 | /api/users | 1450 | 200 |
| 1686837826000 | /api/products | 8920 | 200 |
| 1686837827000 | /api/orders | 1670 | 200 |
| 1686837828000 | /api/payment | 12900 | 500 |
| 1686837829000 | /api/users | 1560 | 200 |
| 1686837830000 | /api/items | 780 | 200 |
| 1686837831000 | /api/orders | 9340 | 200 |
| 1686837832000 | /api/checkout | 9230 | 200 |
| 1686837833000 | /api/products | 1340 | 200 |
| 1686837834000 | /api/users | 4450 | 200 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `stdDevBytes := stdDev(field=bytes_sent)`: Calculates the standard deviation of values in the `bytes_sent` field and assigns the result to a new field named `stdDevBytes`. The `stdDev()` function measures how widely the values are dispersed from their average value.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to understand the variability in the size of data being transferred. This is useful for identifying unusual patterns in data transfer sizes, establishing normal ranges for network traffic, or detecting anomalies in data transmission. The result is a single value representing the standard deviation. A higher value indicates greater variation in the data. The unit of measurement is the same as the input field (bytes in this case).

  * **Result Event Data**

| stdDevBytes |
| :--- |
| 4289.32 |

# stripAnsiCodes()

[cite\_start]The `stripAnsiCodes()` function removes ANSI color codes and movement commands. [cite: 2401]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional | [cite\_start]Name of output field, default is to replace contents of input field. [cite: 2402] |
| `field` | string | required \<br\> default: `@rawstring` | [cite\_start]Specifies the field where to remove ANSI escape codes. [cite: 2402] |

***

## Function Operation

[cite\_start]The `stripAnsiCodes()` function is used to clean up log data by removing ANSI escape codes, which are often used for text formatting (like colors) and cursor movement in terminal outputs. [cite: 2401] [cite\_start]By default, this function operates on the `@rawstring` field, but you can specify a different input `field` to clean. [cite: 2402] [cite\_start]The cleaned string can either replace the contents of the input field or be stored in a new output field if the `as` parameter is used. [cite: 2402] This ensures that the extracted data is plain text, making it easier to read and analyze without visual clutter from control characters.

***

## Examples

### Remove the ANSI escape codes from the message field

This example demonstrates how to remove ANSI escape codes from a specific field named `message`.

  * **Query Example**

    ```
    message := "\x1b[93;41mColor"
    stripAnsiCodes(message)
    | @display:= message
    ```

  * **Step-by-Step**

    1.  `message := "\x1b[93;41mColor"`: A field named `message` is created and assigned a string containing an ANSI color code for "Color".
    2.  `stripAnsiCodes(message)`: This function is applied to the `message` field, removing the ANSI escape code (`\x1b[93;41m`).
    3.  `| @display:= message`: The cleaned `message` field is then aliased to `@display` for visualization purposes.

  * **Result Event Data**
    (Implicitly, the `message` field will now contain only "Color" without the ANSI codes, and `@display` will reflect this cleaned value.)

---

### Remove all ANSI escape codes from @rawstring

This example shows how to remove ANSI escape codes from the raw event string (`@rawstring`).

  * **Query Example**

    ```
    stripAnsiCodes()
    ```

  * **Step-by-Step**

    1.  `stripAnsiCodes()`: Since no `field` parameter is specified, the function defaults to processing `@rawstring`. It removes all ANSI escape codes and movement commands found within the `@rawstring` of each event.

  * **Result Event Data**
    (The `@rawstring` field in each event will be updated to contain the content without ANSI codes.)

# subnet()

The `subnet()` function computes a subnet from an IPv4 field; by default, it emits it into a subnet field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `subnet` | Specifies the name of the output field. |
| `bits` | number | required | Specifies the prefix bits to include in the subnet, for example, 23. |
| `field` | string | required | Specifies the input field. |

***

## Function Operation

The `subnet()` function takes an IPv4 address from a specified input `field` and calculates its corresponding subnet. The size of the subnet is determined by the `bits` parameter, which defines the number of prefix bits to include (e.g., 24 for a /24 subnet). The computed subnet address is then outputted into a new field. By default, this output field is named `subnet`, but a custom name can be assigned using the `as` parameter. This function is useful for network analysis, security logging, and grouping events by network segments.

***

## Examples

### Compute subnet for `ipAddress` using 23-bit prefix; emit into `subnet` field

This example demonstrates how to compute a subnet from an IP address field using a 23-bit prefix and store the result in a field named `subnet`.

  * **Query Example**

    ```
    subnet(ipAddress, bits=23, as=subnet)
    ```

  * **Step-by-Step**

    1.  The query targets an `ipAddress` field within events.
    2.  `subnet(ipAddress, bits=23, as=subnet)`: Calculates the subnet of the `ipAddress` field using a 23-bit prefix. The resulting subnet address is stored in a new field named `subnet`.

  * **Summary and Results**
    This query computes a subnet for a given IPv4 address field, specifying the prefix length and the output field name.

# sum()

The `sum()` function calculates the sum for a field over a set of events. The result is returned in a field named `sum`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `sum` | Name of output field. |
| `field` | string | required | Field to extract a number from and sum over. |

***

## Function Operation

The `sum()` function is an aggregate function that computes the total sum of all numerical values within a specified field across a given set of events. The result of this calculation is a single numerical value, which is then returned in a new output field. By default, this output field is named `_sum`, but you can customize its name using the `as` parameter. This function is frequently used in conjunction with other aggregate functions like `bucket()` or `groupBy()` to sum values over time intervals or within specific groups, respectively.

***

## Examples

### How many bytes did our webserver send per minute

This is a basic example of using `sum()` with `bucket()` to calculate the total bytes sent per minute.

  * **Query Example**
    ```
    bucket(function=sum(bytes_sent))
    ```

### Bucket Events Into Groups

This example uses the `bucket()` function to request 24 buckets over a period of one day and then sums the "count" field within each bucket.

  * **Query Example**

    ```
    bucket(buckets=24, function=sum("count"))
    parseTimestamp(field=_bucket, format=millis)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `bucket(buckets=24, function=sum("count"))`: Buckets the events into 24 groups spanning over a period of one day, using the `sum()` function on the `count` field.
    3.  `parseTimestamp(field=_bucket, format=millis)`: Extracts the timestamp from the generated `_bucket` field and converts it to a date-time value. The `bucket` function outputs the timestamp as an epoch value in the `_bucket` field. This results in an additional bucket containing all the data after the requested timespan for the requested number of buckets.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to optimize data storage and query performance by making it easier to manage and locate data subsets when performing analytics tasks. The resulting outputs show 25 buckets; the original requested 24 buckets and in addition the bucket for the extracted timestamp.

  * **Result Event Data**

| bucket | \_sum | @timestamp |
| :--- | :--- | :--- |
| 1681290000000 | 1322658945428 | 1681290000000 |
| 1681293600000 | 1879891517753 | 1681293600000 |
| 1681297200000 | 1967566541025 | 1681297200000 |
| 1681300800000 | 2058848152111 | 1681300800000 |
| 1681304400000 | 2163576682259 | 1681304400000 |
| 1681308000000 | 2255771347658 | 1681308000000 |
| 1681311600000 | 2342791941872 | 1681311600000 |
| 1681315200000 | 2429639369980 | 1681315200000 |
| 1681318800000 | 2516589869179 | 1681318800000 |
| 1681322400000 | 2603409167993 | 1681322400000 |
| 1681326000000 | 2690189000694 | 1681326000000 |
| 1681329600000 | 2776920777654 | 1681329600000 |
| 1681333200000 | 2873523432202 | 1681333200000 |

---

### Calculate Total Network Bandwidth Per Host

This example calculates total inbound and outbound network traffic per host using nested `sum()` functions with `groupBy()`, then calculates the total bandwidth consumption, and finally sorts by total traffic.

  * **Query Example**

    ```
    event_simpleName="NetworkConnectStats"
    groupBy([ComputerName], function=[
        sum(field="BytesReceived", as=InboundTraffic),
        sum(field="BytesSent", as=OutboundTraffic)
    ])
    TotalTraffic := InboundTraffic + OutboundTraffic
    sort(field="TotalTraffic", order="desc")
    ```

  * **Input Event Data**

    | @timestamp | event\_simpleName | ComputerName | BytesReceived | BytesSent |
    | :--- | :--- | :--- | :--- | :--- |
    | 1686837825000 | NetworkConnectStats | DESKTOP-A1 | 15000000 | 8000000 |
    | 1686837825000 | NetworkConnectStats | DESKTOP-A1 | 8900000 | 4600000 |
    | 1686837825000 | NetworkConnectStats | LAPTOP-B2 | 25000000 | 12500000 |
    | 1686837826000 | NetworkConnectStats | SERVER-C3 | 95000000 | 47000000 |
    | 1686837826000 | NetworkConnectStats | DESKTOP-A1 | 12000000 | 6000000 |
    | 1686837826000 | NetworkConnectStats | LAPTOP-B2 | 18000000 | 9000000 |
    | 1686837827000 | NetworkConnectStats | SERVER-C3 | 85000000 | 42000000 |
    | 1686837827000 | NetworkConnectStats | DESKTOP-D4 | 5000000 | 2500000 |
    | 1686837827000 | NetworkConnectStats | LAPTOP-B2 | 22000000 | 11000000 |
    | 1686837828000 | NetworkConnectStats | SERVER-C3 | 105000000 | 50000000 |
    | 1686837828000 | NetworkConnectStats | DESKTOP-D4 | 6500000 | 3500000 |
    | 1686837828000 | NetworkConnectStats | DESKTOP-A1 | 9800000 | 5000000 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `event_simpleName="NetworkConnectStats"`: Filters events to include only those where `event_simpleName` equals `NetworkConnectStats`.
    3.  `groupBy([ComputerName], function=[sum(field="BytesReceived", as=InboundTraffic), sum(field="BytesSent", as=OutboundTraffic)])`: Groups the data by the `ComputerName` field and calculates two aggregate values: the sum of `BytesReceived` (stored in a field named `InboundTraffic`), and the sum of `BytesSent` (stored in a field named `OutboundTraffic`).
    4.  `TotalTraffic := InboundTraffic + OutboundTraffic`: Creates a new field named `TotalTraffic` containing the sum of `InboundTraffic` and `OutboundTraffic` values.
    5.  `sort(field="TotalTraffic", order="desc")`: Sorts the results based on the `TotalTraffic` field in descending order, showing hosts with the highest bandwidth consumption first.
    6.  Event Result set.

  * **Summary and Results**
    The query is used to analyze network bandwidth consumption patterns across different hosts in the network. This is useful for identifying hosts consuming excessive bandwidth, monitoring network usage patterns, or detecting potential network-intensive applications or anomalies. The traffic values are in bytes and each row represents the aggregated traffic for a unique host.

  * **Result Event Data**

| ComputerName | InboundTraffic | OutboundTraffic | TotalTraffic |
| :--- | :--- | :--- | :--- |
| SERVER-C3 | 285000000 | 139000000 | 424000000 |
| LAPTOP-B2 | 65000000 | 32500000 | 97500000 |
| DESKTOP-A1 | 45700000 | 24600000 | 70300000 |
| DESKTOP-D4 | 11500000 | 6000000 | 17500000 |

---

### Calculate a Percentage of Successful Status Codes Over Time

This example calculates a percentage of successful status codes over time within the `timeChart()` function, using `sum()` and `count()` within an embedded aggregate.

  * **Query Example**

    ```
    success: if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
            [sum(success, as=success), count(as=total)]
            pct_successful := (success/total)*100
            | drop([success, total])
        }
    ], span=15m, limit=100)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `success: if(status >= 500, then=0, else=1)`: Adds a `success` field: if the `status` field's value is greater than or equal to 500, `success` is set to 0 (indicating failure), otherwise it's set to 1 (indicating success).
    3.  `timeChart(series=customer, function=[{...}], span=15m, limit=100)`: Creates a new timechart, generating a new series named `customer` that uses a compound function.
          * `[sum(success, as=success), count(as=total)]`: This embedded aggregate (defined using the square bracket syntax, shorthand for `stats()`) calculates the `sum()` of the `success` field (effectively counting successful events) and the `count()` of all events (total events) within each time bucket, assigning results to fields named `success` and `total` respectively.
          * `pct_successful := (success/total)*100`: Calculates the percentage of successful events within each time bucket. At this point, the process is still within the aggregate context.
          * `| drop([success, total])`: Drops the temporary `success` and `total` fields, which were used only for calculation, from the array generated by the aggregate.
    4.  The `span=15m` sets the time bucket interval to 15 minutes, and `limit=100` limits the overall number of results from the timechart.
    5.  Event Result set.

  * **Summary and Results**
    This query shows how an embedded aggregate can be used to generate a sequence of values that can be formatted (in this case to calculate percentages) and generate a new event series for the aggregate values. It's useful for monitoring performance over time, specifically the success rate of operations based on status codes.

# join()

The `join()` function in LogScale is used to combine two LogScale searches by matching results based on specified keys or fields.

***

## Parameters

| Parameter | Type | Required | Default Value | Description |
| :-------- | :------------- | :-------- | :------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `end` | string | optional | End of main query | Specifies either the timestamp relative to the main query's end (for example, end 2h will be two hours before the end of the main query) or an absolute timestamp in milliseconds since UTC. |
| `field` | array of strings | required | (none) | Specifies which field in the event (log line) must match the given column value. |
| `include` | array of strings | optional | none | Specifies columns to include from the subquery. For events in the subquery that do not have one or more of the named include fields, the output will be the empty string. |
| `key` | array of strings | optional | Value of the field parameter | Specifies which fields of the subquery to join on. Defaults to the value of the `field` parameter. |
| `limit` | number | optional | 100000 | Specifies the maximum number of rows in the subquery. Minimum: 1, Maximum: 200000. |
| `live` | boolean | optional | Same as main query | Controls if the subquery runs as live or static query. |
| `max` | integer | optional | 1 | Maximum number of events found in subquery if several share join key. |
| `mode` | string | optional | inner | Specifies the mode (inner or left) of the join. `inner`: Perform an inner join; return only results that match in both queries. `left`: Perform a left join; all values from the parent query are included, matched to any corresponding events in subquery. |
| `query` | function | required | (none) | The subquery to execute producing the values to join with. The parameter name `query` can be omitted. |
| `repo` | string | optional | Repo of main query | Specify which view/repo in which to perform the subquery. |
| `start` | string | optional | Start of main query | Specifies either the timestamp relative to the main query's end (for example, start 2h will be two hours before the end of the main query) or an absolute timestamp in milliseconds since UTC. |
| `view` | string | optional | View of main query | Specify which view/repo in which to perform the subquery. |

***

## Function Operation

The `join()` function performs two passes over the data: one for the primary query and one for the subquery. By default, in `mode=inner`, it acts as a filter, allowing events through only if they match on the join keys. If `mode=left` is specified, events that do not match the join keys will also be included. The `include` parameter extracts specified fields from the subquery's result and adds them to matching events. The `max=N` parameter controls how many rows/events are picked up in the subquery if multiple events share the same join key. Parameters like `start`, `end`, `view`, and `live` can be used to control the subquery's time interval, repository/view, and live query status, inheriting defaults from the primary query. There is also a `limit` parameter to specify the maximum size of the inner query's result set.

* **Warning:** The `join()` function should not be used as part of a live query because the two passes over the data (primary query and subquery) can be executed on different data sets, potentially leading to inconsistent results. When used in a live query, the query will be run in a repeated mode, in which the server chooses the repetition interval based on the resources used by the function. This can impact the liveness of the query, in that long-running repeated queries can be throttled, and thus be less live than expected. LogScale recommends using `defineTable()` with `match()` instead of negated join for filtering items not part of a data set.

***

## Examples

### 1\. Filter For Items Not Part of Data Set Using `join()`

This example uses `join()` with a negation to find all session IDs from data set A that are not present in data set B.

* **Query Example:**

```
#repo=A session_id=*
| !join(query={#repo=B session_id=*}, field=session_id, key=session_id)
```

* **Input Event Data:**

* **Repository A:**

| timestamp | session\_id | user\_name | action | status |
| :------------------ | :---------- | :---------- | :------- | :------ |
| 2025-04-01T07:00:00Z | 123456 | john.doe | login | success |
| 2025-04-01T07:05:00Z | 123457 | jane.smith | download | success |
| 2025-04-01T07:10:00Z | 123458 | mike.jones | upload | failed |
| 2025-04-01T07:15:00Z | 123459 | sara.wilson | login | success |
| 2025-04-01T07:20:00Z | 123460 | bob.brown | logout | success |

* **Repository B:**

| timestamp | session\_id | user\_name | action | status |
| :------------------ | :---------- | :--------- | :------- | :------ |
| 2025-04-01T07:00:00Z | 123456 | john.doe | login | success |
| 2025-04-01T07:05:00Z | 123457 | jane.smith | download | success |
| 2025-04-01T07:20:00Z | 123460 | bob.brown | logout | success |

* **Step-by-Step:**

1.  **Starting with the source repository events.**
2.  **`#repo=A session_id=*`**: This part filters for all events from repository A that have a `session_id` field.
3.  **`| !join(query={#repo=B session_id=*}, field=session_id, key=session_id)`**: This performs a negated join with repository B. It returns sessions that exist in repository A but *not* in repository B. The negation operator `!` makes this an anti-join operation.

* **Result Event Data:**

| timestamp | session\_id | user\_name | action | status |
| :------------------ | :---------- | :---------- | :----- | :------ |
| 2025-04-01T07:10:00Z | 123458 | mike.jones | upload | failed |
| 2025-04-01T07:15:00Z | 123459 | sara.wilson | login | success |
---
#### 2\. Preview Content in a Lookup File With `readFile()` and Filter With `join()`

This example uses `readFile()` to look up a `host_names.csv` file and then filters for host names that are not sending any logs.

* **Query Example:**

```
readFile("host_names.csv")
| !join(query={groupBy(host_name)}, field=host_name, key=host_name, include=[host_name, id])
```

* **Input Event Data:**

`host_names.csv` content:

```
host_name, host_id
DESKTOP-VSKPBK8, 1
FINANCE, 2
homer-xubuntu, 3
logger, 4
DESKTOP-1, 5
DESKTOP-2, 6
DESKTOP-3, 7
```

* **Step-by-Step:**

1.  **Starting with the source repository events.**
2.  **`readFile("host_names.csv")`**: This part displays the content of the `.csv` file.
3.  **`!join(query={groupBy(host_name)}, field=host_name, key=host_name, include=[host_name, id])`**: This filters for host names that do not send any logs.

* **Result Event Data:**

| host\_id | host\_name |
| :------- | :---------- |
| 5 | DESKTOP-1 |
| 6 | DESKTOP-2 |
| 7 | DESKTOP-3 |

# json:prettyPrint()

The `json:prettyPrint()` function in LogScale formats a JSON field for improved readability. This can be an expensive operation, so it is recommended to apply it after filtering your data at the end of the query to prevent unnecessary formatting of data that will be discarded.

***

## Parameters:

| Parameter | Type | Required | Default Value | Description |
| :-------- | :----- | :-------- | :------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `as` | string | optional | (none) | The name of the field to store the output in. |
| `field` | string | optional | `@rawstring` | The name of the field to format. This parameter name can be omitted. |
| `step` | number | optional | 2 | The indentation in number of characters, with a minimum of 2 spaces. |
| `strict` | boolean | optional | `false` | If set to `true`, only valid JSON input will produce a value in the output field. By default, invalid JSON is copied to the output field unmodified. |

***

## Function Operation:

The `json:prettyPrint()` function takes a JSON field and formats it for better readability. If no field is specified, the `@rawstring` field will be formatted. If the specified field does not contain valid JSON, the unmodified input value is stored in the output field, unless the `strict` parameter is set to `true`. When `strict` is `true`, invalid JSON input will not produce a value in the output field. It is advised to use this function after filtering data to reduce processing overhead.

***

## Examples:

### 1\. Format JSON

This example demonstrates how to format JSON in the `@rawstring` field using the `json:prettyPrint()` function.

* **Query Example:**

```
#type=json
| account = 123
| json:prettyPrint()
```

* **Input Event Data:**
(No input event data provided in the source for this example, only the query and steps.)

* **Step-by-Step:**

1.  Start with the source repository events.
2.  `#type=json`: This filters for events with the JSON type.
3.  `| account = 123`: This filters for events related to account `123`. It is recommended to filter the event set as much as possible before using `json:prettyPrint()` to prevent unnecessary formatting of discarded data.
4.  `| json:prettyPrint()`: This formats the JSON content for improved readability. Without a specified field, it formats the `@rawstring` field. If the input is not valid JSON, it returns unmodified values; to prevent this, the `strict` parameter can be set.
5.  Event Result set.

* **Result Event Data:**
The query is used to make JSON data more readable in the results. Formatting JSON in the `@rawstring` field after filtering data is important as it is a resource-intensive operation. Without the `strict` parameter set to `true`, the `json:prettyPrint()` function attempts to format even invalid JSON, which might lead to unexpected results.

---

### 2\. Format Only Valid JSON

This example shows how to format only valid JSON data using the `json:prettyPrint()` function with the `strict` parameter.

* **Query Example:**

```
formattedJson := json:prettyPrint(field=message, strict=true)
```

* **Input Event Data:**
(No input event data provided in the source for this example, only the query and steps.)

* **Step-by-Step:**

1.  Start with the source repository events.
2.  `formattedJson := json:prettyPrint(field=message, strict=true)`: This uses the `field` parameter to specify the `message` field as the source of JSON data and sets the `strict` parameter to `true` to only process valid JSON. It formats the valid JSON data for improved readability and assigns the results to a new field named `formattedJson`. If the JSON in the `message` field is invalid, the `formattedJson` field will not be created for that event because the `strict` parameter is set to `true`.
3.  Event Result set.

* **Result Event Data:**
The query is used to make valid JSON data more readable in the results. Without the `strict` parameter set to `true`, the `json:prettyPrint()` function attempts to format even invalid JSON, which might lead to unexpected results.

# kvParse()

The `kvParse()` function in LogScale is used to parse events encoded with key/value pairs into individual fields and values. It can run an extra key/value parser on events.

***

## Parameters:

| Parameter | Type | Required | Default Value | Description |
| :-------- | :------------- | :-------- | :------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `as` | string | optional | (none) | Prefix for all resolved field keys. |
| `excludeEmpty` | array of strings | optional | `false` | If the value of a key is empty, exclude the field. |
| `field` | array of strings | optional | `@rawstring` | Fields that should be key-value parsed. The parameter name `field` can be omitted. |
| `override` | boolean | optional | `false` | Override existing values for keys that already exist in the event. |
| `separator` | string | optional | `=` | The token that separates the key from the value (a single char only). |
| `separatorPadding` | string | optional | `unknown` | Helps the function recognize unquoted empty values and parse them by specifying whether there is a whitespace around the key-value separator (typically). Values: `no` (assumes no whitespace), `unknown` (whether padding is not recognized), `yes` (indicates whitespace). |

***

## Function Operation:

The `kvParse()` function is used to parse key/values that follow specific formats, such as `key=value`, `key="value"`, `key='value'`, or `key value`. Both the key and value can be quoted using single or double quotes, and if quotes are used, they must be terminated. For a log line like `2017-02-22T13:14:01.917+0000 [main thread] INFO UserService creating new user id=123, name='john doe' email=john@doe`, the key/value parser extracts fields like `id=123`, `name=john doe`, and `email=john@doe`. The `field` parameter specifies which fields should be key/value parsed, with `@rawstring` being the default for parsing the raw string.

***

## Examples:

### 1\. Key/value parse a nested field (JSON input)

This example demonstrates how to key/value parse a nested field within a JSON input.

* **Query Example:**

```
parseJson()
kvParse(metadata)
```

* **Input Event Data:**

```json
{
  "service": "paymentService",
  "type": "payment",
  "metadata": "host=server5, transactionID=123, processingTime=100"
}
```

* **Step-by-Step:**

1.  The `parseJson()` function parses the JSON input, making `service`, `type`, and `metadata` accessible as fields.
2.  The `kvParse(metadata)` function then parses the `metadata` field, which contains key/value pairs.

* **Result Event Data:**
(The output event data is not explicitly provided in the source for this example, but the `metadata` field would be parsed into `host`, `transactionID`, and `processingTime` fields.)

---

### 2\. Keep the first value for duplicated keys, with a preset field

This example shows how to handle duplicated keys when a field has a preset value, ensuring the first encountered value is kept.

* **Query Example:**

```
kvParse(onDuplicate=keepFirst, override=true)
```

* **Input Event Data:**

```
name='john doe' name='jane doe'
```

* **Step-by-Step:**

1.  The `kvParse()` function is applied to the input string.
2.  `onDuplicate=keepFirst`: This parameter specifies that if duplicate keys are encountered, the first value for that key should be retained.
3.  `override=true`: This parameter ensures that if a field named `name` already exists (e.g., if it was preset to 'alice'), the value from the `kvParse` operation will override the existing value.

* **Result Event Data:**

```
name: john doe
```

---

### 3\. Parse String as CSV

This example demonstrates how to parse a string that contains key/value pairs and then a CSV-encoded field, using `kvParse()` followed by `parseCsv()`.

* **Query Example:**

```
kvParse()
parseCsv(result, columns=[count, status, completion, precision, sourcetask])
```

* **Input Event Data:**

```
2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117, success,27%,3.14"
```

* **Step-by-Step:**

1.  `kvParse()`: This parses the initial string into key-value pairs. In this input, it would likely extract `result="117, success,27%,3.14"`.
2.  `parseCsv(result, columns=[count, status, completion, precision, sourcetask])`: This then takes the `result` field (extracted by `kvParse()`) and parses its CSV-encoded content into the specified columns. It adds fields like `count` with value `117`, `status` with `success`, `completion` with `27%`, and `precision` with `3.14`.

* **Result Event Data:**

| completion | count | precision | result | status |
| :--------- | :---- | :-------- | :----- | :------- |
| 27% | 117 | 3.14 | 117, success, 27%, 3.14 | success |

# length()

Computes the number of characters in a string field.

***

## Parameters

| Parameter | Type | Required | Default Value | Description |
|---|---|---|---|---|
| `as` | string | optional | length | Name of output field. |
| `field` | string | required | | The name of the input field to length. |

***

## Function Operation
The `length()` function computes the number of characters in a string field. This count includes all visible characters, spaces, and punctuation marks in the log entry.

***

## Examples

### 1. Count Characters in Field

* **Query Example**: `length(@rawstring)`
* **Step-by-step**:
    1.  Starting with the source repository events, the query `length(@rawstring)` counts the number of characters in the `@rawstring` field and outputs the result in a field named `length`.
    2.  This count includes all visible characters, spaces, and punctuation marks in the log entry.
    3.  Using the `as` parameter, it is also possible to define another output field, for example, `rawLength`, by adding `length(@rawstring, as="rawLength")`.
* **Summary**: The query counts all characters (visible characters, spaces, and punctuation marks) in a log entry. This is useful for managing and analyzing security logs, ensuring complete data capture for threat detection and incident response.

# linReg()

The linReg() function calculates a linear relationship between two variables by using least-squares fitting.

***

## Parameters

| Parameter | Type | Required | Default Value | Description |
|---|---|---|---|---|
| `prefix` | string | optional | | Prefix for all output field names. |
| `X` | string | required | | Specifies the field name that contains the independent variable. |
| `Y` | string | required | | Specifies the field name that contains the dependent variable. |

***

## Function Operation
The `linReg()` function calculates a linear relationship between two variables by using least-squares fitting. The function creates this relationship between x and y variables: $y = \\text{slope} \* x + \\text{intercept}$. The result is outputted in fields named `_slope` and `_intercept`. Also outputted is the adjusted R-squared value in a field named `_r2` and the number of data points in a field named `_n`. These four key values indicate relationship strength and reliability. Note that if all x values are the same or if all y values are the same, then the function cannot calculate results, therefore, nothing is outputted.

***

## Examples

### 1. Calculate Relationship Between X And Y Variables
This example calculates the linear relationship between message size and transmission time using the `linReg()` function.

  * **Query**: `linReg(x=bytes_sent, y=send_duration)`

  * **Input Event Data**:

| @timestamp | bytes\_sent | send\_duration |
|---|---|---|
| 2025-04-07 13:00:00 | 1024 | 0.15 |
| 2025-04-07 13:00:01 | 2048 | 0.25 |
| 2025-04-07 13:00:02 | 4096 | 0.45 |
| 2025-04-07 13:00:03 | 8192 | 0.85 |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  This correlates `bytes_sent` with `send_duration`, showing the relationship between message size and transmission time (the variables x and y) and outputs the results in fields named `_slope` (slope value), `_intercept` (intercept value), `_r2` (adjusted R-squared value), and `_n` (number of data points). These four key values indicate relationship strength and reliability.
    3.  Event Result set.

  * **Summary and Results**: The query is used to calculate a linear relationship between `bytes_sent` (x variable) and `send_duration` (y variable). Calculating the relationship between size of data transferred and time taken to send data is useful, for example, in trend analysis, performance monitoring, or anomaly detection.

  * **Result Event Data**:

| \_slope | \_intercept | r2 | n |
|---|---|---|---|
| 9.823069852941172E-5 | 0.04276470588235326 | 0.9996897336895081 | 10 |

```
* `_slope` is the additional time needed per byte sent.
* `_intercept` is the baseline transmission time.
* `_r2` is the statistical accuracy of the linear model.
* `_n` is the total number of data points analyzed.
```

---

### 2. Calculate Relationship Between X And Y Variables
This example calculates the linear relationship between server load and total response size using the `linReg()` function with `bucket()`.

  * **Query**:

    ```
    bucket (function=[ sum(bytes_sent, as=x), avg(server_load_pct, as=y)])
    | linReg(x=x, y=y)
    ```

  * **Input Event Data**:

| @timestamp | bytes\_sent | server\_load\_pct |
|---|---|---|
| 2024-01-15T09:00:00Z | 156780 | 45.2 |
| 2024-01-15T09:05:00Z | 234567 | 52.8 |
| 2024-01-15T09:10:00Z | 189234 | 48.6 |
| 2024-01-15T09:15:00Z | 345678 | 65.3 |
| 2024-01-15T09:20:00Z | 123456 | 42.1 |
| 2024-01-15T09:25:00Z | 278901 | 58.7 |
| 2024-01-15T09:30:00Z | 198765 | 51.4 |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  This buckets the data points by time, then calculates the sum of bytes sent for each bucket returning the result in a field named x, and calculates the average server load percentage for each bucket returning the result in a field named y.
    3.  This correlates x with y, showing the relationship between the variables x and y and outputs the results in fields named slope (slope value), intercept (intercept value), r2 (adjusted R-squared value), and n (number of data points). These four key values indicate relationship strength and reliability.
    4.  Event Result set.

  * **Summary and Results**: The query is used to calculate a linear relationship between bytes sent (x variable) and server\_load\_pct (y variable). Calculating the relationship between server load percentage and total response size is useful to identify different operational patterns, such as, for example, performance bottlenecks, resource allocation issues, or to identify system optimization opportunities.

  * **Result Event Data**:

| \_slope | \_intercept | r2 | n |
|---|---|---|---|
| 0.00010617525557193158 | 28.934098111407938 | 0.991172367336835 | 10 |

```
* `_slope` is the rate of change between server load and response size.
* `_intercept` is the baseline relationship value.
* `_r2` is the statistical accuracy of the linear model.
* `_n` is the total number of data points analyzed.
```
---

### 3. Calculate Relationship Between X And Y Variables
This example calculates the linear relationship between server load and each of several types of request types using the `linReg()` function with `bucket()` and `groupBy()`.

  * **Query**:

    ```
    bucket (function=[ avg(server_load_pct, as=y), groupBy(request_type, function=count(as=x))])
    | groupBy(request_type, function=linReg(x=x, y=y))
    ```

  * **Input Event Data**:

| @timestamp | server\_load\_pct | request\_type |
|---|---|---|
| 2024-01-15T09:00:00.000Z | 45.2 | GET |
| 2024-01-15T09:00:00.000Z | 45.2 | POST |
| 2024-01-15T09:00:00.000Z | 45.2 | GET |
| 2024-01-15T09:05:00.000Z | 52.8 | GET |
| 2024-01-15T09:05:00.000Z | 52.8 | PUT |
| 2024-01-15T09:05:00.000Z | 52.8 | POST |
| 2024-01-15T09:10:00.000Z | 48.6 | GET |
| 2024-01-15T09:10:00.000Z | 48.6 | GET |
| 2024-01-15T09:10:00.000Z | 48.6 | DELETE |
| 2024-01-15T09:15:00.000Z | 65.3 | POST |
| 2024-01-15T09:15:00.000Z | 65.3 | POST |
| 2024-01-15T09:15:00.000Z | 65.3 | GET |
| 2024-01-15T09:20:00.000Z | 42.1 | GET |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  This buckets the data points by time, then calculates the average server load for each time bucket returning the result in a field named y. It also groups the request types in a field named request\_type and makes a count of requests by type in each time bucket returning the result in a field named x.
    3.  This correlates x with y, showing the relationship between the variables x and y for each HTTP request type and outputs the results in fields named slope (slope value), intercept (intercept value), r2 (adjusted R-squared value), and n (number of data points). These four key values indicate relationship strength and reliability.
    4.  Event Result set.

  * **Summary and Results**: The query is used to analyze how different HTTP request types affect server load. The analysis helps identify which HTTP request types have the strongest impact on server performance.

  * **Result Event Data**:

| request\_type | \_slope | \_intercept | r2 | n |
|---|---|---|---|---|
| DELETE | \<no value\> | \<no value\> | \<no value\> | \<no value\> |
| GET | -13.749999999999941 | 72.7999999999999 | 0.5941824574313592 | 5 |
| POST | 16.29999999999992 | 32.70000000000012 | 0.7196207242484238 | 3 |
| PUT | \<no value\> | \<no value\> | \<no value\> | \<no value\> |

```
* `_slope` is the impact rate of request volume on server load.
* `_intercept` is the baseline server load when there are no requests of a specific type.
* `_r2` is the statistical accuracy of the relationship.
* `_n` is the total number of data points analyzed.
```

# lower()

The lower() function converts text to lower case letters. It can process text from event fields or other sources. By default, it uses the system locale, but it is possible to specify a different language and locale if needed.

***

## Parameters

| Parameter | Type | Required | Default Value | Description |
|---|---|---|---|---|
| `as` | string | optional | lower | The name of the output field. |
| `field` | string | required | | The name of the input field with the value to convert to lower case. |
| `locale` | string | optional | system locale | Locale to use, as ISO-639 language and an optional ISO-3166 country (for example, en or en\_US). |
| `type` | string | optional | | The name of the locale to use as ISO 639 language and an ISO 3166 country. When not specified, it uses the system locale. |

***

## Function Operation
The `lower()` function converts text to lowercase letters. It can process text from event fields or other sources. By default, it uses the system locale, but it is possible to specify a different language and locale if needed. In addition to providing the field of events to change to all lowercase letters, as well as optionally assigning a name to the resulting field, you can specify the country and language so that conversion is done correctly and without odd characters. For the value of `type`, you can specify just the language, or you can refine that choice by including the country (e.g., "en" for English, "en\_UK" for UK English, "en\_US" for US English). Specifying the correct locale is particularly important for languages with non-Latin alphabets, such as Russian with Cyrillic letters.

***

## Examples

### Create New Array by Appending Expressions
This example demonstrates creating a new flat array by appending new expressions using the `array::append()` function in conjunction with `lower()`.

  * **Query**:

    ```
    array::append(array="related.user[]", values=[lower(source.user.name), lower(destination.user.name)])
    ```

  * **Input Event Data**:
    `source.user.name="user_1" destination.user.name="USER_2"`

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  The query `array::append(array="related.user[]", values=[lower(source.user.name), lower(destination.user.name)])` creates a new array `related.user[]` containing information about all user names seen on the event. Notice that the `lower()` function formats the results into lowercase before appending them to the array.
    3.  Event Result set.

  * **Summary and Results**: This query is used to create a new flat array based on values from an array of expressions.

  * **Result Event Data**:

| source.user.name | destination.user.name | related.user[0] | related.user[1] |
|---|---|---|---|
| user\_1 | USER\_2 | user\_1 | user\_2 |

---

### Format a String to Upper Case and Lower Case
This example uses `upper()` and `lower()` functions with `concat()` to concatenate two fields containing error messages, where one field's result is all lowercase letters and the other field's results are all uppercase letters.

  * **Query**:

    ```
    lower(@error_msg[0], as=msg1)
    | upper(@error_msg[1], as=msg2)
    | concat([msg1, msg2], as=test)
    ```

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `lower(@error_msg[0], as=msg1)`: Formats the first element (index 0) of the `@error_msg` array to lowercase and returns the results in a field named `msg1`.
    3.  `| upper(@error_msg[1], as=msg2)`: Formats the second element (index 1) of the `@error_msg` array to uppercase and returns the results in a field named `msg2`.
    4.  `| concat([msg1, msg2], as=test)`: Concatenates (combines) the values in field `msg1` and field `msg2`, and returns the concatenated string in a new field named `test`.
    5.  Event Result set.

  * **Summary and Results**: The query is used to either convert strings to lowercase or uppercase and return the new concatenated strings/results in a new field, in this example, concatenating error messages. The specific labeling of `msg1` and `msg2` is particularly useful when you have more than one field that uses the same query function. By converting fields to consistent cases, it helps standardize data for easier analysis and comparison. The concatenation allows you to combine multiple fields into a single field, which can be useful for creating unique identifiers or grouping related information.

---

### Standardize Values And Combine Into Single Field
This example standardizes values using the `upper()` and `lower()` functions and combines them into a single field with `concat()`.

  * **Query**:

    ```
    lower(#severity, as severity)
    | upper (#category, as category)
    | concat([severity, category], as=test)
    | top(test)
    ```

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `lower(#severity, as severity)`: Converts the values in the `#severity` field to lowercase and returns the results in a field named `severity`.
    3.  `| upper (#category, as category)`: Converts the values in the `#category` field to uppercase and returns the results in a field named `category`.
    4.  `concat([severity, category], as=test)`: Concatenates (combines) the values in field `category` and field `severity`, and returns the concatenated string in a new field named `test`.
    5.  `| top(test)`: Finds the most common values of the field `test`—the top of an ordered list of results—along with their count. The result of the count of their occurrences is displayed in a field named `_count`.
    6.  Event Result set.

  * **Summary and Results**: The query is used to standardize the format of the values in the fields `#category` and `severity` and concatenate the values into a single field, showing which combinations of severity and category are most common in the data. The specific labeling of `category` and `severity` is particularly useful when you have more than one field that uses the same query function. By converting fields to consistent cases, it helps standardize data for easier analysis and comparison. The concatenation allows you to combine multiple fields into a single field, which can be useful for creating unique identifiers or grouping related information. It provides a quick overview of the distribution of events across different severity-category combinations.

  * **Result Event Data**:

| test | \_count |
|---|---|
| infoALERT | 90005 |
| infoFILTERALERT | 36640 |
| errorALERT | 17256 |
| warningGRAPHQL | 14240 |
| warningALERT | 13617 |
| warningSCHEDULEDSEARCH | 11483 |
| infoSCHEDULEDSEARCH | 5917 |
| warningFILTERALERT | 1646 |
| errorFILTERALERT | 1487 |
| infoACTION | 3 |

```
Notice how the value of `#severity` is in lowercase letters, and the value of `#category` is in uppercase letters.
```

# lowercase()

The `lowercase()` function is deprecated for use in query contexts and is meant for use in parsers only. For uses in queries, the `lower()` function should be used. It lowercases the contents of either or both of the field names or values of a string field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | array of strings | required | The name of the input field or fields (in `[]`) to lowercase. Use the special value `*` as the only field for ALL fields. When in this mode, only the lowercased fields remain. |
| `include` | string | optional \<br\> default: `values` | What to lowercase.\<br\>**Values:**\<br\>`both`: Convert both the values and field names to lowercase.\<br\>`fields`: Convert the field names to lowercase.\<br\>`values`: Convert the values of the fields to lowercase.|
| `locale` | string | optional | The name of the locale to use, as ISO 639 language and an optional ISO 3166 country, such as `da`, `da_DK` or `en_US`. When not specified, uses the system locale.|

***

## Function Operation

The `lowercase()` function is primarily intended for use within parsers, not directly in query contexts. It converts the contents of string field values, field names, or both, to lowercase. If `field` is set to `*`, only the lowercased fields will remain. The `locale` parameter allows specifying a particular language and country for correct lowercase conversion, otherwise, the system locale is used.

***

## Examples

### Lowercase Field Value

This example shows how to lowercase the value of a specific field.

  * **Query Example**

    ```
    lowercase("Bar")
    ```

  * **Input Event Data**
    (Assume an event with a field named `Bar-CONTENTS` where the value is "CONTENTS")

  * **Result Event Data**
    (The `Bar` field's content will be "contents")

---

### Lowercase Field Value (explicit `include`)

This example demonstrates lowercasing only the value of a field, explicitly specifying the `include` parameter.

  * **Query Example**

    ```
    lowercase("BaR", include="values")
    ```

  * **Input Event Data**
    (Assume an event with a field named `BAR CONTENTS` where the value is "CONTENTS")

  * **Result Event Data**
    (The `BaR` field's content will be "contents", while the field name `BAR` remains "CONTENTS")

---

### Lowercase Multiple Fields (both names and values)

This example shows how to lowercase both the field names and their values for specified fields.

  * **Query Example**

    ```
    lowercase(field=["foo", "bar"], include="both")
    ```

  * **Input Event Data**
    (Assume an event with fields `Foo` with value "VALUE1" and `BAR` with value "VALUE2")

  * **Result Event Data**
    (Fields `foo` with value "value1" and `bar` with value "value2" will be present)

---

### Lowercase All Fields (both names and values)

This example demonstrates lowercasing both the field names and their values for all fields in an event.

  * **Query Example**

    ```
    lowercase(field="*", include="both")
    ```

  * **Input Event Data**
    (Assume an event with a field named `BAR CONTENTS` where the value is "CONTENTS")

  * **Result Event Data**
    (The field name will be `bar contents` and its value will be "contents". The original `BAR CONTENTS` field will no longer be present.)

# match()

The `match()` function matches or joins data from query results with a table. The table can be provided either as a lookup CSV file or through a limited form of JSON file, uploaded using Lookup Files or as an ad-hoc table using Ad-hoc Tables.

-----

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `column` | optional string, field, or array | optional | Which column in the file to use for the match. A single column or an array of columns can be specified. |
| `field` | required string or array | required | Which field in the event (log line) must match the given column value. A single field or an array of fields can be specified. Field and column must have the same length, are matched in order and must all match. |
| `file` | required string | required | Specifies the source file (when using Look Up files) or the name of the ad-hoc table. The file name should be specified with `.csv` or `.json` suffix. |
| `glob` | optional boolean | optional \<br\> default: `false` | This parameter is deprecated. Use `mode` parameter with the `glob` option instead. |
| `ignoreCase` | optional boolean | optional \<br\> default: `false` | If true, ignore case when matching against the CSV data. \<br\>**Values:**\<br\>`false`: Perform a case-sensitive match of the event values with the lookup table. `true`: Ignore the case of the event values with the lookup table. |
| `include` | optional string or array | optional | The columns to include. If no argument is given, include all columns from the corresponding row in the output event. |
| `mode` | optional string | optional | The function to use when matching against keys. \<br\>**Values:**\<br\>`cidr`: The key is interpreted as a CIDR subnet and the event is matched if the field contains an IP within the subnet. If multiple subnets match, the most specific one is selected or an arbitrary one if there are multiple equally specific subnets. \<br\>`glob`: The key is interpreted as a globbing pattern and matched accordingly, for example, a CSV key value of `*thisMatch*` would match the field value of `123thisMatch456`. When using `mode=glob`, the underlying CSV is limited to 20,000 rows/lines. For self-hosted customers, the maximum value for glob matches is configurable using `GLOB_MATCH_LIMIT`. \<br\>`string`: The matching is done using exact string matching. |
| `strict` | optional boolean | optional \<br\> default: `true` | If true (the default) selects only the fields that match a key in the file; if false lets all events through (works like the deprecated `lookup()`). |

-----

## Function Operation

The `match()` function is used to join or match data from query results with an external table. This table can be either a CSV file or a JSON file.

For CSV files, whitespace is included in keys and values, and commas in values should be quoted. The first line of a CSV is interpreted as column titles. When using `match()` with a single column, the last matching row in the CSV is used.

For JSON files, two formats are supported: object-based and array-based. In the object-based variant, the lookup values are declared as an object with a key and embedded fields, where the key field does not have a name; the key for each value is used for matching. Nested JSON objects are not supported and will not be matched, though uploading a file with them will not fail. In the array-based variant, lookup values are an array of objects, and the key field is selected using the `column` parameter in `match()`. Any field in the JSON file can be used as the match value.

The default behavior of `match()` when `strict` is set to `true` works like an `INNER JOIN`, selecting only fields that match a key in the file. When `strict` is set to `false`, the function enriches events, allowing all events through but only enriching those with matches.

The `mode` parameter controls the matching behavior:

  * `cidr`: Interprets the key as a CIDR subnet and matches if the event field contains an IP within that subnet. If multiple subnets match, the most specific one is selected.
  * `glob`: Interprets the key as a globbing pattern. When using `mode=glob`, the underlying CSV is limited to 20,000 rows/lines.
  * `string`: Performs exact string matching.

The `match()` function does not report an error if the file format cannot be parsed. Up until LogScale version 1.140, the file is limited to 1,000,000 rows/lines for exact matching using `mode=string`.

-----

## Examples

### Match Multiple Pairs of Event Fields Against Multiple Columns in .CSV Lookup File

This example demonstrates how to compare multiple pairs of event fields against multiple columns in a CSV lookup file simultaneously using the `match()` function.

  * **Query Example**

    ```
    match(test.csv, field=[field1, field2], column=[column1, column2])
    ```

  * **Input Event Data**
    (Example `test.csv` file data)

    | column1 | column2 | column3 |
    | :--- | :--- | :--- |
    | a | b | d |
    | C | e | f |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  For each event, `match(test.csv, field=[field1, field2], column=[column1, column2])` checks if `field1` matches `column1` and `field2` matches `column2`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to match multiple pairs of event fields against multiple columns in the `.CSV` file named `test.csv`. Multiple field matching helps validate and enrich complex event data.

  * **Result Event Data**

    | column3 | field1 | field2 |
    | :--- | :--- | :--- |
    | f | C | e |

---

### Filter For Items Not Part of Data Set Using `match()`

This example shows how to find the set difference using the `match()` function with negation to search and find IP addresses that are not part of a known list (`known_ips.csv`).

  * **Query Example**

    ```
    src_ip=*
    | !match("known_ips.csv", field=src_ip)
    ```

  * **Input Event Data**
    (Example incoming data)

    | timestamp | src\_ip | dst\_ip | src\_port | dst\_port | protc |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-04-01T07:00:00Z | 192.168.1.101 | 10.0.0.50 | 52431 | 443 | TCP |
    | 2025-04-01T07:00:01Z | 172.16.0.24 | 8.8.8.8 | 33221 | 53 | UDP |
    | 2025-04-01T07:00:02Z | 192.168.1.150 | 172.16.0.100 | 49223 | 80 | TCP |
    | 2025-04-01T07:00:03Z | 10.0.0.75 | 192.168.1.1 | 55678 | 22 | TCP |
    | 2025-04-01T07:00:04Z | 192.168.1.200 | 1.1.1.1 | 44556 | 53 | UDP |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `src_ip=*`: Filters for all events that have a `src_ip` field.
    3.  `!match("known_ips.csv", field=src_ip)`: Excludes (filters out) any events where the `src_ip` field matches entries in the file `known_ips.csv`, and returns a list of IP addresses that are not found in the specified file. The negation operator is used to return non-matching results.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to search for unknown or unexpected source IP addresses matched up against a known list. This is useful for detecting potential security threats and monitoring for unauthorized network access.

  * **Result Event Data**

    | timestamp | src\_ip | dst\_ip | src\_port | dst\_port | protc |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-04-01T07:00:00Z | 192.168.1.101 | 10.0.0.50 | 52431 | 443 | TCP |
    | 2025-04-01T07:00:01Z | 172.16.0.24 | 8.8.8.8 | 33221 | 53 | UDP |

---

### Match Event Fields Against Lookup Table Values Adding Specific Columns

This example compares event IP fields with CIDR ranges in a lookup table using the `match()` function with the `mode` parameter set to `cidr`. The query matches IP addresses against CIDR blocks (CIDR subnets) and adds specific network information from the columns `info` and `type` to the output rows.

  * **Query Example**

    ```
    match(file="cidr-file.csv", column="cidr-block", field=ip, mode=cidr, include=["info", "type"])
    ```

  * **Input Event Data**
    (Example incoming event data)

    | @timestamp | ip | action |
    | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | 10.0.1.25 | login |
    | 2024-01-15T09:01:00.000Z | 192.168.1.100 | connect |
    | 2024-01-15T09:02:00.000Z | 172.16.5.12 | access |

    (Example `cidr-file.csv` file data)

    | cidr-block | info | type | location | department |
    | :--- | :--- | :--- | :--- | :--- |
    | 10.0.1.0/24 | Internal Network | corporate | HQ | IT |
    | 192.168.1.0/24 | Development Network | test | Lab | Engineering |
    | 172.16.0.0/16 | Production Network | critical | DC | Operations |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `match(file="cidr-file.csv", column="cidr-block", field=ip, mode=cidr, include=["info", "type"])`: Uses CIDR ranges matching to match the `cidr-block` column of the `cidr-file.csv` lookup table file against the IP addresses (`ip` field) in the events, and adds specific network information to the output rows. It will only add the specified columns of the matching row, and the column names become new field names. When the `mode` parameter is set to `cidr`, the event is matched if the field contains an IP address within the CIDR subnet. If multiple subnets match, the most specific one is selected, or an arbitrary one if there are multiple equally specific subnets.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to match IP addresses against CIDR blocks and add specific network information from the columns `info` and `type` to the output rows. This query helps analyze network traffic and security events by mapping IP addresses to network segments.

  * **Result Event Data**

    | @timestamp | ip | action | info | type |
    | :--- | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | 10.0.1.25 | login | Internal Network | corporate |
    | 2024-01-15T09:01:00.000Z | 192.168.1.100 | connect | Development Network | test |
    | 2024-01-15T09:02:00.000Z | 172.16.5.12 | access | Production Network | critical |

    Notice how only the specified fields from the `cidr-file.csv` file appear in output.

---

### Match Event Fields Against Lookup Table Values Allowing All Events to Pass

This example compares event fields with column values in a lookup table using the `match()` function with the `strict` parameter set to `false` to allow also non-matching events to pass. When `strict` is set to `false`, all events (including non-matching ones) are passed through, but only events with matches will be enriched with all other columns of the matching row.

  * **Query Example**

    ```
    id=~ match(file="users.csv", column=userid, strict=false)
    ```

  * **Input Event Data**
    (Example incoming event data)

    | @timestamp | id | action | source\_ip |
    | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 |
    | 2024-01-15T09:01:00.000Z | unknown-user | login\_attempt | 10.0.0.2 |
    | 2024-01-15T09:02:00.000Z | dev-user-456 | code\_push | 10.0.0.3 |

    (Example `users.csv` file data)

    | userid | department | access\_level | location |
    | :--- | :--- | :--- | :--- |
    | ADMIN-123 | IT | administrator | HQ |
    | dev-user-456 | Engineering | developer | Remote |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `id=~ match(file="users.csv", column=userid, strict=false)`: When an event ID matches the column `userid` in the `users.csv` lookup file, all columns from that first matching row are added to the event. The column names become new field names. As the `strict` parameter is set to `false`, all events (including non-matching events) are passed through, but only events with matches will be enriched with all other columns of the matching row.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to enrich matching events while allowing all events to pass through. Matching events against an authorized users list is useful for filtering out unauthorized access attempts, for validation of user activities, or other monitoring. Showing non-matching events in output as well can be useful for tracking unauthorized access attempts, identifying unknown users or systems, or for monitoring suspicious activities.

  * **Result Event Data**

    | @timestamp | id | action | source\_ip | department | access\_level |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 | IT | administrator |
    | 2024-01-15T09:01:00.000Z | unknown-user | login\_attempt | 10.0.0.2 | \<no value\> | \<no value\> |
    | 2024-01-15T09:02:00.000Z | dev-user-456 | code\_push | 10.0.0.3 | Engineering | developer |

    After matching, the output combines original event fields with matched user details. Notice how also non-matching events (in this example `unknown-user`) appear in output.

---

### Match Event Fields Against Patterns in Lookup Table Values

This example compares event fields with column values containing patterns in a lookup table using the `match()` function with glob pattern matching (`mode=glob`). This query matches event IDs against patterns in a user list and then adds the matching user details (other columns from the row in the `users.csv` file) to the events.

  * **Query Example**

    ```
    id = match(file="users.csv", column=userid, mode=glob, ignoreCase=true)
    ```

  * **Input Event Data**
    (Example incoming event data)

    | @timestamp | id | action | source\_ip |
    | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 |
    | 2024-01-15T09:01:00.000Z | dev-user-456 | code\_push | 10.0.0.2 |
    | 2024-01-15T09:02:00.000Z | TEST\_789 | test\_run | 10.0.0.3 |
    | 2024-01-15T09:03:00.000Z | support-001 | ticket\_update | 10.0.0.4 |
    | 2024-01-15T09:04:00.000Z | unknown-user | login\_attempt | 10.0.0.5 |

    (Example `users.csv` file data)

    | userid | department | access\_level | location | title |
    | :--- | :--- | :--- | :--- | :--- |
    | ADMIN-\* | IT | administrator | HQ | System Administrator |
    | dev-user-\* | Engineering | developer | Remote | Software Engineer |
    | TEST\_\* | QA | tester | Lab | QA Engineer |
    | support-\* | Support | agent | Office | Support Specialist |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `id match(file="users.csv", column=userid, mode=glob, ignoreCase=true)`: Uses glob pattern matching to match the `userid` column of the `users.csv` lookup table file against the IDs in the events. When an event ID matches a pattern in `users.csv`, all (non-pattern) columns from that first matching row are added to the event. The column names become new field names.
        Pattern matching examples based on the input data and lookup file are:
          * `ADMIN-123` matches `ADMIN-*`
          * `dev-user-456` matches `dev-user-*`
          * `TEST_789` matches `TEST_*`
          * `support-001` matches `support-*`
    3.  Event Result set.

  * **Summary and Results**
    The query is used to match event IDs against patterns in a user list, and then add the matching user details (other columns from the row in the `users.csv` file) to the events. Pattern-based matching with field enrichment is useful, for example, if you want to add user context to authentication logs.

  * **Result Event Data**

    | @timestamp | id | action | source\_ip | department | access\_level |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 | IT | administrator |
    | 2024-01-15T09:01:00.000Z | dev-user-456 | code\_push | 10.0.0.2 | Engineering | developer |
    | 2024-01-15T09:02:00.000Z | TEST\_789 | test\_run | 10.0.0.3 | QA | tester |
    | 2024-01-15T09:03:00.000Z | support-001 | ticket\_update | 10.0.0.4 | Support | agent |

    After matching, the output combines original event fields with matched user details. Only events with matching patterns appear in output.

# math:abs()

The `math:abs()` function calculates the absolute value of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `abs` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:abs()` function computes the absolute value of the numerical data in the specified input field. The result is outputted into a new field, which by default is named `abs` unless a different name is specified using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:abs()` in the document.

# math:arccos()

The `math:arccos()` function calculates the arc cosine of a field; the returned angle is in the range $0.0$ through $\\pi$.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `arccos` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:arccos()` function computes the arc cosine (inverse cosine) of the numerical values in the specified input field. The resulting angle is expressed in radians and will fall within the range of $0.0$ to $\\pi$. The computed arc cosine value is stored in a new output field. By default, this output field is named `arccos`, but a custom name can be assigned using the `as` parameter.

It's important to note that math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:arccos()` in the document.

# math:arcsin()

The `math:arcsin()` function calculates the arc sine of a field; the returned angle is in the range $-\\pi/2$ through $\\pi/2$.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `arcsin` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:arcsin()` function computes the arc sine (inverse sine) of the numerical values in the specified input field. The resulting angle is expressed in radians and will fall within the range of $-\\pi/2$ to $\\pi/2$. The computed arc sine value is stored in a new output field. By default, this output field is named `arcsin`, but a custom name can be assigned using the `as` parameter.

It's important to note that math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:arcsin()` in the document.

# math:arctan()

The `math:arctan()` function calculates the arc tangent of a value; the returned angle is in the range $-\\pi/2$ through $\\pi/2$.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `arctan` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:arctan()` function computes the arc tangent (inverse tangent) of the numerical values in the specified input field. The resulting angle is expressed in radians and will fall within the range of $-\\pi/2$ to $\\pi/2$. The computed arc tangent value is stored in a new output field. By default, this output field is named `arctan`, but a custom name can be assigned using the `as` parameter.

It's important to note that math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:arctan()` in the document.

# math:arctan2()

The `math:arctan2()` function calculates the angle in radians from the positive x-axis to the point $(x, y)$ in the plane.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_arctan2` | The name of the output field. |
| `X` | string | required | The x-coordinate. |
| `y` | string | required | The y-coordinate. |

***

## Function Operation

The `math:arctan2()` function computes the angle (in radians) from the positive x-axis to a point specified by its x and y coordinates. The `X` parameter represents the x-coordinate, and the `y` parameter represents the y-coordinate. The calculated angle is then outputted into a new field, which is named `_arctan2` by default but can be customized using the `as` parameter.

It is important to note that math functions performed on ARM architecture may yield slightly different results in very high-precision calculations when compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:arctan2()` in the document.

# math:ceil()

The `math:ceil()` function rounds the field value to the smallest (closest to negative infinity) double value that is greater than or equal to the field value and is equal to a mathematical integer.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_ceil` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:ceil()` function computes the ceiling of the numerical values in the specified input field. This means it rounds the value up to the next whole number. The result is outputted into a new field, which by default is named `_ceil` unless a different name is specified using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:ceil()` in the document.

# math:cos()

The `math:cos()` function calculates the cosine of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `COS` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:cos()` function computes the cosine of the numerical values present in the specified input field. The calculated cosine value is then stored in a new output field. By default, this output field is named `COS`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:cos()` in the document.

# math:cosh()

The `math:cosh()` function computes the hyperbolic cosine of a double field. The hyperbolic cosine of $x$ is defined to be $(e^x + e^{-x})/2$ where $e$ is Euler's number.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `cosh` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:cosh()` function calculates the hyperbolic cosine of the numerical values in the specified input field. The hyperbolic cosine of a number $x$ is mathematically defined as $(e^x + e^{-x})/2$, where $e$ is Euler's number (the base of the natural logarithm). The calculated hyperbolic cosine value is then stored in a new output field. By default, this output field is named `cosh`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:cosh()` in the document.

# math:deg2rad()

The `math:deg2rad()` function converts angles from degrees to radians.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_deg2rad` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:deg2rad()` function takes numerical values from the specified input field, which are assumed to be angles in degrees, and converts them into their equivalent radian measures. The calculated radian value is then stored in a new output field. By default, this output field is named `_deg2rad`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:deg2rad()` in the document.

# math:exp()

The `math:exp()` function calculates Euler's number $e$ raised to the power of a double value in a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `exp` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:exp()` function computes the value of Euler's number ($e$, approximately $2.71828$) raised to the power of the numerical value present in the specified input field. The result of this exponential calculation is then stored in a new output field. By default, this output field is named `exp`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:exp()` in the document.

# math:expm1()

The `math:expm1()` function calculates $e^x - 1$. For values of $x$ close to 0, the exact sum of `expm1(x) + 1` is much closer to the true result of $e^x$ than `exp(x)`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `expm1` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:expm1()` function computes the value of Euler's number ($e$) raised to the power of the numerical value in the specified input field, and then subtracts 1 from the result. This function is particularly useful when the value of $x$ (from the input field) is very close to 0, as `expm1(x) + 1` provides a more accurate approximation of $e^x$ than `exp(x)` in such cases. The calculated value is then stored in a new output field. By default, this output field is named `expm1`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:expm1()` in the document.

# math:floor()

The `math:floor()` function rounds the field value to the largest (closest to positive infinity) double value that is less than or equal to the field value and is equal to a mathematical integer.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_floor` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:floor()` function computes the floor of the numerical values in the specified input field. This means it rounds the value down to the nearest whole number. The result is outputted into a new field, which by default is named `_floor` unless a different name is specified using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:floor()` in the document.

# math:log()

The `math:log()` function calculates the natural logarithm (base $e$) of the value in a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_log` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log()` function computes the natural logarithm (logarithm to base $e$) of the numerical values in the specified input field. The calculated natural logarithm is then stored in a new output field. By default, this output field is named `_log`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log()` in the document.

# math:log1p()

The `math:log1p()` function calculates the natural logarithm of the sum of the field value and 1. For small values of x, the result of `log1p(x)` is much closer to the true result of $\\ln(1 + x)$ than the floating-point evaluation of `log(1.0 + x)`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `log1p` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log1p()` function computes the natural logarithm of $(1 + x)$, where $x$ is the numerical value from the specified input field. This function is particularly useful for maintaining precision when $x$ is a very small number, as it provides a more accurate result than directly calculating `log(1.0 + x)`. The calculated value is then stored in a new output field. By default, this output field is named `log1p`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log1p()` in the document.

# math:log2()

The `math:log2()` function calculates the base 2 logarithm of a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `log2` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log2()` function computes the base 2 logarithm of the numerical values present in the specified input field. The calculated base 2 logarithm is then stored in a new output field. By default, this output field is named `log2`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log2()` in the document.

# math:log10()

The `math:log10()` function calculates the base 10 logarithm of a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_log10` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log10()` function computes the base 10 logarithm of the numerical values present in the specified input field. The calculated base 10 logarithm is then stored in a new output field. By default, this output field is named `_log10`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log10()` in the document.

# math:mod()

The `math:mod()` function calculates the floor modulus of a field value and a divisor, for example, $x \\pmod y$ where $y$ is the divisor and is a field. Both the field or divisor are floor rounded before the calculation. Decimal values are supported.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `mod` | The name of the output field. |
| `divisor` | number | required | The divisor to use, for example, $x \\pmod{\\text{divisor}}$. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:mod()` function computes the floor modulus of a given field's value (dividend) and a specified divisor. This operation returns the remainder of the division, ensuring the result has the same sign as the divisor. Both the field value and the divisor are rounded down (floor-rounded) to the nearest integer before the calculation is performed. Decimal values are supported for both the dividend and the divisor. The calculated modulus is then stored in a new output field. By default, this output field is named `mod`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:mod()` in the document.

# math:pow()

The `math:pow()` function calculates the field value to the exponent power, that is, $\\text{field}^{\\text{exponent}}$. It supports decimal values.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_pow` | The name of the output field. |
| `exponent` | number | required | The exponent to use, for example, $\\text{exp}$ in $x^{\\text{exp}}$. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:pow()` function computes the result of raising the numerical value from the specified `field` to the power of the `exponent`. Both the base (from `field`) and the `exponent` can be decimal values. The calculated result is then stored in a new output field. By default, this output field is named `_pow`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:pow()` in the document.

# math:rad2deg()

The `math:rad2deg()` function converts angles from radians to degrees.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_rad2deg` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:rad2deg()` function takes numerical values from the specified input field, which are assumed to be angles in radians, and converts them into their equivalent degree measures. The calculated degree value is then stored in a new output field. By default, this output field is named `_rad2deg`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:rad2deg()` in the document.

# math:sin()

The `math:sin()` function calculates the sine of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `sin` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:sin()` function computes the sine of the numerical values present in the specified input field. The calculated sine value is then stored in a new output field. By default, this output field is named `sin`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:sin()` in the document.

# math:sinh()

The `math:sinh()` function calculates the hyperbolic sine of a double field. The hyperbolic sine of $x$ is defined to be $(e^x - e^{-x})/2$ where $e$ is Euler's number.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `sinh` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:sinh()` function computes the hyperbolic sine of the numerical values in the specified input field. The hyperbolic sine of a number $x$ is mathematically defined as $(e^x - e^{-x})/2$, where $e$ is Euler's number (the base of the natural logarithm). The calculated hyperbolic sine value is then stored in a new output field. By default, this output field is named `sinh`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:sinh()` in the document.

# math:sqrt()

The `math:sqrt()` function calculates the rounded positive square root of a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_sqrt` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:sqrt()` function computes the positive square root of the numerical values present in the specified input field. The calculated square root is then rounded and stored in a new output field. By default, this output field is named `_sqrt`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:sqrt()` in the document.

# math:tan()

The `math:tan()` function calculates the trigonometric tangent of an angle in a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `tan` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:tan()` function computes the trigonometric tangent of the numerical angle values present in the specified input field. The calculated tangent value is then stored in a new output field. By default, this output field is named `tan`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:tan()` in the document.

# math:tanh()

The `math:tanh()` function calculates the hyperbolic tangent of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `tanh` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:tanh()` function computes the hyperbolic tangent of the numerical values present in the specified input field. The calculated hyperbolic tangent value is then stored in a new output field. By default, this output field is named `tanh`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:tanh()` in the document.

# max()

The `max()` function finds the largest number for the specified field over a set of events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `max` | Name of output field. |
| `field` | string | required | Field to extract a number from. |
| `type` | string | optional | description |

***

## Function Operation

The `max()` function is an aggregate function used to determine the highest numerical value present in a specified field across a collection of events. When used, it returns a single event containing this maximum value in an output field. By default, this output field is named `max`, but you can assign a custom name using the `as` parameter. The `field` parameter, which can be omitted if the function is used in a context where the input field is clear, indicates which field's values should be evaluated.

***

## Examples

### Return what was the maximum responsetime

This is a basic example of using `max()` to find the highest response time.

  * **Query Example**
    ```
    max(responsetime)
    ```

### Filter for events in the repository with maximum responsetime values greater than 5 seconds

This example demonstrates how to filter events based on the maximum response time.

  * **Query Example**
    ```
    max(responsetime)
    _max > 5
    ```

### Alert Query for Parser Issues

This alert query tries to balance reacting when there are problems with parsers without being too restrictive.

  * **Query Example**

    ```
    #type=humio #kind=logs
    | loglevel=WARN
    | class = c.h.d.ParserLimitingJob
    | "Setting reject ingest for"
    groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])
    | timeDiff:=_max - _min
    timeDiff > 300000 and count > 10
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=humio #kind=logs`: Filters on all logs across all hosts in the cluster.
    3.  `| loglevel=WARN`: Filters for all events where the `loglevel` is equal to `WARN`.
    4.  `| class = c.h.d.ParserLimitingJob`: Assigns the value `c.h.d.ParserLimitingJob` to the `class` for the logs having the `loglevel` value `WARN`.
    5.  `| "Setting reject ingest for"`: Filters for events containing the string "Setting reject ingest for". This is the error message generated when ingested events are rejected.
    6.  `groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])`: Groups the returned result by the field `id`, makes a count on the events, and returns the minimum timestamp and maximum timestamp. This returns a new event set, with the fields `id`, `_count`, `_min`, and `_max`.
    7.  `| timeDiff:=_max - _min`: Calculates the time difference between the maximum timestamp values (`_max`) and the minimum timestamp values (`_min`) and returns the result in a new field named `timeDiff`.
    8.  `| timeDiff > 300000 and count > 10`: Returns all events where the value of `timeDiff` is greater than 300000 and where there are more than 10 occurrences.
    9.  Event Result set.

  * **Summary and Results**
    This query is used to set up alerts for parser issues. Setting up alerts for parser issues will allow to proactively reach out to customers where their queries are being throttled and help them.

---

### Calculate Minimum and Maximum Response Times

This example calculates minimum and maximum response times using multiple aggregate functions in square brackets.

  * **Query Example**

    ```
    [min_response:= min(responsetime), max_response:= max(responsetime)]
    ```

  * **Input Event Data**

    | @timestamp | endpoint | responsetime | status\_code |
    | :--- | :--- | :--- | :--- |
    | 1686837825000 | /api/users | 145 | 200 |
    | 1686837826000 | /api/products | 892 | 200 |
    | 1686837827000 | /api/orders | 167 | 200 |
    | 1686837828000 | /api/payment | 1290 | 500 |
    | 1686837829000 | /api/users | 156 | 200 |
    | 1686837830000 | /api/items | 78 | 200 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `[min_response:= min(responsetime), max_response = max(responsetime)]`: In a single operation, calculates the minimum value from the `responsetime` field and returns the results in a field named `min_response`, and calculates the maximum value from the `responsetime` field and returns the results in a field named `max_response`. Square brackets allow multiple aggregations to be performed in a single operation.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the range of response times by calculating both the minimum and maximum values. The results are returned in fields with names specified in the field assignments. This query is useful, for example, to monitor service performance, identify outliers in response times, or establish performance baselines.

  * **Result Event Data**

    | min\_response | max\_response |
    | :--- | :--- |
    | 78 | 1290 |

    Note that only one row is returned containing both calculated values.

---

### Compute Aggregate Value for Each Array Element With Same Index

This example computes an aggregate value for each array element with the same index using the `array::reduceColumn()` function.

  * **Query Example**

    ```
    maxTimes := array::reduceColumn(times, var=x, function={time := max(x)})
    ```

  * **Input Event Data**

    | times\[0] | times\[1] | times\[2] |
    | :--- | :--- | :--- |
    | 1 | 2 | 3 |
    | 5 | 1 | 0 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `maxTimes := array::reduceColumn(times, var=x, function={time := max(x)})`: Computes the maximum time for each array element with the same index in the array and reduces it to one value.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the maximum time for each array element with the same index in a flat array.

  * **Result Event Data**

    | \_reduceColumn\[0] | \_reduceColumn\[1] | \_reduceColumn\[2] |
    | :--- | :--- | :--- |
    | 5 | 2 | 3 |

---

### Compute an Aggregated Value of an Array on All Events

This example computes an aggregated value of a flat array on all events using the `array::reduceAll()` function.

  * **Query Example**

    ```
    array::reduceAll("values[]", var=x, function=max(x))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `array::reduceAll("values[]", var=x, function=max(x))`: Computes the maximum value over all the values within the array `values[]` by using `max()` on each element, and then across each event in the event set.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to compute a value from all events and array elements of a specified array. The `reduce()` method is recommended when you need to have a single value returned from iterating over your array. Only aggregate functions that return a single event with a single field (such as `avg()`, `count()`, `sum()`, `max()` etc.) are allowed as the function argument.

---

### Create a Pivot Table

This example creates a view of LogScale activity by creating a pivot table.

  * **Query Example**

    ```
    groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})
    transpose(header=type)
    | drop(column)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})`: The first step to creating a pivot table is the base query that will create the initial summary of the information. In this fragment, a nested `groupBy()` aggregation is used. The embedded aggregation creates a group of the maximum access time for a given user, by using `max()` on the `@timestamp` against the `actor.user.id`. This creates a table of the last event by the user. The outer `groupBy()` then creates an aggregation of this maximum user time against the `type` which defines the operation performed. The result is a table of the last user and time for a specific operation.
    3.  `transpose(header=type)`: The `transpose()` function will convert individual columns into rows, switching the orientation. For example, the `type` column will now become the `type` row. The `header` parameter is used to specify `type` as the column header.
    4.  `drop(column)`: The table created contains the summarized information pivoted around the user ID and last event time further summarized by the type of the event. However, there is a `column` field in the event stream that was generated from the old row before the table was pivoted. That column can be removed by dropping the `column` field from the events using `drop()`.
    5.  Event Result set.

  * **Summary and Results**
    Pivoting an event set of data allows for the information to be displayed and summarized in a format that may make more logical sense as a display format. The query allows for creation of a pivot table showing the last user and time for a specific operation.

  * **Result Event Data (after first groupBy)**

    | type | actor.user.id | \_max |
    | :--- | :--- | :--- |
    | alert.clear-error | 007WGPBX9YbvZbK0rBMd5fgH | 1700546666592 |
    | alert.create | 007WGPBX9YbvZbK0rBMd5fgH | 1699004139419 |
    | alert.update | 007WGPBX9YbvZbK0rBMd5fgH | 1700546666676 |
    | dashboard.create | 007WGPBX9YbvZbK0rBMd5fgH | 1698417330709 |
    | dataspace.query | 007WGPBX9YbvZbK0rBMd5fgH | 1700721296197 |

  * **Result Event Data (after transpose)**

    | alert.clear-error | alert.create | alert.update |
    | :--- | :--- | :--- |
    | 1700546666592 | 1699004139419 | 1700546666676 |
    | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKO |

  * **Result Event Data (final after drop)**

    | alert.clear-error | alert.create | alert.update |
    | :--- | :--- | :--- |
    | 1700546666592 | 1699004139419 | 1700546666676 |
    | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd\! |

---

### Rounding Within a Timechart

This example rounds down a number in a field and displays information in a timechart using the `round()` and `timeChart()` functions.

  * **Query Example**

    ```
    timeChart(function={max(value) | round(_max, how=floor)})
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `timeChart(function={max(value) | round(_max, how=floor)})`: Creates a time chart using `max()` as the aggregate function for a field named `value` to find the highest value in each time bucket, and returns the result in a field named `_max`. It then rounds the implied field `_max` from the aggregate `max()` using the `floor` option to round down the value. For example, if original `_max` values were 10.8, 15.3, 20.7, after floor rounding they would be 10, 15, 20.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to round down maximum values over time to the nearest integer (whole value). This is useful when displaying information in a time chart. Rounding to the nearest integer will make it easier to distinguish the differences between values when used on a graph for time-based visualization. The query simplifies the data presentation. To round to a specific decimal accuracy, the `format()` function must be used.

# min()

The `min()` function finds the smallest number for the specified field over a set of events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `min` | Name of output field. |
| `field` | string | required | Field to extract a number from. |
| `type` | string | optional | description |

***

## Function Operation

The `min()` function is an aggregate function used to determine the lowest numerical value present in a specified field across a collection of events. When used, it returns a single event containing this minimum value in an output field. By default, this output field is named `min`, but you can assign a custom name using the `as` parameter. The `field` parameter indicates which field's values should be evaluated.

***

## Examples

### Return what was the minimum responsetime

This is a basic example of using `min()` to find the lowest response time.

  * **Query Example**
    ```
    min(responsetime)
    ```

### Filter for events with a responsetime greater than 5 seconds

This example demonstrates how to filter events based on the minimum response time.

  * **Query Example**
    ```
    min(responsetime)
    _min > 5
    ```

---

### Alert Query for Parsers Issues

This alert query tries to balance reacting when there are problems with parsers, without being too restrictive.

  * **Query Example**

    ```
    #type=humio #kind=logs
    | loglevel=WARN
    | class = c.h.d.ParserLimitingJob
    | "Setting reject ingest for"
    groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])
    | timeDiff:=_max - _min
    | timeDiff > 300000 and count > 10
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=humio #kind=logs`: Filters on all logs across all hosts in the cluster.
    3.  `| loglevel=WARN`: Filters for all events where the `loglevel` is equal to `WARN`.
    4.  `| class = c.h.d.ParserLimitingJob`: Assigns the value `c.h.d.ParserLimitingJob` to the class for the logs having the `loglevel` value `WARN`.
    5.  `| "Setting reject ingest for"`: Filters for events containing the string "Setting reject ingest for". This is the error message generated when ingested events are rejected.
    6.  `groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])`: Groups the returned result by the field `id`, makes a count on the events, and returns the minimum timestamp and maximum timestamp. This returns a new event set, with the fields `id`, `_count`, `_min`, and `_max`.
    7.  `| timeDiff:=_max - _min`: Calculates the time difference between the maximum timestamp values and the minimum timestamp values and returns the result in a new field named `timeDiff`.
    8.  `| timeDiff > 300000 and count > 10`: Returns all events where the values of `timeDiff` are greater than 300000 and where there are more than 10 occurrences.
    9.  Event Result set.

  * **Summary and Results**
    This query is used to set up alerts for parser issues. Setting up alerts for parser issues will allow proactively reaching out to customers where their queries are being throttled and help them.

---

### Calculate Minimum and Maximum Response Times

This example calculates minimum and maximum response times using multiple aggregate functions in square brackets.

  * **Query Example**

    ```
    [min_response:= min(responsetime), max_response := max(responsetime)]
    ```

  * **Input Event Data**

    | @timestamp | endpoint | responsetime | status\_code |
    | :--- | :--- | :--- | :--- |
    | 1686837825000 | /api/users | 145 | 200 |
    | 1686837826000 | /api/products | 892 | 200 |
    | 1686837827000 | /api/orders | 167 | 200 |
    | 1686837828000 | /api/payment | 1290 | 500 |
    | 1686837829000 | /api/users | 156 | 200 |
    | 1686837830000 | /api/items | 78 | 200 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `[min_response:= min(responsetime), max_response = max(responsetime)]`: In a single operation, calculates the minimum value from the `responsetime` field and returns the results in a field named `min_response`, and calculates the maximum value from the `responsetime` field and returns the results in a field named `max_response`. Square brackets allow multiple aggregations to be performed in a single operation.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the range of response times by calculating both the minimum and maximum values. The results are returned in fields with names specified in the field assignments. This query is useful, for example, to monitor service performance, identify outliers in response times, or establish performance baselines.

  * **Result Event Data**

    | min\_response | max\_response |
    | :--- | :--- |
    | 78 | 1290 |

    Note that only one row is returned containing both calculated values.

# now()

The `now()` function assigns the current time to the field provided by parameter `as`. The time is represented as milliseconds since January 1, 1970 (UTC). In historical queries, the current time is when the query is issued. In live queries, the current time is when `now()` is processed for each event. The value, therefore, depends on where in the query `now()` is placed. If it is placed before the first aggregate function, it is only evaluated the first time the query sees the event. If it is placed after the first aggregate function, it is evaluated continuously, and gives the live value of the current system time, which can divert between LogScale nodes.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `now` | Name of output field. |

***

## Function Operation

The `now()` function captures the current time and assigns it to a specified field. This time is expressed as milliseconds since January 1, 1970 (UTC). The behavior of `now()` differs between historical and live queries. In historical queries, it reflects the time the query was initiated. In live queries, it captures the time each event is processed by the `now()` function itself, meaning its value can vary depending on its position within the query pipeline and can even differ across LogScale nodes if placed after the first aggregate function. If `now()` is placed before the first aggregate function, it is only evaluated once per event.

***

## Examples

### Assign Current Time of Search Time Interval to Field

This example uses the `now()` function in a comparison to determine if an event is "old" (more than 1000 milliseconds old) by comparing the difference between the end time of the search interval and the event's `@timestamp` field.

  * **Query Example**

    ```
    $isold:=(now()-@timestamp)>1000$
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `isold:=(now()-@timestamp)>1000`: Returns results where the current time minus the ingestion timestamp are greater than 1000, and assigns the returned results to a new field named `isold`. More specifically, `now()` returns the end time of the search interval, `@timestamp` is the timestamp of the individual event. `(now()-@timestamp)` calculates the difference between these two times, giving the age of the event relative to the end of the search interval. This difference is compared to 1000 milliseconds (1 second). If the difference is greater than 1000 milliseconds, `isold` will be set to `true`; otherwise, it will be `false`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to quickly identify events that are more than 1 second older than the end of the search interval. Assigning the current time of the search time interval to a field is useful when, for example, searching for security events. When running a query, the events you are searching do not know the span of the search (the system just returns a list of the events in the given time interval), but you might want to show the relative time of the event timestamp compared to the search window. For example, if you search for a bunch of events that happened yesterday and you are searching from yesterday 00:00 to 23:59, you then want to calculate '3 hours before' or even '2s before' because when searching for security events that time difference may be important.

---

### Process Current Time in Live Queries

This example uses the `now()` function in live queries to process the current time for each event.

  * **Query Example**

    ```
    $curr:=now()$
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `$curr:=now()`: Processes current time for each event and returns the timestamps in a field named `curr` field. It records when events occur. The timestamp represents milliseconds.
    3.  Event Result set.

  * **Summary and Results**
    The query is used in live queries to process the current time for each event. Note that in live queries, this query returns timestamps based on the `now()` function's location:

      * Before the first aggregate function: Returns the initial event processing timestamp (it is only evaluated the first time the query sees the event).
      * After the first aggregate function: Returns continuously updated timestamps from each LogScale node (it gives the live value of the current system time, which can divert between LogScale nodes when `now()` is placed after the first aggregate function).
        The `now()` function and capturing current timestamps is useful in security contexts for incident response timing, threat detection timestamps, and security event correlation.

# default()

The `default()` function sets a default value for one or more fields. If a specified field does not exist in an event, this function creates it with the given value. If the field already exists, its original value is kept.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | string or array | required | The field or fields to set the default value for. |
| **replaceEmpty** | boolean | optional <br> default: `false` | If set to `true`, the function will also override fields that exist but have an empty string as their value. |
| **value** | string | required | The default value to assign to the field(s). The parameter name `value` can be omitted. |

***

## Example

### Set a Default Value and Replace Empty Fields

This example shows how to ensure a `message` field always has a value by setting a default. It uses the `replaceEmpty` parameter to handle cases where the field exists but is an empty string, which is important for data normalization.

* **Query Example**
    ```
    default(field=message, value="N/A", replaceEmpty=true)
    ```

* **Step-by-Step**
    1.  The query starts with a set of events.
    2.  For each event, the `default()` function checks the `message` field.
    3.  If the `message` field does not exist, it is created with the value `"N/A"`.
    4.  Because `replaceEmpty=true` is set, if the `message` field exists but its value is an empty string (`""`), its value is also changed to `"N/A"`.
    5.  If the `message` field exists and has a non-empty value, it remains unchanged.

* **Summary and Results**
    This query is crucial for data preparation, especially before using functions like `eval()` that require fields to be present. It ensures that every event will have a consistent, non-empty `message` field, which can be critical for analysis and quick triage in security or operational logs.

# drop()

The `drop()` function is a filter that removes specified fields or columns from a result set.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **fields** | array of strings | required | The name of the field or a list of field names to be discarded. The parameter name `fields` can be omitted. |

***

## Example

### Calculate a Percentage and Drop Intermediate Fields

This advanced example shows `drop()` being used to remove temporary fields that were created for an intermediate calculation, ensuring the final output is clean. The query calculates the percentage of successful HTTP status codes over time.

* **Query Example**
    ```
    success := if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
          [sum(success, as=success), count(as=total)]
          | pct_successful := (success / total) * 100
          | drop([success, total])
        }
      ], span=15m, limit=100)
    ```

* **Step-by-Step**
    1.  A temporary field `success` is created, assigned `1` for successful status codes and `0` for server errors.
    2.  `timeChart()` begins creating a time series chart.
    3.  Inside an embedded aggregation `{...}`:
        * The `sum()` of the `success` field and a `count()` of all events are calculated, creating two new temporary fields: `success` and `total`.
        * These fields are used to calculate the `pct_successful`.
        * `drop([success, total])` is then used to remove the temporary `success` and `total` fields, as they are no longer needed in the final result.

* **Summary and Results**
    The query generates a time chart that shows only the final `pct_successful` value for each customer. The `drop()` function is essential here for cleaning up the intermediate data used in the calculation, making the final output focused and relevant.

# dropEvent()

The `dropEvent()` function is used to discard events. It has different behaviors depending on whether it's used during the parsing/ingestion phase or in a normal search query.

***

## Parameters

The `dropEvent()` function does not accept any parameters.

***

## Function Operation

The behavior of `dropEvent()` changes based on its context:
* **During Parsing**: When used in a parser, the function permanently drops the event. The event data will not be stored in Falcon LogScale.
* **During a Search**: When used in a normal search query, the function acts as an alias for `false`. It filters the event from the search results but does not delete it from storage.

***

## Examples

### Drop an Event During Parsing

This example shows how to use `dropEvent()` within a parser to prevent events that match a specific condition from being ingested and stored.

* **Query Example**
    ```
    parseJson()
    | case { 
        someField = "some_value" | dropEvent(); 
        * }
    | parseTimestamp(field=@timestamp)
    ```

* **Step-by-Step**
    1.  `parseJson()`: The query first parses incoming JSON data to extract fields.
    2.  `case`: A `case` statement checks the value of `someField`.
    3.  If `someField` equals `"some_value"`, the `dropEvent()` function is called, and the parsing for that event is terminated, preventing it from being stored.
    4.  All other events (matched by `*`) are passed on to the next step, `parseTimestamp()`.

* **Summary and Results**
    This query is used to filter out unwanted data at the point of ingestion. By dropping events with a specific field value, you can reduce storage costs and noise in your repository.

---

### Drop Events Based on Multiple Conditions in a Search

This more complex example demonstrates using `dropEvent()` in a regular search query to filter results based on several different conditions.

* **Query Example**
    ```
    case {
      fielda = "badresult" | dropEvent();
      fieldb = "badresult" | dropEvent();
      wildcard("badip*", field=[fieldc, fieldd]) | dropEvent();
      *
    }
    ```

* **Step-by-Step**
    1.  The query uses a `case` statement to evaluate multiple conditions on the events in the search results.
    2.  The first two conditions check if `fielda` or `fieldb` have the exact value `"badresult"`. If either is true, the event is dropped from the results.
    3.  The third condition uses the `wildcard()` function to check if either `fieldc` or `fieldd` contains a value starting with `"badip"`. If so, the event is dropped.
    4.  The final `*` ensures that any events not matching the drop conditions are kept in the results.

* **Summary and Results**
    This query is used to filter out events from a search result based on multiple, specific criteria. In this context, `dropEvent()` functions as a simple filter (`false`), removing unwanted events from the final view to help narrow down an investigation.

# duration()

The `duration()` function computes the number of milliseconds in a given time period. It makes timestamp comparisons easier to read and less error-prone by allowing the use of human-readable time syntax (e.g., `5m`, `2d`).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_duration` | The name of the output field that will store the millisecond value. |
| **duration**| string | required | The time duration specification, defined using Relative Time Syntax. The parameter name `duration` can be omitted. |

***

## Function Operation

This function takes a string representing a time duration (like `"5m"` for five minutes) and returns the equivalent number of milliseconds. This numeric value can then be used in calculations and comparisons, which is especially useful for creating dynamic dashboard parameters.

***

## Example

### Narrow the Search Interval

This example demonstrates how to use `duration()` to filter events, keeping only those that have occurred within the last two days.

* **Query Example**
    ```
    test(@timestamp > now() - duration("2d"))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  The `duration("2d")` function is called, which calculates the total number of milliseconds in two days.
    3.  This millisecond value is subtracted from the current time (`now()`) to establish a cutoff timestamp for two days ago.
    4.  The `test()` function then compares each event's `@timestamp` to this cutoff, keeping only the events that are more recent.

* **Summary and Results**
    This query effectively filters the search results to a rolling two-day window. Using `duration()` makes the query more readable and maintainable than hard-coding the millisecond value for two days (`172800000`). It is a convenient way to define relative time windows in searches and dashboards.

# end()

[cite_start]The `end()` function assigns a timestamp to an output field, representing the end of the search time interval in milliseconds since the UTC epoch. [cite: 646, 647]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_end` | [cite_start]The name of the output field. [cite: 654] [cite_start]The parameter name can be omitted. [cite: 655] |

***

## Function Operation

[cite_start]This function provides the timestamp for the end of the current search window. [cite: 647] [cite_start]For **live queries**, `end()` is equivalent to `now()`, returning the current time. [cite: 648] [cite_start]For **subqueries**, it returns the end time of that specific subquery's search interval. [cite: 649] [cite_start]It's important to note that `end()` is **not compatible** with parsers because they do not operate within a search interval. [cite: 651]

***

## Example

### Calculate Event Age Relative to the End of the Search

This example uses `end()` to calculate the age of an event relative to the end of the search window, creating a new boolean field `isold` if the event is older than one second.

* **Query Example**
    ```
    isold := (end() - @timestamp) > 1000
    ```

* **Step-by-Step**
    1.  [cite_start]The query starts with the source repository events. [cite: 687]
    2.  [cite_start]`end()` returns the timestamp for the end of the search interval. [cite: 691]
    3.  [cite_start]`@timestamp` is the timestamp of the individual event. [cite: 691]
    4.  [cite_start]The difference `(end() - @timestamp)` calculates the age of the event relative to the end of the search. [cite: 692]
    5.  [cite_start]This difference is compared to `1000` milliseconds (1 second). [cite: 693] [cite_start]If the age is greater than 1000, the new field `isold` is set to `true`; otherwise, it's `false`. [cite: 693, 694]

* **Summary and Results**
    [cite_start]This query is useful for quickly identifying "old" events within a given time frame. [cite: 697] [cite_start]For example, in a security investigation, knowing if an event occurred at the very beginning or end of a search window can be important for establishing a timeline. [cite: 700]

# eval()

[cite_start]The `eval()` function creates or modifies a field by evaluating a given expression. [cite: 3] [cite_start]The result of the expression is stored in a new or existing field. [cite: 3, 4]

***

## Parameters

[cite_start]This function does not take any formal parameters. [cite: 9] [cite_start]Instead, its arguments consist of one or more comma-separated assignment expressions (e.g., `eval(field1 = expression1, field2 = expression2)`). [cite: 7]

***

## Function Operation

[cite_start]The `eval()` function processes expressions from left to right. [cite: 21] [cite_start]It supports basic arithmetic (`+`, `-`, `*`, `/`) and comparison (`==`, `!=`) operators. [cite: 6] [cite_start]An assignment only occurs if the expression can be successfully evaluated. [cite: 25] [cite_start]If a field used in the expression is missing from an event or is not a number, the evaluation fails, and no new field is created. [cite: 26, 27] [cite_start]If the target field already existed, its value will remain unchanged in case of a failed evaluation. [cite: 28]

***

## Example

### Match Field to Timespan

This example demonstrates how to use `eval()` after a `timeChart()` function to normalize aggregated data. It converts a 5-minute event count into an average per-minute count.

* **Query Example**
    ```
    timechart(method, span=5min)
    | eval(_count = _count / 5)
    ```

* **Step-by-Step**
    1.  [cite_start]`timechart(method, span=5min)`: This function first groups events into 5-minute buckets. [cite: 60] [cite_start]By default, it counts the events in each bucket and outputs the result to a field named `_count`. [cite: 61]
    2.  `eval(_count = _count / 5)`: The results from the time chart are then piped to `eval()`. [cite_start]This expression takes the `_count` value for each 5-minute bucket, divides it by 5, and then overwrites the `_count` field with this new per-minute average. [cite: 64]

* **Summary and Results**
    [cite_start]This query is useful for displaying per-minute rates while still benefiting from the improved performance of aggregating data into larger time buckets. [cite: 65] [cite_start]The final chart will show a value that represents the average count per minute, even though the data is bucketed in 5-minute intervals. [cite: 64, 70]

# eventFieldCount()

[cite_start]The `eventFieldCount()` function computes the number of internal fields an event contains and adds this count to the event in a new field[cite: 3176]. [cite_start]It's important to note that tags are generally not counted as fields[cite: 3176].

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_eventFieldCount` | [cite_start]The name of the output field that will store the field count[cite: 3177]. |

***

## Example

### Search for Events by Number of Fields

[cite_start]This example demonstrates how to find events that have a low field count, which can be useful for identifying sparsely populated or potentially incomplete events[cite: 3182, 3204].

* **Query Example**
    ```
    eventFieldCount()
    | _eventFieldCount < 6
    ```
    [cite_start][cite: 3185, 3186]

* **Step-by-Step**
    1.  `eventFieldCount()`: This function is first called to go through the events. [cite_start]It calculates the number of fields for each event and adds a new field, `_eventFieldCount`, containing this number[cite: 3195].
    2.  `| [cite_start]_eventFieldCount < 6`: The events, now augmented with the field count, are then piped to a filter[cite: 3189]. [cite_start]This filter keeps only the events where the value of `_eventFieldCount` is less than 6[cite: 3200].

* **Summary and Results**
    [cite_start]The query is used to get an overview of events with a low field count[cite: 3204]. [cite_start]The final result set will only include events that contain fewer than six fields[cite: 3200].

# eventInternals()

The `eventInternals()` function adds a set of fields to an event that describe its storage locations, such as the datasource ID.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **prefix** | string | optional <br> default: `@` | The prefix for the new fields that are created. |

***

## Example

### Count Events From Each Datasource

This example uses `eventInternals()` to add storage location information to events, and then uses that information to group and count the number of events coming from each datasource.

* **Query Example**
    ```
    eventInternals()
    | groupBy([@datasourceId])
    ```

* **Step-by-Step**
    1.  `eventInternals()`: This function first processes the events and adds fields describing their storage location. In this case, it adds the `@datasourceId` field to each event.
    2.  `groupBy([@datasourceId])`: The events, now enriched with the datasource ID, are then grouped by this new field. By default, `groupBy` will also count the number of events in each group and place the result in a `_count` field.

* **Summary and Results**
    The query is used to find the number of events per datasource. The final output is a table listing each unique datasource ID and the total count of events that originated from it.

# eventSize()

The `eventSize()` function determines the number of bytes an event's values use internally for disk storage. It does not count the bytes used for storing field names.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_eventSize` | The name of the output field that will store the event size. |

***

## Function Operation

This function adds a field to each event containing its on-disk storage size in bytes. It's important to note that this does not reflect the RAM usage of an event during a query, and as a result, events generated by aggregations will have a size of zero.

***

## Example

### Track Event Size Statistics Over Time

This example shows how to calculate the size of each event and then report statistical information about those sizes (max, percentiles) in a time chart. This can be useful for monitoring trends in event size.

* **Query Example**
    ```
    eventSize(as=eventSize)
    | timeChart(function=[max(eventSize), percentile(field=eventSize, percentiles=[50,75,90,99])])
    ```

* **Step-by-Step**
    1.  `eventSize(as=eventSize)`: This first step calculates the size of each individual event and stores it in a new field named `eventSize`.
    2.  `timeChart(...)`: The events, now containing their size, are piped into `timeChart()`. This function calculates two sets of statistics for each time bucket:
        * `max(eventSize)`: Finds the largest event size within that bucket.
        * `percentile(...)`: Calculates the 50th, 75th, 90th, and 99th percentiles for the event sizes in that bucket.

* **Summary and Results**
    The query generates a graph showing the relative sizes of events over time. This allows you to visualize trends, such as the maximum event size and the distribution of event sizes (e.g., 99% of events are smaller than X bytes), which can help identify performance issues related to large events.

# fieldset()

The `fieldset()` function retrieves and displays a list of all available fields within the current event set.

***

## Parameters

The `fieldset()` function does not take any parameters.

***

## Function Operation

This function is useful for quickly getting an overview of the data schema, especially when you are new to a dataset. It returns a list of all field names present in the events at that stage of the query. The list is context-specific; if you run `fieldset()` after an aggregation function like `groupBy()`, it will show the fields present in the aggregated results, not the original events.

***

## Example

### Request a List of Fields After Aggregation

This example demonstrates how the output of `fieldset()` changes when used after an aggregation. It first groups events and then lists the fields available in the aggregated result.

* **Query Example**
    ```
    groupBy([#type, @host])
    | fieldset()
    ```

* **Step-by-Step**
    1.  `groupBy([#type, @host])`: This function first groups all events by the `#type` and `@host` fields. By default, it also creates a `_count` field that contains the number of events in each group. The original fields are discarded, and only `#type`, `@host`, and `_count` remain.
    2.  `fieldset()`: This function is then called on the aggregated results. It lists the fields that are now present: `_count`, `@host`, and `#type`.

* **Result Event Data**
    The output is a list of the fields that exist after the `groupBy` operation, not the full list of fields from the original events.
    ```
    _count
    @host
    #type
    ```

# fieldstats()

The `fieldstats()` function displays statistics about the fields within the current event stream, such as the total count and the number of distinct values for each field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **limit** | integer | optional <br> default: 200 | Limits the number of fields to generate statistics for (min 1, max 10,000). |

***

## Function Operation

This function returns a new set of events where each event contains statistics for a field found in the original input stream. The new events will have the following fields:
* **#type**: The name of the parser used when the data was ingested.
* **field**: The name of the field these statistics are for.
* **_count**: The total number of times the field appears in the incoming event set.
* **_distinct**: The number of unique values for that field in the incoming event set.

***

## Example

### Extract Field Statistics

This example uses `fieldstats()` to get a statistical overview of all fields in the current repository.

* **Query Example**
    ```
    fieldstats()
    ```

* **Step-by-Step**
    1.  The query starts with all events in the current time range.
    2.  `fieldstats()` is called, which processes the stream and generates a new event for each field, containing its statistics.

* **Result Event Data**
    The output is a table where each row provides statistics for a single field from the original data.

| #type | field | _count | _distinct |
| :--- | :--- | :--- | :--- |
| humio | decodedContentLength | 60501 | 107 |
| humio | originalName | 154824 | 19 |
| humio | tx_fifo | 16296 | 1 |
| humio | humioClass | 1641588 | 320 |
| humio | p75 | 268658 | 42439 |
| humio | bucketId | 234418 | 2 |
| ... | ... | ... | ... |

# findTimestamp()

The `findTimestamp()` function automatically finds and parses the first timestamp it can identify within a given field by trying a variety of common formats. It is primarily intended for use in generic parsers where the exact timestamp format may not be known in advance.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **addErrors** | boolean | optional <br> default: `true` | If `true`, adds an error field to the event if a timestamp cannot be found. |
| **as** | string | optional <br> default: `@timestamp` | The output field for the parsed timestamp, represented as milliseconds since the Unix epoch. |
| **field** | string | optional <br> default: `@rawstring` | The field to search for a timestamp in. |
| **timezone** | string | optional | Specifies the timezone (e.g., `UTC`, `America/New_York`) to use if the timestamp itself does not contain one. This is required for timestamps without timezone information. |
| **timezoneAs**| string | optional <br> default: `@timezone` | The output field for the parsed timezone. |

***

## Function Operation

The function searches for a timestamp only within the first 128 characters of the specified field. If the exact format of your timestamps is known, it is better to use the `parseTimestamp()` function.

### Supported Formats
The function attempts to match against several formats, including:
* `year month day hour minute second`
* `month day [year] hour minute second`
* `epochsecond [subsecond]`
* And others, with optional subseconds and timezones.

### Key Behaviors
* **Missing Timezone**: If a timestamp does not include a timezone, the `timezone` parameter must be provided, otherwise the timestamp will not be parsed.
* **Missing Date**: If the date is missing, the current date is used.
* **Missing Year**: If the year is missing, the system infers the year to be the most recent one that keeps the date within 7 days of the present.
* **Two-Digit Years**: Two-digit years are assumed to be between 2013 and 2099.

> **Note**
> For timestamps in a time zone with Daylight Saving Time, it is recommended to specify the `timezone` parameter as an offset (e.g., `UTC+02:00`) to avoid ambiguity during the switch between DST and Standard Time.

***

## Examples

The document provides several syntax examples demonstrating different parameter combinations.

### Set a Default Timezone

This is used in a parser when the incoming timestamps are known to be from a specific timezone but do not include that information in the string itself.

# format()

The `format()` function creates formatted strings using printf-style specifiers and adds the result to a new field. It can format a single field or an array of fields.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_format` | The name of the output field for the formatted string. |
| **field** | array of strings | required | The field or fields to format. |
| **format**| string | required | The formatting codes (e.g., `"%s, %.2f"`) for the given fields. The parameter name `format` can be omitted. |
| **timezone**| string | optional | The timezone to use when formatting date and time values. |

***

## Function Operation

The `format()` function uses a format string containing specifiers that define how to represent the values from the `field` parameter. The general syntax for a specifier is `%[flags][width][.precision]type`.

### Supported Types
* **Numbers**: Use `%d` or `%i` for integers, `%f` for floating-point decimals, and `%X` for hexadecimal.
* **Strings**: Use `%s` to include a string value.
* **Date/Time**: Use `%t` followed by a modifier to format timestamps (which must be in milliseconds since epoch). For example, `%tH` extracts the hour, and `%tF` formats the date as `YYYY-MM-DD`.
* **Booleans**: Use `%b` for `true`/`false`.

### Key Flags
* **`,`**: Adds thousands separators to numbers.
* **`.<number>`**: Specifies precision (e.g., number of decimal places for floats).
* **`0`**: Left-pads numbers with zeros instead of spaces.
* **`#`**: Adds a `0x` prefix to hexadecimal numbers.

***

## Examples

### Calculate the Mean of CPU Time and Format the Result

This example demonstrates a common analysis workflow: calculate an average, convert the units, and then use `format()` to make the final number readable.

* **Query Example**
    ```
    avg(field=cputimeNanos)
    | cputime := (_avg/1000000)
    | format("%,.2f", field=_avg, as=_avg)
    ```

* **Step-by-Step**
    1.  `avg()`: First, the query calculates the average of the `cputimeNanos` field.
    2.  The next step converts the average from nanoseconds to milliseconds to make it more readable.
    3.  `format()`: This is the final step. It takes the calculated average and formats it to two decimal places (`.2f`) with a thousands separator (`,`).

* **Result Event Data**
    The final output is a single, cleanly formatted field.
    ```
    _avg
    0.14
    ```

---

### Create a Frequency Count with Formatted, Clickable Links

This example shows how `format()` can be used to create structured text, such as markdown links, making aggregated results interactive.

* **Query Example**
    ```
    top(repo)
    | format("[Link](https://example.com/%s)", field=repo, as=link)
    ```

* **Input Event Data**
    A series of events with a `repo` field.

| @timestamp | repo |
| :--- | :--- |
| 2023-06-15T10:00:00Z | "frontend-app" |
| 2023-06-15T10:05:00Z | "backend-api" |
| 2023-06-15T10:10:00Z | "frontend-app" |
| ... | ... |

* **Step-by-Step**
    1.  `top(repo)`: This function first finds the most frequent values in the `repo` field and provides a count for each.
    2.  `format()`: The results are then piped to `format()`. For each repository name, it constructs a markdown link string, inserting the repository name (from the `repo` field, represented by `%s`) into the URL. The result is stored in a new field named `link`.

* **Result Event Data**
    The output is a table showing each repository, its event count, and a new `link` column containing a clickable markdown link.

| repo | _count | link |
| :--- | :--- | :--- |
| frontend-app | 4 | `[Link](https://example.com/frontend-app)` |
| backend-api | 2 | `[Link](https://example.com/backend-api)` |
| monitoring-tool | 1 | `[Link](https://example.com/monitoring-tool)`|
| database-service | 1 | `[Link](https://example.com/database-service)`|

# formatDuration()

The `formatDuration()` function converts a numerical duration into a human-readable string (e.g., `6h30m52s`).

> **Important**
> The value of the field being formatted must be an integer. If the input value is a floating-point number, it will not be converted. [cite_start]You can use the `round()` or `format()` functions to convert it to an integer first. [cite: 3801, 3802, 3803]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: input field name | The name of the output field. |
| **field** | string | required | The name of the input field containing the duration. Must be an integer. [cite_start]The parameter name `field` can be omitted. [cite: 3792] |
| **from** | string | optional <br> default: `ms` | The time unit of the input field. [cite_start]Possible values are: `d` (days), `h` (hours), `m` (minutes), `s` (seconds), `ms` (milliseconds), `ns` (nanoseconds), `us` (microseconds). [cite: 3792] |
| **precision**| number | optional <br> default: 0 | The number of time units to include in the output string. [cite_start]A value of 0 includes all units. [cite: 3792] |

***

## Example

### Format a Duration with Limited Precision

This example demonstrates how to format a duration from a field, limiting the output to the three most significant time units to improve readability.

* **Query Example**
    ```
    formatDuration(duration, precision=3)
    ```

* **Step-by-Step**
    1.  The query starts with events that have a field named `duration` containing an integer value in milliseconds. [cite_start]For example, `23452553`. [cite: 3821, 3822]
    2.  [cite_start]`formatDuration()` reads the integer from the `duration` field. [cite: 3820]
    3.  [cite_start]The `precision=3` parameter instructs the function to display the result using only the three largest time units. [cite: 3823, 3824] [cite_start]For the value `23452553` ms (which is 6 hours, 30 minutes, 52 seconds, and 553 milliseconds), a precision of 3 would likely result in `6h30m52s`. [cite: 3822, 3824]

* **Summary and Results**
    This query is used to improve the readability of log entries that contain duration fields. By limiting the precision, you can present the most significant parts of the time duration, making it easier to understand at a glance. [cite_start]For instance, `6h30m` might be shown if `precision=2` was used. [cite: 3825, 3828]

# formatDuration()

The `formatDuration()` function converts a numerical duration into a human-readable string (e.g., `6h30m52s`).

> **Important**
> The value of the field being formatted must be an integer. If the input value is a floating-point number, it will not be converted. [cite_start]You can use the `round()` or `format()` functions to convert it to an integer first. [cite: 3801, 3802, 3803]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: input field name | The name of the output field. |
| **field** | string | required | The name of the input field containing the duration. Must be an integer. [cite_start]The parameter name `field` can be omitted. [cite: 3792] |
| **from** | string | optional <br> default: `ms` | The time unit of the input field. [cite_start]Possible values are: `d` (days), `h` (hours), `m` (minutes), `s` (seconds), `ms` (milliseconds), `ns` (nanoseconds), `us` (microseconds). [cite: 3792] |
| **precision**| number | optional <br> default: 0 | The number of time units to include in the output string. [cite_start]A value of 0 includes all units. [cite: 3792] |

***

## Example

### Format a Duration with Limited Precision

This example demonstrates how to format a duration from a field, limiting the output to the three most significant time units to improve readability.

* **Query Example**
    ```
    formatDuration(duration, precision=3)
    ```

* **Step-by-Step**
    1.  The query starts with events that have a field named `duration` containing an integer value in milliseconds. [cite_start]For example, `23452553`. [cite: 3821, 3822]
    2.  [cite_start]`formatDuration()` reads the integer from the `duration` field. [cite: 3820]
    3.  [cite_start]The `precision=3` parameter instructs the function to display the result using only the three largest time units. [cite: 3823, 3824] [cite_start]For the value `23452553` ms (which is 6 hours, 30 minutes, 52 seconds, and 553 milliseconds), a precision of 3 would likely result in `6h30m52s`. [cite: 3822, 3824]

* **Summary and Results**
    This query is used to improve the readability of log entries that contain duration fields. By limiting the precision, you can present the most significant parts of the time duration, making it easier to understand at a glance. [cite_start]For instance, `6h30m` might be shown if `precision=2` was used. [cite: 3825, 3828]

# geohash()

The `geohash()` function calculates a geohash value from two fields that represent latitude and longitude coordinates.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_geohash` | The name of the field that will be created to store the geohash value. |
| **lat** | string | optional <br> default: `ip.lat` | The field to use for the latitude coordinate. |
| **lon** | string | optional <br> default: `ip.lon` | The field to use for the longitude coordinate. |
| **precision**| number | optional <br> default: 12 | The precision to use for the calculation. |

***

## Example

### Calculate the Geohash Value for a Set of Coordinates

This example takes the latitude and longitude for London and calculates the corresponding geohash value. The latitude and longitude coordinates must be in decimal degrees.

* **Query Example**
    ```
    londonLat := 51.507222
    | londonLon := -0.1275
    | geohash(lat=londonLat, lon=londonLon)
    ```

* **Step-by-Step**
    1.  The query first creates two fields, `londonLat` and `londonLon`, with the respective coordinates for London.
    2.  The `geohash()` function is then called, using `londonLat` for the latitude and `londonLon` for the longitude.
    3.  The result is stored in a new field named `_geohash` by default.

* **Result Event Data**

| _geohash | londonLat | londonLon |
| :--- | :--- | :--- |
| gcpvj0e5m415 | 51.507222 | -0.1275 |

# getField()

The `getField()` function retrieves the value of a field whose name is not known beforehand but is instead computed dynamically at runtime.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_getField` | The name of the output field. |
| **source**| expression | required | An expression that evaluates to the name of the field you want to read. |

***

## Function Operation

This function works by first evaluating the `source` expression to get a string, and then it uses that resulting string as the name of the field to read from the event. This is particularly useful in two scenarios:
1.  When a field's name is constructed dynamically, for example, based on the a value from another field.
2.  When a field's name contains special characters like spaces or hyphens (e.g., `"host-name"`), which cannot be referenced directly in other functions.

***

## Examples

### Get the Last Element of an Array

This example demonstrates how to dynamically construct a field name to get the last element of an array, whatever its length may be.

* **Query Example**
    ```
    index := array:length("foo[]") - 1
    | fieldName := format("foo[%s]", field=[index])
    | result := getField(fieldName)
    ```

* **Input Event Data**
    An event with an array field:
    `{"foo": ["a", "b", "c", "d"]}`

* **Step-by-Step**
    1.  `array:length("foo[]") - 1`: First, the query calculates the index of the last element in the `foo` array (in this case, `4 - 1 = 3`).
    2.  `format("foo[%s]", ...)`: Next, it uses the calculated index to construct the full field name as a string: `"foo[3]"`. This string is stored in a new field called `fieldName`.
    3.  `getField(fieldName)`: Finally, `getField()` uses the value of the `fieldName` field (`"foo[3]"`) as the name of the field to look up, retrieving the value `d`.

* **Result Event Data**

| fieldName | result |
| :--- | :--- |
| foo[3] | d |

---

### Get a Value with a Direct vs. Indirect Field Name

This example clearly shows the difference between providing a static string (direct lookup) and providing a field name (indirect/dynamic lookup) to `getField()`.

* **Input Event Data**
    An event with two fields, where the value of `foo` is the name of another field, `bar`.
    ```
    foo: "bar"
    bar: 123
    ```

* **Query 1: Direct Lookup (Static Name)**
    ```
    result := getField("foo")
    ```
    **Result**: This looks for a field literally named "foo" and returns its value. `result` will be `"bar"`.

* **Query 2: Indirect Lookup (Dynamic Name)**
    ```
    result := getField(foo)
    ```
    **Result**: This first evaluates the `foo` field to get its value (`"bar"`). It then looks for a field with that name (`bar`) and returns *its* value. `result` will be `123`.

# groupBy()

The `groupBy()` query function groups events together based on the unique values in one or more specified fields and then performs an aggregate function on each group. By default, it counts the events in each group.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | array of strings | required | The field or fields to group by. The parameter name `field` can be omitted. |
| **function**| array of aggregate functions | optional <br> default: `count(as=_count)` | The aggregate function(s) to apply to each group. To get only unique values without aggregation, use an empty list: `function=[]`. |
| **limit** | string or integer | optional <br> default: 20,000 | The maximum number of groups to create. Can be set to `max` to use the system's maximum limit (default 1,000,000). |

***

## Function Operation

The `groupBy()` function is a powerful tool for summarizing data. It creates a new event for each unique combination of values in the specified `field`(s).

### Limits and Memory
It's important to be aware of the limits. `groupBy()` operates entirely in memory and has a maximum number of groups it can handle, controlled by system settings (`GroupDefaultLimit` and `GroupMaxLimit`). If the number of unique groups exceeds this limit, the function may return incomplete results and a warning. Similarly, if the function exceeds its internal memory allocation, it will stop adding new groups, which can lead to inconsistent results.

### Series Selection
When a `limit` is applied, `groupBy()` prioritizes keeping the "top-N" groups. The "top" groups are determined by the highest numerical values produced by the aggregate functions across all fields, not by the names of the groups themselves.

***

## Examples

### Nested groupBy() for Sub-Totals

This advanced example shows a `groupBy()` nested within another `groupBy()` to produce both a total count for a primary field (`method`) and sub-counts for a secondary field (`statuscode`) within each primary group.

* **Query Example**
    ```
    [groupBy(method, function=[count(as=method_total), groupBy(statuscode, function=count(as=method_status_count))])]
    ```

* **Step-by-Step**
    1.  The outer `groupBy(method, ...)` groups all events by the HTTP method (e.g., GET, POST).
    2.  For each method, it performs two aggregations:
        * `count(as=method_total)`: Counts all events for that method to get a total.
        * `groupBy(statuscode, ...)`: A nested grouping then takes the events for that method and further groups them by `statuscode`, counting the events for each status.
    
* **Result Event Data**
    The result is a table that shows the total count for each method, repeated for each sub-grouped status code.

| method | method_total | statuscode | method_status_count |
| :--- | :--- | :--- | :--- |
| GET | 14078 | 200 | 7326 |
| GET | 14078 | 301 | 1246 |
| GET | 14078 | 404 | 5371 |
| HEAD | 139 | 200 | 14 |
| HEAD | 139 | 404 | 97 |

---

### Alert Query for Parser Issues

This example uses `groupBy()` in a real-world security monitoring scenario to create a high-fidelity alert for persistent parser errors.

* **Query Example**
    ```
    #type=humio #kind=logs
    | loglevel=WARN
    | class = c.h.d.ParserLimitingJob
    | "Setting reject ingest for"
    | groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])
    | timeDiff := _max - _min
    | timeDiff > 300000 and _count > 10
    ```
* **Step-by-Step**
    1.  The query first filters logs to find specific warnings about parsers rejecting events.
    2.  `groupBy(id, ...)` groups these errors by the parser `id`.
    3.  For each parser, it calculates three metrics: the total `count()` of errors, the `min()` timestamp of the first error, and the `max()` timestamp of the last error.
    4.  The query then calculates the `timeDiff` (duration of the errors) and filters the results to alert only when a parser has had more than 10 errors (`_count > 10`) over a period of more than 5 minutes (`timeDiff > 300000`).
* **Summary and Results**
    This query effectively uses `groupBy()` to aggregate statistics that allow for intelligent alerting, ignoring minor glitches and focusing on significant, ongoing problems.

---

### Detect Brute-Force Attacks using `partition()`

This example showcases using `groupBy()` to apply a complex event sequence analysis with the `partition()` function to each potential actor (identified by `key`).

* **Query Example**
    ```
    head()
    | groupBy(
        key,
        function=partition(
          condition=test(status=="success"),
          split="after",
          [
            { status="failure" | count(as=failures) },
            range(@timestamp, as=timespan),
            selectLast(status)
          ]
        )
      )
    | failures >= 3
    | status = "success"
    ```
* **Step-by-Step**
    1.  `head()` ensures events are sorted by time.
    2.  `groupBy(key, ...)` applies the following logic to each unique `key` (e.g., a user ID or IP address).
    3.  `partition()`: This function splits the sequence of events for a given key every time a "success" event occurs.
    4.  Within each partition (a sequence of failures followed by one success), it counts the number of failures, calculates the time span of the failures, and selects the final status.
    5.  The final two lines filter these partitions to find only those that represent a potential brute-force attack: 3 or more failures followed by a success.
* **Result Event Data**
    The output is a list of successful login events that were preceded by at least three failures.

| key | failures | timespan | status |
| :--- | :--- | :--- | :--- |
| a | 5 | 1600 | success |
| c | 3 | 3100 | success |

---

### Create a Pivot Table

This example uses a nested `groupBy()` to prepare data, which is then piped to `transpose()` to pivot the results, turning rows into columns for a summary view.

* **Query Example**
    ```
    groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})
    | transpose(header=type)
    | drop(column)
    ```

* **Step-by-Step**
    1.  The nested `groupBy` first finds the `max(@timestamp)` for each `actor.user.id`.
    2.  The outer `groupBy` then groups this result by `type` (the action performed). This creates a flat table of `type`, `actor.user.id`, and `max` timestamp.
    3.  `transpose(header=type)` pivots this table. It uses the values from the `type` column as the new column headers.
    4.  `drop(column)` removes a leftover metadata field from the transpose operation, cleaning up the final output.
* **Result Event Data**
    The final output is a pivot table where the rows might be user IDs and timestamps, and the columns are the different types of actions, showing the last time each user performed each action.

| alert.clear-error | alert.create | alert.update |
| :--- | :--- | :--- |
| 1700546666592 | 1699004139419 | 1700546666676 |
| 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH |

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

# head()

The `head()` function retrieves a specified number of the oldest events first. It sorts events by timestamp (`@timestamp` or `@ingesttimestamp`) to determine which are the oldest.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **limit** | number | optional <br> default: 200 | The maximum number of events to return. |

***

## Function Operation

This function acts as an aggregator to ensure that events are processed in chronological order. This is a requirement for many sequence functions (like `accumulate()`, `neighbor()`, etc.) that depend on the order of events to calculate their results correctly.

***

## Example

### Calculate a Running Average of Field Values

This example uses `head()` to ensure events are properly ordered by time before calculating a running average with the `accumulate()` function.

* **Query Example**
    ```
    head()
    | accumulate(avg(value))
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| c | 1 |
| d | 2 |

* **Step-by-Step**
    1.  `head()`: This first step retrieves and orders the events by time, starting with the oldest ones.
    2.  `accumulate(avg(value))`: The ordered events are then piped to the `accumulate()` function, which computes a running average of the `value` field for each event in the sequence.

* **Result Event Data**
    The output shows a new `_avg` field where each row represents the cumulative average up to that point.

| _avg | key | value |
| :-- | :-- | :-- |
| 5 | a | 5 |
| 5.5 | b | 6 |
| 4 | c | 1 |
| 3.5 | d | 2 |

# if()

The `if()` function allows you to choose one of two expressions to execute based on a condition, similar to a ternary operator (`? :`) in many programming languages. Unlike a `case` statement, `if()` can be embedded as an expression inside other functions.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_if` | The name of the output field. |
| **condition**| expression | required | The conditional expression to evaluate. The parameter name `condition` can be omitted. |
| **else** | expression | required | The expression to execute if the condition is false. |
| **then** | expression | required | The expression to execute if the condition is true. |

***

## Examples

### Add a Field Based on Values of Another Field

This example uses multiple nested `if()` functions to categorize HTTP status codes into human-readable classes, demonstrating how to build `if-elseif-else` logic.

* **Query Example**
    ```
    statusClass := if(regex("^1", field=statuscode), then="informational", else=
                   if(regex("^2", field=statuscode), then="successful", else=
                   if(regex("^3", field=statuscode), then="redirection", else=
                   if(regex("^4", field=statuscode), then="client error", else=
                   if(regex("^5", field=statuscode), then="server error", else="unknown")))))
    ```
* **Step-by-Step**
    This query creates a new field called `statusClass`. It checks the `statuscode` field in a sequence:
    1.  If the status code starts with a "1", `statusClass` becomes "informational".
    2.  If not, it checks if it starts with a "2", setting `statusClass` to "successful".
    3.  This continues for "3" (redirection), "4" (client error), and "5" (server error).
    4.  If none of the conditions are met, `statusClass` is set to "unknown".

---

### Calculate a Percentage of Successful Status Codes Over Time

This example shows `if()` being used as a pre-processing step to create a binary field, which is then used in a more complex `timeChart()` aggregation to calculate a success rate.

* **Query Example**
    ```
    success := if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
          [sum(success, as=success), count(as=total)]
          | pct_successful := (success / total) * 100
          | drop([success, total])
        }
      ], span=15m, limit=100)
    ```

* **Step-by-Step**
    1.  `if(status >= 500, then=0, else=1)`: The query first creates a new field named `success`. It's set to `0` if the `status` is a server error (500 or greater) and `1` otherwise.
    2.  `timeChart()`: This function then takes the events. Inside its embedded aggregation:
        * `sum(success)` adds up all the `1`s to get a count of successful requests.
        * `count()` gets the total number of requests.
        * These two values are used to calculate the `pct_successful`.
        * The temporary `success` and `total` fields are then dropped from the final output.

* **Summary and Results**
    This query creates a time chart showing the percentage of successful requests per customer. The `if()` function is crucial for transforming the status codes into a numeric value that can be easily aggregated to calculate the percentage.

---

### Determine a Score Based on a Field's Percentile Rank

This example demonstrates using `if()` to assign a categorical score to an event based on where its `filesize` falls in a statistical distribution calculated by the `percentile()` function.

* **Query Example**
    ```
    percentile(filesize, percentiles=[40, 80], as=score)
    | symbol := if(filesize > score_80, then=":+1:", else=if(filesize > score_40, then="so-so", else=":-1:"))
    ```
* **Step-by-Step**
    1.  `percentile()`: This function first calculates the filesize values that correspond to the 40th and 80th percentiles, storing them in fields `score_40` and `score_80`.
    2.  `if()`: A nested `if()` statement then creates a new `symbol` field for each event:
        * If the event's `filesize` is greater than the 80th percentile value (`score_80`), the symbol is set to `:+1:`.
        * If not, it checks if the `filesize` is greater than the 40th percentile value (`score_40`), setting the symbol to `so-so`.
        * If neither is true, the symbol is set to `:-1:`.

* **Summary and Results**
    This query is a powerful way to categorize data based on its relative rank within a dataset, rather than on fixed thresholds. The `if()` function provides the conditional logic to assign these categorical labels.

# in()

The `in()` query function is a filter used to select events where a given field's value matches any of the values in a specified list.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | string | required | The field on which to filter events. The parameter name `field` can be omitted. |
| **ignoreCase** | boolean | optional <br> default: `false` | If set to `true`, the search will be case-insensitive. |
| **values** | array of strings | required | The list of values to match against the field. Only one match is required. Values can contain wildcards (`*`). |

***

## Function Operation

This function checks if the value of the specified `field` is present in the provided `values` list. It acts as an "OR" condition for multiple values. It can be negated by prefixing the call with an exclamation mark (e.g., `!in(...)`) to find events where the field value is *not* in the list.

***

## Examples

### Categorize Errors in Log Levels

This example uses `in()` as the condition within an `if()` function to create a new field that categorizes events based on their status code.

* **Query Example**
    ```
    critical_status := if(in(status, values=["500", "404"]), then="Critical", else="Non-Critical")
    ```

* **Input Event Data**
    ```
    srcIP=192.168.1.5 loglevel=ERROR status=404 user=admin
    srcIP=10.0.0.1 loglevel=INFO status=200 user=user1
    srcIP=192.168.1.15 loglevel=ERROR status=500 user=admin
    ```

* **Step-by-Step**
    1.  The query creates a new field named `critical_status`.
    2.  The `if()` function's condition uses `in()` to check if the `status` field contains either "500" or "404".
    3.  If the condition is true, `critical_status` is set to "Critical".
    4.  Otherwise, it is set to "Non-Critical".

* **Result Event Data**
    The output shows the original events enriched with the new `critical_status` category.

| srcIP | loglevel | status | user | critical_status |
| :--- | :--- | :--- | :--- | :--- |
| 192.168.1.5 | ERROR | 404 | admin | Critical |
| 10.0.0.1 | INFO | 200 | user1 | Non-Critical |
| 172.16.0.5 | WARN | 422 | user2 | Non-Critical |
| 192.168.1.15| ERROR | 500 | admin | Critical |
| 10.0.0.12 | DEBUG | 302 | user1 | Non-Critical |

---

### Categorize Events Based on Values in Multiple Fields

This advanced example uses a `case` statement with both `in()` and its negated form `!in()` to apply a sequence of rules for categorizing events into a `type` field.

* **Query Example**
    ```
    case {
      in(srcIP, values=["192.168.1.*"]) | type := "Internal";
      !in(loglevel, values=["DEBUG", "INFO"]) | type := "Critical";
      * | type := "Other"
    }
    ```
* **Input Event Data**
    ```
    srcIP=192.168.1.5 loglevel=ERROR status=404 user=admin
    srcIP=10.0.0.1 loglevel=INFO status=200 user=user1
    srcIP=172.16.0.5 loglevel=WARN status=422 user=user2
    ```

* **Step-by-Step**
    The `case` statement evaluates conditions in order:
    1.  It first checks if the `srcIP` starts with `192.168.1.`. If it does, the `type` is set to "Internal" and the statement stops for that event.
    2.  If the first condition is false, it checks if the `loglevel` is *not* "DEBUG" or "INFO". If true, `type` is set to "Critical".
    3.  If neither of the first two conditions is met, the wildcard `*` acts as a default case, setting `type` to "Other".

* **Result Event Data**
    The query assigns a `type` to each event based on the prioritized logic.

| srcIP | loglevel | status | user | type |
| :--- | :--- | :--- | :--- | :--- |
| 192.168.1.5 | ERROR | 404 | admin | Internal |
| 10.0.0.1 | INFO | 200 | user1 | Other |
| 172.16.0.5 | WARN | 422 | user2 | Critical |
| 192.168.1.15| ERROR | 500 | admin | Internal |
| 10.0.0.12 | DEBUG | 302 | user1 | Other |

---

### Differentiate Between Types of Log Levels

This example uses `in()` within a `match` statement to create a `severity` field based on the `loglevel`.

* **Query Example**
    ```
    loglevel match {
      /.*ERROR.*/ => severity := "High";
      in(values=["DEBUG", "INFO"]) => severity := "Low";
      * => severity := "Medium"
    }
    ```

* **Step-by-Step**
    The `match` statement checks the `loglevel` field:
    1.  If it contains the word "ERROR", `severity` is set to "High".
    2.  If not, it checks if the `loglevel` is `in` the list `["DEBUG", "INFO"]`. If so, `severity` is set to "Low".
    3.  If neither matches, the default case sets `severity` to "Medium" (this would catch levels like "WARN").

* **Result Event Data**
    Each event gets a `severity` rating based on its log level.

| srcIP | loglevel | status | user | severity |
| :--- | :--- | :--- | :--- | :--- |
| 192.168.1.5 | ERROR | 404 | admin | High |
| 10.0.0.1 | INFO | 200 | user1 | Low |
| 172.16.0.5 | WARN | 422 | user2 | Medium |
| 192.168.1.15| ERROR | 500 | admin | High |
| 10.0.0.12 | DEBUG | 302 | user1 | Low |

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

# ipLocation()

The `ipLocation()` function adds geolocation data to events based on an IPv4 or IPv6 address. It uses the MaxMind GeoLite2 database to enrich events with four new fields: `.country`, `.city`, `.lon` (longitude), and `.lat` (latitude).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | A prefix to use for the new geolocation fields. Defaults to the name of the input field. |
| **field** | string | optional <br> default: `ip` | The field containing the IP address to look up. The parameter name `field` can be omitted. |

***

## Function Operation

This function reads an IP address from the specified `field` and adds geolocation data to the event. For example, if the input field is `ip`, the function will add `ip.country`, `ip.city`, `ip.lon`, and `ip.lat`. The underlying GeoLite2 database is updated automatically if the cluster has a valid license. Note that city information may not be available for all IP addresses.

***

## Example

### Retrieve Location Data From a Specified Field

This example demonstrates how to use `ipLocation()` to get geolocation data for IP addresses that are located in the `@rawstring` of an event.

* **Query Example**
    ```
    ipLocation(field=@rawstring)
    ```

* **Input Event Data**
    A list of events where the raw string is an IP address.

| @rawstring |
| :--- |
| 165.225.194.1|
| 1.2.3.4 |
| 4.3.2.1 |
| 8.8.8.8 |

* **Step-by-Step**
    1.  The query starts with the source events.
    2.  `ipLocation(field=@rawstring)` tells the function to read the IP address from the `@rawstring` field of each event.
    3.  The function enriches each event with four new fields, using `@rawstring` as the prefix: `@rawstring.country`, `@rawstring.city`, `@rawstring.lon`, and `@rawstring.lat`.

* **Result Event Data**
    The output shows the original events now enriched with the new geolocation fields.

| @rawstring | @rawstring.city | @rawstring.country| @rawstring.lat |
| :--- | :--- | :--- | :--- |
| 165.225.194.1| Copenhagen | DK | 55.674 |
| 1.2.3.4 | <no value> | AU | -33.494 |
| 4.3.2.1 | <no value> | US | 37.751 |
| 8.8.8.8 | <no value> | US | 37.751 |
