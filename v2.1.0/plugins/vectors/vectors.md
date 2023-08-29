# Vector Plugin (Version 1)
Author: Kimberlee Model

The `vectors_v1` plugin provides a limited number of operations which may be
implemented using vector computation for better performance.

## Operations

The following operations are supported.
* `add`: add pairs of elements from two vectors to form a third vector.
* `mul`: multiply pairs of elements from two vectors to form a third vector.
* `addc`: adds a constant value to each element of a vector.
* `mulc`: multiplies each element of a vector by a constant value.
* `add_scalar`: adds a scalar wire to each element of a vector.
* `mul_scalar`: multiplies each element of a vector by a scalar wire.
* `sum`: Add all elements from a single vector to form a single element.
* `product`: Multiply all elements from a single vector to form a single
  element.
* `dotproduct`: Multiply pairs of elements from two vectors, them sum to form a
  single element.

For example, using type 2 for vectors of size 8, the following multiply
operation could be used.
```
@function(vector_mul_2_8, @out: 2:8, @in: 2:8, 2:8)
  @plugin(vectors_v1, mul);
```

The vector plugin operations do not use any additional plugin arguments.
Instead, the plugin function signature must match the following constraints.

### `add` and `mul`

Add or multiply pairs of elements from two lists (of the same length). For a
given type `t` and length `s`, the following parameters are used.

#### Function Signature

* Output `t:s`: output vector
* Input `t:s`: left input vector
* Input `t:s`: right input vector

#### Plugin Binding

* Plugin Name: `vectors_v1`
* Operation: `add` or `mul`

### `addc` and `mulc`

Add or multiply each element of a vector by a constant value. For a given type
`t` and length `s`, the following parameters are used.

#### Function Signature

* Output `t:s`: output vector
* Input `t:s`: input vector

#### Plugin Binding

* Plugin Name: `vectors_v1`
* Operation: `addc` or `mulc`
* Number: constant value of the type `t`

### `add_scalar` and `mul_scalar`

Add or multiply each element of a vector by a scalar wire. For a given type `t`
and length `s`, the following parameters are used.

#### Function Signature

* Output `t:s`: output vector
* Input `t:s`: input vector
* Input `t:1`: input scalar

#### Plugin Binding

* Plugin Name: `vectors_v1`
* Operation `addc` or `mulc`

### `sum` and `product`

Add or multiply all elements of a list. For a given type `t` and length `s`, the
following parameters are used.

#### Function Signature

* Output `t:1`: The `sum` or `product` output wire
* Input `t:s`: The input vector

#### Plugin Binding

* Plugin Name: `vectors_v1`
* Operation: `sum` or `product`

### `dotproduct`

Multiply pairs of elements from two lists, then sum the resulting products. For
a given type `t` and length `s`, the following parameters are used.

#### Function Signature

* Output `t:1`: output wire
* Input `t:s`: left input vector
* Input `t:s`: right input vector

#### Plugin Binding

* Plugin Name: `vectors_v1`
* Operation: `dotproduct`

---

**Distribution Statement “A”:** Approved for Public Release, Distribution Unlimited.

This material is based upon work supported by DARPA under Contracts No. HR001120C0087, HR001120C0086, HR001120C0085 and Agreement No. HR00112020021. Any opinions, findings and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of DARPA.
