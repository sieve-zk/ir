# Extended Arithmetic (Version 1)
Author: Kimberlee Model

The `extended_arithmetic_v1` plugin provides additional arithmetic operations
over the basic addition and multiplication, easing the development of a broader
range of statements.

## Operations

The following operations are supported.

* `less_than` Compares two inputs, returning 1 if the first input is less than
  the second input, or 0 otherwise.
* `less_than_equal` Compares two inputs, returning 1 if the first input is less
  than or equal to the second input, or 0 otherwise.
* `division` Divides two inputs as if they were integers, returning first the
  quotient and then the remainder.
* `bit_decompose` Decomposes a single input into a bit vector, returning it as a
  range whose length is the prime’s width.

```
@function(lt, @out: 0:1, @in: 0:1, 0:1)
  @plugin(extended_arithmetic_v1, less_than);
@function(lte, @out: 0:1, @in: 0:1, 0:1)
  @plugin(extended_arithmetic_v1, less_than_equal);
@function(div, @out: 0:1, 0:1, @in: 0:1, 0:1)
  @plugin(extended_arithmetic_v1, division);
@function(bit_decomp, @out: 0:7, @in: 0:1)
  @plugin(extended_arithmetic_v1, bit_decompose);
```

### `less_than` and `less_than_equal`

These compare two inputs and indicate 1 or 0 for truth or false. For a given
type `t` the following parameters are used.

####  Function Signature

* Output `t:1`: 1 or 0
* Input `t:1`: Left side of comparison
* Input `t:1`: Right side of comparison

#### Plugin Binding

* Plugin Name: `extended_arithmetic_v1`
* Operation: `less_than` or `less_than_equal`

### `division`

This operation divides one input by the other, returning a quotient and a
remainder. For a given type `t` the following parameters are used.

#### Function Signature

* Output `t:1`: Quotient
* Output `t:1`: Remainder
* Input `t:1`: Numerator/dividend
* Input `t:1`: Denominator/divisor

#### Plugin Binding

* Plugin Name: `extended_arithmetic_v1`
* Operation: `division`

### `bit_decompose`

The operation accepts one input, `n`, and decomposes it into a bit vector. The
vector, `v`, is represented as a range of output wires with each element chosen
from `{0,1}`, such that `sum(vi * 2l-i) == n`. The output vector’s length, `l`,
must be exactly `ceiling(log2(p))`, were `p` is the field’s prime. The most
significant bit is stored first and the least significant bit is last. For a
given type `t` with prime or modulus `p`, the following parameters are used.

#### Function Signature

* Output `t:ceiling(log2(p))`: The decomposed bits
* Input: `t:1`: the value to decomposed

#### Plugin Binding

* Plugin Name: `extended_arithmetic_v1`
* Operation: `bit_decompose`

---

**Distribution Statement “A”:** Approved for Public Release, Distribution Unlimited.

This material is based upon work supported by DARPA under Contracts No. HR001120C0087, HR001120C0086, HR001120C0085 and Agreement No. HR00112020021. Any opinions, findings and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of DARPA.
