---
layout: post
title: postgres concatanation pitfall using || vs concat
---

TL; DR just have look on result of example below, and be aware to make proper casts if using `||`

```
vagrant=> SELECT null || 'foo';
 ?column? 
----------
 
(1 row)

vagrant=> SELECT concat(null, 'foo');
 concat 
--------
 foo
(1 row)
```




> Note: Before PostgreSQL 8.3, these functions would silently accept values of several non-string data types as well, due to the presence of implicit coercions from those data types to text. Those coercions have been removed because they frequently caused surprising behaviors. However, the string concatenation operator (`||`) still accepts non-string input, so long as at least one input is of a string type, as shown in Table 9-6. For other cases, insert an explicit coercion to text if you need to duplicate the previous behavior.

from docs [ https://www.postgresql.org/docs/9.1/static/functions-string.html]( https://www.postgresql.org/docs/9.1/static/functions-string.html)