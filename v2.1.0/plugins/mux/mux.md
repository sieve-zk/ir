# Multiplexer Plugin (Version 1)

The multiplexer plugin provides the ability to select an output from amongst
multiple inputs. The mux operations accept groups of candidate inputs and select
one output based on a selector wire. The decode operation may be combined with a
dotproduct operation (from the vectors plugin) to form a multiplexer with a
signature differing from the original operations.

## `mux`
```
@plugin(mux_v1, <permissiveness>)
```

Chooses between multiple inputs based on a conditional value. The `permissiveness`
generic  argument must be one of the identifiers `permissive` or `strict`.

### Arguments

* `cond`: A wire range of any field type. If the field type is boolean, the wire
  range can have any count; otherwise (when used with arithmetic fields), the
  wire count must be exactly 1. The value on this wire (or, in the boolean case,
  the values of the wires interpreted as a single big-endian integer) is the
  conditional value that selects one of the input sets.
* `x_i`: `N` sets of wire ranges, each such set having the same types and counts
  as the result. The types must have the same type as `cond`. `x_i` is the `i`th
  branch input (where `i` ranges from 0 to `N`-1). If the number of cases, `N`,
  is greater than either the prime or 2 to the power of bit-width, the plugin
  may cause the circuit to be invalid.

### Results

* `out`: Any number of wire ranges whose types are all equal to the type of `cond`,
  with any counts. If the `cond` wire range (interpreted as a single big-endian
  integer, if necessary) has a value `i` in the range 0 to `N`-1, then output value
  is the value of `x_i`. If the value of the cond wire range is not in the range 0
  to `N`-1, then a permissive multiplexer returns zero, while a strict multiplexer
  results in a failure in the same manner as `@assert_zero`.

Note, this operation may use the `mux_v1` plugin name, or the `mux_v0` plugin
name, for backwards compatibility.

### Example

```
@function(mux_example, @out 0:5, 0:1 @in 0:1, 0:5, 0:1, 0:5, 0:1)
    @plugin(mux_v1, strict)

$600 ... $604, $700 <- @call(mux_example, $100, $200 ... $204, $300, $400 ...  $404, $500)

// If $100 is zero, the mux operation copies the first set of inputs,
// similar to the behavior of this code:
$600 ... $604 <- @call(copy4, $200 ... $204)
$700 <- @call(copy1, $300)

// If $100 is one, the mux operation instead copies the second set of
// inputs:
$600 ... $604 <- @call(copy5, $400 ... $404)
$700 <- @call(copy1, $500)
```

### Example (with multiple cases)

```
@function(mux4_vec8, @out: 0:8, @in: 0:1, 0:8, 0:8, 0:8, 0:8)
    @plugin(mux_v1, strict);
$600 ... $607 <- @call(mux4_vec8, $100, $200 ... $207, $300 ... $307, $400 ... $407, $500 ... $507)

// If $100 is zero, the mux operation copies the first set of inputs,
// similar to the behavior of this code:
$600 ... $607 <- @call(copy8, $200 ... $207)

// If $100 is one, the mux operation instead copies the second set of
// inputs:
$600 ... $607 <- @call(copy8, $300 ... $307)

// If $100 is two, the mux operation instead copies the third set of
// inputs:
$600 ... $607 <- @call(copy8, $400 ... $407)

// If $100 is three, the mux operation instead copies the fourth set of
// inputs:
$600 ... $607 <- @call(copy8, $500 ... $507)
```

## `decode`

The `decode` operation accepts a wire (the selector) and returns a vector. The
return vector will have all indexes set to zero, except for the index
corresponding to the input wire’s value, which will be set to one. The return
vector may be used as a parameter to `dotproduct` along with the multiplexer’s
input vector.

The `decode` operation has two modes: `strict` and `permissive`, same as the
original `mux` operations. In `strict` mode, an out of bounds selector will have
the effect of failing an `@assert_zero`. In `permissive` mode, an out of bounds
selector will return 0.

With some type `t`, and vector length `s` the following parameters are allowed.
When `t` is the prime field `GF(2)`, a bit-width `w` is also used. If `s` is
greater than either the prime (or type appropriate overflow value) or `2^w`, the
plugin may cause the circuit to be invalid.

#### Function Signature

* Output `t:s`: the return vector of zeroes and a one.
* Input `t:1` or `t:w`: the selector wire

#### Plugin Binding

* Plugin Name: `mux_v1`
* Operation: `decode`
* Mode: `strict` or `permissive`

#### Example
```
@plugin mux_v1;
@plugin vectors_v1;
@type field 127;
@type field 101;
@convert(@out: 1:1, @in: 0:1);
@begin

// Decoder from the Mux plugin
//                           out_vec   selector
@function decoder4_127(@out: 0:4, @in: 0:1)
  @plugin(mux_v1, decode, strict);

// Dot product from the vectors plugin
//                       out       l_vec r_vec
@function dot4_127(@out: 0:1, @in: 0:4,  0:4)
  @plugin(vectors_v1, dotprod);
  
@function dot4_101(@out: 1:1, @in: 1:4,  1:4)
  @plugin(vectors_v1, dotprod);

// Mux is a regular function now.
//                       out       selector candidates
@function mux4_127(@out: 0:1, @in: 0:1,     0:4)
  $6...$9 <- @call(decoder4_127, $1);
  $0 <- @call(dot4_127, $6...$9, $2...$5);
@end

// Multifield is easy now, with well specified convert gates
//                         out127 out101    selector c_127 c_101
@function mux4_multi(@out: 0:1,   1:1, @in: 0:1,     0:4,  1:4)
  $6...$9 <- @call(decoder4_127, $1);
  $0 <- @call(dot4_127, $6...$9, $2...$5);

  @new(1: $5...$8);
  1:$5 <- @convert(0: $6);
  1:$6 <- @convert(0: $7);
  1:$7 <- @convert(0: $8);
  1:$8 <- @convert(0: $9);

  $0 <- @call(dot4_101, $5...$9, $1...$4);
@end
```

---

**Distribution Statement “A”:** Approved for Public Release, Distribution Unlimited.

This material is based upon work supported by DARPA under Contracts No. HR001120C0087, HR001120C0086, HR001120C0085 and Agreement No. HR00112020021. Any opinions, findings and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of DARPA.
