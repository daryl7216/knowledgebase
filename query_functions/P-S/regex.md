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