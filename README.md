# CBOR typeof tag
Specifies a [CBOR][cbor] tag applied to any CBOR item for describing it's type.

## Overview

This document specifies a [CBOR tag][iana-cbor-tags] 15 applied to any CBOR item for describing it's type:

```
Tag: 15
Data item: any
Semantics: Describes the type of the data item
Point of contact: 0xZensh <txr1883@gmail.com>
Description of semantics: https://github.com/cbor-schema/cbor-typeof-tag
```

This tag is used to express CBOR data structures (also known as CBOR Schema) in CBOR.

## Basic Type Examples
### Typeof Unsigned Integer
An example of expressing unsigned integer with default value `0`:
```
0xcf00    -- Diagnostic: 15(0)
CF        -- Tag 15
   00     -- 0
```

Valid CBOR items: `0`, `1`, `2`, ...

Invalid CBOR items: `-1`, `"abc"`, `null`, ...

### Typeof Negative Integer
An example of expressing negative integer with default value `-1`:
```
0xcf20    -- Diagnostic: 15(-1)
CF        -- Tag 15
   20     -- -1
```

Valid CBOR items: `-1`, `-2`, `-3`, ...

Invalid CBOR items: `0`, `"abc"`, `null`, ...

### Typeof Byte String
An example of expressing byte string with default zero bytes:
```
0xcf40    -- Diagnostic: 15(h'')
CF        -- Tag 15
   40     -- h''
```

Valid CBOR items: `h''`, `h'01020304'`, ...

Invalid CBOR items: `0`, `"abc"`, `null`, ...

### Typeof Text String
An example of expressing text string with default empty text:
```
0xcf60    -- Diagnostic: 15("")
CF        -- Tag 15
   60     -- ""
```

Valid CBOR items: `""`, `"abc"`, ...

Invalid CBOR items: `0`, `h''`, `null`, ...

### Typeof Array
#### An example of expressing array with any type of items:
```
0xcf80    -- Diagnostic: 15([])
CF        -- Tag 15
   80     -- []
```
Valid CBOR items: `[]`, `[1, 2]`, `[1, "abc", null]`, ...

Invalid CBOR items: `0`, `{}`, `null`, ...

#### An example of expressing array with unsigned integer items:
```
0xcf81cf00    -- Diagnostic: 15([15(0)])
CF            -- Tag 15
   81         -- Array of length 1
      CF00    -- 15(0)
```

Valid CBOR items: `[]`, `[1]`, `[1, 2]`, ...

Invalid CBOR items: `0`, `{}`, `null`, `[-1]`, ...

### Typeof Tuples
#### An example of expressing a two-tuple:
```
0xcf82cf00cf00    -- Diagnostic: 15([15(0), 15(0)])
CF                -- Tag 15
   82             -- Array of length 2
      CF00        -- 15(0)
      CF00        -- 15(0)
```
Valid CBOR items: `[0, 0]`, `[1, 2]`, `[1, 99]`, ...

Invalid CBOR items: `[0]`, `[0, 1, 2]`, `[0, ""]`, `{}`, `null`, ...

#### An example of expressing a triple:
```
0xcf83cf60cf00cf80    -- Diagnostic: 15([15(""), 15(0), 15([])])
CF                      -- Tag 15
   83                   -- Array of length 3
      CF60              -- 15("")
      CF00              -- 15(0)
      CF80              -- 15([])
```
Valid CBOR items: `["", 0, []]`, `["key", 1, [1, 2]]`, ...

Invalid CBOR items: `[0]`, `[0, 1, 2]`, `["", 0, null]`, `{}`, `null`, ...

### Typeof Object
#### An example of expressing object with any type of keys and values:
```
0xcfa0    -- Diagnostic: 15({})
CF        -- Tag 15
   A0     -- []
```
Valid CBOR items: `{}`, `{1: 2, 3: 4}`, `{"a": 1, "b": [2, 3]}`, ...

Invalid CBOR items: `0`, `[]`, `null`, ...

#### An example of expressing object with text string keys and unsigned integer values:
```
0xcfa1cf60cf00    -- Diagnostic: 15({15(""): 15(0)})
CF                -- Tag 15
   A1             -- Map of 1 key/value pair
      CF60        -- 15(""), typeof key
      CF00        -- 15(0), typeof value
```

Valid CBOR items: `{}`, `{"key1": 1}`, `{"key": 1, "key2": 1}`, ...

Invalid CBOR items: `0`, `{1: 2, 3: 4}`, `null`, `[]`, ...

#### An example of expressing object with specified keys and values:
```
0xcfa26161cf006162cf60    -- Diagnostic: 15({"a": 15(0), "b": 15("")})
CF                    -- Tag 15
   A2                 -- Map of 2 key/value pair
      6161            -- "a", specified key
         CF00         -- 15(0), typeof value
      6162            -- "b", specified key
         CF60         -- 15(""), typeof value
```

Valid CBOR items: `{"a": 1, "b": ""}`, `{"a": 1, "b": "abc"}`, ...

Invalid CBOR items: `{}`, `{"a": 1, 3: 4}`, `null`, `[]`, ...

