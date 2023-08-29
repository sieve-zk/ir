# RAM Plugin (Version 0)

RAM with read and write operations.

There are two variants of the RAM plugin, one intended for use in boolean
circuits and one for arithmetic:
* The `ram_bool_v0` plugin, designed for boolean circuits, requires that the
  field type be boolean, but allows addresses and values to consist of multiple
  field elements. The widths of addresses and values are set in the declaration
  of the plugin type.
* The `ram_arith_v0` plugin, designed for arithmetic circuits, works with any
  field type, but requires addresses and values to consist of a single field
  element each.

The two plugins otherwise provide the same functionality, as described below.

## `ram` type

### `ram_bool_v0` variant
```
@plugin(ram_bool_v0, ram, <field_idx>, <addr_count>, <value_count>, <num_allocs>, <total_alloc_size>, <max_live_alloc_size>)
```
Values of this type are identifiers for RAMs.
* `field_idx`: The index of a type. Must be a field type. This is used as the
  base type for both addresses and values.  For the `ram_bool_v0` variant, this is
  required to be a boolean field (i.e. the modulus of the chosen field must be
  2).
* `addr_count`: Number of field elements that make up each address. For
  comparison against size, the `addr_count` elements are interpreted as a single
  big-endian number.
* `value_count`: Number of field elements that make up each value.
* `num_allocs`: A numeric hint giving an upper bound on the number of RAMs of
  this type that will be created in this circuit.
* `total_alloc_size`: A numeric hint giving an upper bound on the sum of the
  sizes of all RAMs of this type that will be created in this circuit.
* `max_live_alloc_size`: A numeric hint giving an upper bound on the amount of
  live memory that may be in use at once among all RAMs of this type.  The
  amount of live memory in use is equal to the sum of the sizes of all RAMs that
  have been created so far and that have not yet been destroyed by passing the
  RAM wire to `@delete`.

### `ram_arith_v0` variant
```
@plugin(ram_arith_v0, ram, <field_idx>, <num_allocs>, <total_alloc_size>, <max_live_alloc_size>)
```
This type behaves the same as the `ram_bool_v0` type, except that `field_idx`
may refer to any field type, and `addr_count` and `value_count` are omitted as
they are implicitly required to be 1.

## `init`

```
@plugin(ram_bool_v0, init, <size>)
@plugin(ram_arith_v0, init, <size>)
```

Creates a new RAM with size cells, where valid addresses range from 0 to size-1,
and returns its identifier.

### Arguments

* `initial_value`: A single range containing `value_count` wires of type
  `field_idx`, where `addr_count` and `field_idx` are the values used in the ram
  type of the function’s output.  All cells of the RAM will initially be filled
  with this value.

### Results

* `ram`: A single wire of some `ram` type.

### Example
```
// Type 99 is a ram type:
@type @plugin(ram_bool_v0, ram, 0, 16, 32, 1, 1024, 1024)

@function(init_ram, @out 99:1, @in 0:32)
    @plugin(ram_bool_v0, init, 1024)

$1 <- @call(init_ram, $100 … $131)
```

## `read`

```
@plugin(ram_bool_v0, read)
@plugin(ram_arith_v0, read)
```

Reads a value from a given address in a RAM. Reads from out of bound addresses
result in a failure in the same manner as `@assert_zero`.

### Arguments

* `ram`: A single wire of some `ram` type.
* `addr`: A single range containing `addr_count` wires of type `field_idx`,
  where `addr_count` and `field_idx` are the values used in the `ram` type of
  the first argument.

### Results

* `value`: A single range containing `value_count` wires of type `field_idx`,
where `value_count` and `field_idx` are the values used in the `ram` type of the
first argument. Note that for the `ram_arith_v0` variant, `addr_count` and
`value_count` are always 1.

### Example

```
// Type 99 is a ram type:
@type @plugin(ram_bool_v0, ram, 0, 16, 32, 1, 1024, 1024)

@function(read_ram, @out 0:32, @in 99:1, 0:16)
    @plugin(ram_bool_v0, read)

$200 ... $231 <- @call(read_ram, $1, $100 ... $115)
```

## `write`

```
@plugin(ram_bool_v0, write)
@plugin(ram_arith_v0, write)
```

Writes a value to a given address in a RAM. Writes to out of bound addresses
result in a failure in the same manner as @assert_zero.

### Arguments

* `ram`: A single wire of some `ram` type.
* `addr`: A single range containing `addr_count` wires of type `field_idx`,
  where `addr_count` and `field_idx` are the values used in the `ram` type of
  the first argument.
* `value`: A single range containing `value_count` wires of type `field_idx`,
  where `value_count` and `field_idx` are the values used in the `ram` type of
  the first argument.

### Results

* None

Note that for the `ram_arith_v0` variant, `addr_count` and `value_count` are
always 1.

### Example

```
// Type 99 is a ram type:
@type @plugin(ram_bool_v0, ram, 0, 16, 32, 1, 1024, 1024)

@function(write_ram, @in 99:1, 0:16, 0:32)
    @plugin(ram_bool_v0, write)

@call(write_ram, $1, $100 ... $115, $200 ... $231)
```

---

**Distribution Statement “A”:** Approved for Public Release, Distribution Unlimited.

This material is based upon work supported by DARPA under Contracts No. HR001120C0087, HR001120C0086, HR001120C0085 and Agreement No. HR00112020021. Any opinions, findings and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of DARPA.
