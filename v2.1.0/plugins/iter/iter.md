# Iteration Plugin (Version 1)

Map operations on lists.

A “list” is a range of wires to be processed in sequence. The list is divided up into a number of
equally-sized elements, each consisting of one or more wires. For example, the range
$300 ... $309 might be interpreted as a list of ten 1-wire elements, a list of five 2-wire
elements, or a list of one 10-wire element, depending on the operation being performed.

An operation can take and return multiple lists. For the operations defined here, all input and
output lists must have equal lengths in terms of elements. However, the actual wire count may
vary. For example, a map operation could take the range $300 ... $309 as a list of five 2-
wire elements and produce the range $400 ... $404 as a list of five 1-wire elements; as the
number of elements is the same, this map operation is valid.

When an operation takes or returns multiple lists, all input and output lists are traversed in
lockstep. In map, for example, the first call to the closure receives the first element from each of
the input lists, and the elements it returns are stored into the first position of the output lists, and
similarly for the second and later calls.

Operations can take a number of “environment” arguments, which are passed through to each
iteration. This can be used to emulate closures, so that the function that handles each iteration
can access wires that are independent of the list being iterated over.

## `map`
```
@plugin(iter_v0, map, <func_name>, <num_env>, <iter_count>)
```
Maps over one or more lists of `iter_count` elements, producing one or more output lists also
having length `iter_count`.

#### Arguments of function `func_name`

* `env`: `num_env` wire ranges of any types and counts. This is the closure environment,
which will be passed unchanged to `func_name` each time `func_name` is called.
* `in_elems`: Any number of wire ranges of any types and counts.

#### Results of function `func_name`

* `out_elems`: Any number of wire ranges of any types and counts.

#### Arguments of the `map` operation

* `env`: Zero or more wire ranges, matching the `env` arguments of `func_name`.
* `in_lists`: Zero or more wire ranges, matching the `in_elems` arguments of
`func_name`, except that all counts are multiplied by `iter_count`.

#### Results of the `map` operation

* `out_lists`: Zero or more wire ranges, matching the `out_elems` results of
`func_name`, except that all counts are multiplied by `iter_count`.

#### Example
```
@function(foo, @out 0:1, @in 0:1, 1:2, 2:3)
    // ...
@end

@function(map_foo_5, @out 0:5 @in 0:1, 1:10, 2:15)
    @plugin(iter_v0, map, foo, 1, 5)

$400 ... $404 <- @call(map_foo_5, $100, $200 ... $209, $300 ... $314)

// Equivalent:
$400 <- @call(foo, $100, $200 ... $201, $300 ... $302)
$401 <- @call(foo, $100, $202 ... $203, $303 ... $305)
$402 <- @call(foo, $100, $204 ... $205, $306 ... $308)
$403 <- @call(foo, $100, $206 ... $207, $309 ... $311)
$404 <- @call(foo, $100, $208 ... $209, $312 ... $314)
```

## `map_enumerated`
```
@plugin(iter_v0, map_enumerated, <func_name>, <num_env>, <iter_count>)
```

Similar to `map`, but passes an incrementing counter to each iteration.

#### Arguments of function `func_name`

* `env`: `num_env` wire ranges of any types and counts. This is the closure environment,
which will be passed unchanged to `func_name` each time `func_name` is called.
* `index`: A single wire range of any field type with any count. The iteration counter will be
passed here, starting at 0 and incrementing with each iteration. The index wires are
treated as a single big-endian number. If the iteration count would exceed the maximum
representable value, it wraps around to 0.
* `in_elems`: Any number of wire ranges of any types and counts.

#### Results of function `func_name`

* `out_elems`: Any number of wire ranges of any types and counts.

#### Arguments of the `map` operation

* `env`: Zero or more wire ranges, matching the `env` arguments of `func_name`.
* `in_lists`: Zero or more wire ranges, matching the `in_elems` arguments of
`func_name`, except that all counts are multiplied by `iter_count`.

#### Results of the `map` operation

* `out_lists`: Zero or more wire ranges, matching the `out_elems` results of
`func_name`, except that all counts are multiplied by `iter_count`.

#### Example
```
@function(foo, @out 0:1, @in 0:1, 0:1, 1:2, 2:3)
    // ...
@end

@function(map_enumerated_foo_5, @out 0:5 @in 0:1, 1:10, 2:15)
    @plugin(iter_v0, map_enumerated, foo, 1, 5)

$400 ... $404 <- @call(map_enumerated_foo_5, $100, $200 ... $209, $300 ... $314)

// Equivalent:
$990 <- < 0 >
$400 <- @call(foo, $100, $990, $200 ... $201, $300 ... $302)
$991 <- < 1 >
$401 <- @call(foo, $100, $991, $202 ... $203, $303 ... $305)
$992 <- < 2 >
$402 <- @call(foo, $100, $992, $204 ... $205, $306 ... $308)
$993 <- < 3 >
$403 <- @call(foo, $100, $993, $206 ... $207, $309 ... $311)
$994 <- < 4 >
$404 <- @call(foo, $100, $994, $208 ... $209, $312 ... $314)
```
---

**Distribution Statement “A”:** Approved for Public Release, Distribution Unlimited.

This material is based upon work supported by DARPA under Contracts No. HR001120C0087, HR001120C0086, HR001120C0085 and Agreement No. HR00112020021. Any opinions, findings and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of DARPA.