### Typeof `null`
An example of expressing type of null:
```
0xcff6    -- Diagnostic: 15(null)
CF        -- Tag 15
   F6     -- null
```

Valid CBOR items: `null`

Invalid CBOR items: `-1`, `"abc"`, `undefined`, ...

### Typeof `undefined`
An example of expressing type of undefined:
```
0xcff7    -- Diagnostic: 15(undefined)
CF        -- Tag 15
   F7     -- undefined
```

Valid CBOR items: `undefined`

Invalid CBOR items: `-1`, `"abc"`, `null`, ...

### Typeof Boolean
#### An example of expressing type of boolean with default value `false`::
```
0xcff4    -- Diagnostic: 15(false)
CF        -- Tag 15
   F4     -- false
```

Valid CBOR items: `false`, `true`

Invalid CBOR items: `-1`, `"abc"`, `null`, ...

#### An example of expressing type of boolean with default value `true`::
```
0xcff5    -- Diagnostic: 15(true)
CF        -- Tag 15
   F5     -- true
```

Valid CBOR items: `false`, `true`

Invalid CBOR items: `-1`, `"abc"`, `null`, ...

## Union Types Examples
We use Tag 15 with Indefinite-length array to express union types.

### Typeof Integer
An example of expressing integer with default value `0`:
```
0xcf90cf00cf20ff    -- Diagnostic: 15([_ 15(0), 15(-1)])
CF                  -- Tag 15
   90               -- Start indefinite-length array
      CF00          -- 15(0), unsigned integer
      CF20          -- 15(-1), or negative integer
      FF            -- Break (end of array)
```

Valid CBOR items: `0`, `1`, `-1`, ...

Invalid CBOR items: `true`, `"abc"`, `null`, ...

### Typeof Object and `null`
An example of expressing object with specified keys and values:
```
cfa2646e616d65cf6063616765cf90cf00cff6ff    -- Diagnostic: 15({"name": 15(""), "age": 15([_ 15(0), 15(null)])})
CF                                          -- Tag 15
   A2                                       -- Map of 2 key/value pair
      646E616D65                            -- "name", specified key
         CF60                               -- 15(""), text string value
      63616765                              -- "age", specified key
         CF90CF00CFF6FF                     -- 15([_ 15(0), 15(null)]), unsigned integer value or null
```

Valid CBOR items: `{"name": "alice", "age": 18}`, `{"name": "alice", "age": null}`, ...

Invalid CBOR items: `{}`, `{"name": "alice", "age": -1}`, `{"name": "alice"}`, `{"name": "alice", "age": 18, "address": {}}`, ...

### Typeof Object and `undefined`
An example of expressing object with specified keys and values:
```
cfa2646e616d65cf6063616765cf90cf00cff7ff    -- Diagnostic: 15({"name": 15(""), "age": 15([_ 15(0), 15(undefined)])})
CF                                          -- Tag 15
   A2                                       -- Map of 2 key/value pair
      646E616D65                            -- "name", specified key
         CF60                               -- 15(""), text string value
      63616765                              -- "age", specified key
         CF90CF00CFF7FF                     -- 15([_ 15(0), 15(undefined)]), unsigned integer value or not exists (undefined)
```

Valid CBOR items: `{"name": "alice", "age": 18}`, `{"name": "alice"}`, ...

Invalid CBOR items: `{}`, `{"name": "alice", "age": -1}`, `{"name": "alice", "age": null}`, `{"name": "alice", "age": 18, "address": {}}`, ...

### Tuple or Triple
#### An example of expressing a tuple or triple:
```
0xcf83cf60cf00cf90cf80cff7ff    -- Diagnostic: 15([15(0), 15(""), 15([_ 15([]), 15(undefined)])])
CF                      -- Tag 15
   83                   -- Array of length 3
      CF60              -- 15("")
      CF00              -- 15(0)
      CF90CF80CFF7FF    -- 15([])
```
Valid CBOR items: `["", 0]`, `["", 0, []]`, `["key", 1, [1, 2]]`, ...

Invalid CBOR items: `[0]`, `[0, 1, 2]`, `["", 0, null]`, `{}`, `null`, ...

## Type Annotation Examples
We use Tag 15 with Indefinite-length array and a Indefinite-length map value to express type with annotation.

### Typeof Unsigned Integer with Type Annotation
An example of expressing unsigned integer with default value `0`, max value `100` and min value `0`:
```
0xcf90cf00bf636d696e00636d61781864ffff    -- Diagnostic: 15([_ 15(0), {_ "min": 0, "max": 100}])
CF                  -- Tag 15
   90               -- Start indefinite-length array
      CF00          -- 15(0), unsigned integer
      BF            -- Start indefinite-length map
         636d696e   -- "min", specified key
            00      -- 0, value
         636d6178   -- "max", specified key
            1864    -- 100, value
         FF         -- Break (end of map)
      FF            -- Break (end of array)
```

Valid CBOR items: `0`, `1`, `2`, ..., `100`

Invalid CBOR items: `-1`, `101`, `null`, ...

The annotation vocabularies can include "enum", "min", "max", and "comment" among others. These will be defined in the CBOR Schema.

[cbor]: https://datatracker.ietf.org/doc/html/rfc8949
[iana-cbor-tags]: https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml
