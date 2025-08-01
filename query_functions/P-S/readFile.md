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