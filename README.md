# cbor-container-tags

This document proposes a consolidated set of CBOR tags for providing traits on containers encoded as CBOR arrays or maps. This meta-data provides information on:

- whether the container is a dictionary or a list-like collection,
- the uniformity of the element types,
- the preservation of the element insertion order,
- the uniqueness of the keys or values, and,
- the major type used to encode the container elements.


## Summary

| Tag | LSBs  | Category   | Key Type  | Value Type | Ordering  | Duplicates Allowed | Data Item | ADT                 | Related Tag |
| --- | ----- | ---------- | --------- | ---------- | --------- | ------------------ | --------- | ------------------- | ----------- |
| 128 | 00000 | Dictionary | Arbitrary | Arbitrary  | Unordered | No  (keys)         | map       | Map<Any,Any>        | 259         |
| 129 | 00001 | Dictionary | Arbitrary | Arbitrary  | Unordered | Yes (keys)         | array     | Multimap<Any,Any>   | TDB280*     |
| 130 | 00010 | Dictionary | Arbitrary | Arbitrary  | Preserved | No  (keys)         | array     | IndexedMap<Any,Any> | TBD279*     |
| 131 | 00011 | Dictionary | Arbitrary | Arbitrary  | Preserved | Yes (keys)         | array     | List<Pair<Any,Any>> |             |
| 132 | 00100 | Dictionary | Arbitrary | Uniform    | Unordered | No  (keys)         | map       | Map<Any,T>          |             |
| 133 | 00101 | Dictionary | Arbitrary | Uniform    | Unordered | Yes (keys)         | array     | Multimap<Any,T>     |             |
| 134 | 00110 | Dictionary | Arbitrary | Uniform    | Preserved | No  (keys)         | array     | IndexedMap<Any,T>   |             |
| 135 | 00111 | Dictionary | Arbitrary | Uniform    | Preserved | Yes (keys)         | array     | List<Pair<Any,T>>   |             |
| 136 | 01000 | Dictionary | Uniform   | Arbitrary  | Unordered | No  (keys)         | map       | Map<K,Any>          | 275         |
| 137 | 01001 | Dictionary | Uniform   | Arbitrary  | Unordered | Yes (keys)         | array     | Multimap<K,Any>     |             |
| 138 | 01010 | Dictionary | Uniform   | Arbitrary  | Preserved | No  (keys)         | array     | IndexedMap<K,Any>   |             |
| 139 | 01011 | Dictionary | Uniform   | Arbitrary  | Preserved | Yes (keys)         | array     | List<Pair<K,Any>>   |             |
| 140 | 01100 | Dictionary | Uniform   | Uniform    | Unordered | No  (keys)         | map       | Map<K,T>            |             |
| 141 | 01101 | Dictionary | Uniform   | Uniform    | Unordered | Yes (keys)         | array     | Multimap<K,T>       |             |
| 142 | 01110 | Dictionary | Uniform   | Uniform    | Preserved | No  (keys)         | array     | IndexedMap<K,T>     |             |
| 143 | 01111 | Dictionary | Uniform   | Uniform    | Preserved | Yes (keys)         | array     | List<Pair<K,T>>     |             |
| 144 | 10000 | List       | None      | Arbitrary  | Unordered | No                 | array     | Set<Any>            |             |
| 145 | 10001 | List       | None      | Arbitrary  | Unordered | Yes                | array     | Bag<Any>            |             |
| 146 | 10010 | List       | None      | Arbitrary  | Preserved | No                 | array     | IndexedSet<Any>     |             |
| 147 | 10011 | List       | None      | Arbitrary  | Preserved | Yes                | array     | List<Any>           |             |
| 148 | 10100 | List       | None      | Uniform    | Unordered | No                 | array     | Set<T>              |             |
| 149 | 10101 | List       | None      | Uniform    | Unordered | Yes                | array     | Bag<T>              |             |
| 150 | 10110 | List       | None      | Uniform    | Preserved | No                 | array     | IndexedSet<T>       |             |
| 151 | 10111 | List       | None      | Uniform    | Preserved | Yes                | array     | List<T>             | 41          |

