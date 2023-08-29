# Permutation Check Plugin (Version 1)

A plugin for asserting that two lists of wires are permutations of each other.

To use this plugin, include the following in your Circuit IR file:
```
@plugin permutation_check_v1;
```

## Function: `assert_perm`

The following function asserts that two lists of tuples, each tuple containing
`t` wires, are permutations of each other:
```
@plugin(permutation_check_v1, assert_perm, <t>)
```
A declaration of the plugin function MUST have: (1) two input ranges and zero
output ranges, (2) both input ranges must use the same field index and be of
equal length, and (3) the input ranges must be divisible by `t`. A declaration
of the plugin function is invalid if any of the above conditions do not hold.

### Example

The following example defines a function, `assert_perm_32`, that checks that two
lists are permutations of each other, where each tuple in the list contains 32
binary values.
```
@type field 2;
@plugin permutation_check_v1;

// Define a function that uses assert_perm to check that two lists
// containing 1024 tuples, each containing 32 wires, are permutations
// of each other.
@function(assert_perm_32, @in 0:32768, 0:32768)
    @plugin(permutation_check_v1, assert_perm, 32)

@call(assert_perm_32, $0 … $32767, $32768 … $65535)
```

---

**Distribution Statement “A”:** Approved for Public Release, Distribution Unlimited.

This material is based upon work supported by DARPA under Contracts No. HR001120C0087, HR001120C0086, HR001120C0085 and Agreement No. HR00112020021. Any opinions, findings and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of DARPA.
