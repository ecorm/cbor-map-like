# cbor-map-like

This document proposes a consolidated set of CBOR tags for map-like entities involving key-value pairs. These tags encode the following meta-data concerning map-like entities:

- the homogeneity of the key and value types,
- the preservation of the insertion order of the key-value pairs,
- the uniqueness of the keys, and,
- the major type used to encode the key-value pairs.

## Summary

| Tag | LSBs | Homogenous Value | Homogenous Key | Ordering  | Unique Keys | Data Item | Related Tag |
| --- | ---- | ---------------- | -------------- | --------- | ----------- | --------- | ----------- |
| 128 | 0000 | No               | No             | Unordered | Yes         | map       | 259         |
| 129 | 0001 | No               | No             | Unordered | No          | array     | TDB280*     |
| 130 | 0010 | No               | No             | Preserved | Yes         | array     | TBD279*     |
| 131 | 0011 | No               | No             | Preserved | No          | array     |             |
| 132 | 0100 | No               | Yes            | Unordered | Yes         | map       | 275         |
| 133 | 0101 | No               | Yes            | Unordered | No          | array     |             |
| 134 | 0110 | No               | Yes            | Preserved | Yes         | array     |             |
| 135 | 0111 | No               | Yes            | Preserved | No          | array     |             |
| 136 | 1000 | Yes              | Yes            | Unordered | Yes         | map       |             |
| 137 | 1001 | Yes              | Yes            | Unordered | No          | array     |             |
| 138 | 1010 | Yes              | Yes            | Preserved | Yes         | array     |             |
| 139 | 1011 | Yes              | Yes            | Preserved | No          | array     |             |

*TBD279: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

*TBD280: https://github.com/ecorm/cbor-tag-multimap

## Key/Value Type

Bits 2/3 of the tag provides information on the map's key and value types:

- 0b00 Unspecified: There is no specified type for the map's keys and values
- 0b01 Homogenous Key: All keys have the same data type
- 0b10 Homogenous Key/Value: All values have the same data type in addition to all keys having the same data type (the types for keys and values may be distinct).

Maps with heterogenous keys and homogenous values are considered unusual, so they are left out of this specification in order to reduce the number of tags allocated (12 instead of 16).

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

## Related Tags

### Tag 259

Specification: https://github.com/shanewholloway/js-cbor-codec/blob/master/docs/CBOR-259-spec--explicit-maps.md

The above proposed tag 128 may be used instead to guide a JavaScript decoder into intepreting a CBOR map as a JavaScript Map instead of an Object.

### Tag 275

Specification: https://github.com/ecorm/cbor-tag-text-key-map

The above proposed tag 132 may be used instead to guide a decoder into intepreting a CBOR map as a JavaScript-like Object having only text string keys. The decoder would have to verify the first key to establish that the map has homogeneous text string keys.

### Tag TBD279 ###

Draft specification: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

The above proposed tag 130 may be used instead to encode maps where the insertion order of key-value pairs must be preserved.

### Tag TDB280 ###

Draft specification: https://github.com/ecorm/cbor-tag-multimap

The above proposed tag 129 may be used instead to encode a multimap as an array of key-value pairs.