*TBD279: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

*TBD280: https://github.com/ecorm/cbor-tag-multimap


## Category

Bit 4 of the tag provides information on the container's category:

- 0b0xxxx: Dictionary: The elements are key-value pairs
- 0b1xxxx: Collection: The elements are discrete


## Uniformity

For both keys and values, the semantics for uniformity shall be the same as for RFC8746 homogenous arrays (tag 41). That is, "which CBOR data items constitute elements of the same application type is specific to the application" (RFC8746, subsection 3.2).

### Key Type

Bits 3 and 4 of the tag provide information on the container's key type:

- 0b00xxx: Arbitrary key type
- 0b01xxx: Uniform key type
- 0b10xxx: None (not a dictionary)


### Value Type

Bit 2 provides information on the map's value type:

- 0bxx0xx - Arbitrary Value Type
- 0bxx1xx - Uniform Value Type


## Ordering

Bit 1 of the tag represents the container's ordering semantics:

- 0: The order of the elements is unspecified
- 1: Elements are encoded in the same order in which they were inserted

## Uniqueness

Bit 0 of the tag represents the uniqueness of the container's keys or values.

Semantics for a Dictionary container:

- 0: Keys are unique
- 1: Keys may be duplicated (e.g. multimaps)

Semantics for a Collection container:

- 0: Elements are unique (e.g. sets)
- 1: Elements may be duplicated (e.g. lists, bags)


## Data Item

The container is encoded as major type 5 (CBOR map) if and only if

- its category is Dictionary,
- its ordering is unspecified, and,
- its keys are unique.

Otherwise, the container is encoded as major type 4 (CBOR array). In this situation, if the container happens to be a Dictionary, its key-value pairs shall be encoded as illustrated in the following example, shown using CBOR diagnostic notation:

```
272(["key1", 1, "key2", 2])
```

The rationale for not always using major type 4 (CBOR array) is that a receiver that is agnostic of this document's tags will at least be aware that the elements are key-value pairs when it receives major type 5 (CBOR map).

## Abstract Data Type (ADT)

The tag summary table itemizes the abstract data types that are associated with each tag value. The notation used in the ADT column is described below:

| Notation   | Description                                                          |
| ----       | -------------------------------------------------------------------- |   
| Any        | Dynamic type that can contain any suitable value                     |
| T          | Static value type                                                    |
| K          | Static key type                                                      |
| Pair       | Tuple containing a key and a value                                   |
| List       | Order-preserving sequence allowing duplicates                        |
| Set        | Non order-preserving collection of unique elements                   |
| Bag        | Non order-preserving collection of elements that may be duplicated   |
| IndexedSet | Set with unique elements that preserves insertion order              |
| Map        | Associative container mapping unique keys to values                  |
| MultiMap   | Map that allows duplicate keys                                       |
| IndexedMap | Map with unique keys that preserves insertion order                  |


## Related Tags (Informative)

### Tag 41

Specification: https://tools.ietf.org/html/rfc8746, section 3.2

Tag 151 from this specification may be used instead to indicate that a Collection has a homogenous element type.

### Tag 259

Specification: https://github.com/shanewholloway/js-cbor-codec/blob/master/docs/CBOR-259-spec--explicit-maps.md

Tag 128 from this specification may be used instead to guide a JavaScript decoder into intepreting a CBOR map as a JavaScript Map instead of an Object.

### Tag 275

Specification: https://github.com/ecorm/cbor-tag-text-key-map

Tag 136 from this specification may be used instead to guide a decoder into intepreting a CBOR map as a JavaScript-like Object having only text string keys. The decoder would have to verify the first key to establish that the map has uniform text string keys.

### Tag TBD279 ###

Draft specification: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

Tag 130 from this specification may be used instead to encode maps where the insertion order of key-value pairs must be preserved.

### Tag TDB280 ###

Draft specification: https://github.com/ecorm/cbor-tag-multimap

Tag 129 from this specification may be used instead to encode a multimap as an array of key-value pairs.
