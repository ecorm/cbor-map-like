# cbor-map-like

This document proposes a consolidated set of CBOR tags for map-like entities involving key-value pairs. These tags encode the following meta-data concerning map-like entities:

- the uniformity of the key and value types,
- the preservation of the insertion order of the key-value pairs,
- the uniqueness of the keys, and,
- the major type used to encode the key-value pairs.

## Summary

| Tag | LSBs | Uniform Value | Uniform Key | Ordering  | Duplicate Keys Allowed | Data Item | Related Tag |
| --- | ---- | ------------- | ----------- | --------- | ---------------------- | --------- | ----------- |
| 128 | 0000 | No            | No          | Unordered | No                     | map       | 259         |
| 129 | 0001 | No            | No          | Unordered | Yes                    | array     | TDB280*     |
| 130 | 0010 | No            | No          | Preserved | No                     | array     | TBD279*     |
| 131 | 0011 | No            | No          | Preserved | Yes                    | array     |             |
| 132 | 0100 | No            | Yes         | Unordered | No                     | map       | 275         |
| 133 | 0101 | No            | Yes         | Unordered | Yes                    | array     |             |
| 134 | 0110 | No            | Yes         | Preserved | No                     | array     |             |
| 135 | 0111 | No            | Yes         | Preserved | Yes                    | array     |             |
| 136 | 1000 | Yes           | Yes         | Unordered | No                     | map       |             |
| 137 | 1001 | Yes           | Yes         | Unordered | Yes                    | array     |             |
| 138 | 1010 | Yes           | Yes         | Preserved | No                     | array     |             |
| 139 | 1011 | Yes           | Yes         | Preserved | Yes                    | array     |             |

*TBD279: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

*TBD280: https://github.com/ecorm/cbor-tag-multimap

## Key/Value Type

Bits 2/3 of the tag provides information on the map's key and value types:

- 0b00xx Unspecified: There is no specified type for the map's keys and values
- 0b01xx Uniform Key: All keys have the same data type
- 0b10xx Uniform Key/Value: All values have the same data type in addition to all keys having the same data type (the types for keys and values may be distinct).

The semantics for uniformity shall be the same as for RFC8746 homogenous arrays (tag 41). That is, "which CBOR data items constitute elements of the same application type is specific to the application" (RFC8746, subsection 3.2).

Maps with arbitrary keys and uniform values are considered unusual, so they are left out of this specification in order to reduce the number of tags allocated (12 instead of 16).

## Ordering

Bit 1 of the tag represents the map's ordering semantics:

- 0: The order of key-value pairs is unspecified
- 1: Key-value pairs are encoded in the same order in which they were inserted

## Key Uniqueness

Bit 0 of the tag represents the uniqueness of the map's keys.

- 0: Keys are unique
- 1: Keys may be duplicated (i.e. multimaps)

## Data Item

The map is encoded as major type 5 if and only if

- the ordering is unspecified, and,
- the keys are unique.

Otherwise, the map is encoded as an array (major type 4) of key-value pairs as illustrated in the following example using CBOR diagnostic notation:

```
272(["key1", 1, "key2", 2])
```

## Related Tags (Informative)

### Tag 259

Specification: https://github.com/shanewholloway/js-cbor-codec/blob/master/docs/CBOR-259-spec--explicit-maps.md

The above proposed tag 128 may be used instead to guide a JavaScript decoder into intepreting a CBOR map as a JavaScript Map instead of an Object.

### Tag 275

Specification: https://github.com/ecorm/cbor-tag-text-key-map

The above proposed tag 132 may be used instead to guide a decoder into intepreting a CBOR map as a JavaScript-like Object having only text string keys. The decoder would have to verify the first key to establish that the map has uniform text string keys.

### Tag TBD279 ###

Draft specification: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

The above proposed tag 130 may be used instead to encode maps where the insertion order of key-value pairs must be preserved.

### Tag TDB280 ###

Draft specification: https://github.com/ecorm/cbor-tag-multimap

The above proposed tag 129 may be used instead to encode a multimap as an array of key-value pairs.

## Programming Language Containers (Informative)

The following subsections describe how the tags in this document relate to various programming language containers. Containers that are not part of the programming language or its standard libraries are not considered here.

The *Encoding Tag* column in the following tables provide the recommended tag that best represents the given container type. For example, it's possible to
use tag 132 for encoding an ECMAScript `Map` if all keys happen to be of the same type, however tag 128 is more general and applies to any `Map`. When encoding
an ECMAScript `Object`, tag 128 would be technically correct but is too general; tag 132 best presents the fact that an `Object` has text keys only.

The *Decodable Tags* colummn in the following tables, are for data items can be decoded into the destination container without having to inspect the following:

- the uniqueness of the keys,
- the ordering of the keys, and,
- the data types of **every** keys/value pair.

It may however be necessary to inspect the data types of the **first** key-value pair in the case of tags representing uniform keys/values.

### ECMAScript

Container         | Encoding Tag | Decodable Tags |
----------------- | ------------ | -------------- |
`Object`          | 132          | 132, 136       |
`Map`             | 128          | 128, 132, 136  |
`Array` of pairs  | 131          | All            |

### Python

Container           | Encoding Tag | Decodable Tags |
------------------- | ------------ | -------------- |
`TypedDict`         | 136          | 136            |
`namedtuple`        | 132          | 132, 136       |
`dict`              | 128          | 128, 132, 136  |
`OrderedDict`       | 130          | 130, 134, 138  |
`list` of 2-tuples  | 131          | All            |

### C++

Container(s)           | Encoding Tag | Decodable Tags |
---------------------- | ------------ | -------------- |
`Map<K, T>`            | 136          | 136            |
`Map<K, D>`            | 132          | 132, 136       |
`Map<D, D>`            | 128          | 128, 132, 136  |
`MultiMap<K, T>`       | 137          | 137            |
`MultiMap<K, D>`       | 133          | 133            |
`MultiMap<D, D>`       | 129          | 128, 129       |
`Sequence<Pair<K, T>>` | 139          | \[136, 139\]   |
`Sequence<Pair<K, D>>` | 135          | \[132, 139\]   |
`Sequence<Pair<D, D>>` | 131          | All            |

Legend:
- `K`: Static key type
- `T`: Static value type
- `D`: Suitable dynamic type, such as `std::any` or `std::variant`
- `Map`: `std::map` or `std::unordered_map`
- `MultiMap`: `std::multimap` or `std::unordered_multimap`
- `Sequence`: Sequence container that preserves order (e.g. `vector`)
- `Pair`: Object containing a key and a value, such as `std::pair`, or `std::tuple`.

Note that a C++ `std::map` stores its key-value pairs in a sorted fashion, and
does **not** preserve insertion order in the same manner as Python's `OrderedDict`.
