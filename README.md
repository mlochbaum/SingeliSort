# Singeli sort

Algorithms in [Singeli](https://github.com/mlochbaum/Singeli), aiming for high performance and broad adaptivity for sorting CPU-native types (integers and floats). A secondary goal is a well-commented and readable codebase that explains what various methods are good for and how they're implemented to use hardware to its full potential. Will probably end up using SIMD if available to speed up a few things, but this is primarily intended to be a portable rather than SIMD-first sort.

Compile with (add `-t cpp` for C++ compatibility):

    singeli src/sort.singeli -o sort.c

Or the following without a Singeli install. CBQN builds on Unix-like systems (including macOS and WSL) in under a minute; see [docker build](https://github.com/vylsaz/cbqn-win-docker-build) for Windows. There's also a pre-compiled copy of sort.c in `compiled/sort.c`. It may not always be up to date.

    git clone https://github.com/dzaima/CBQN.git
    cd CBQN && make && cd -
    git clone https://github.com/mlochbaum/Singeli.git
    CBQN/BQN Singeli/singeli src/sort.singeli -o sort.c

To benchmark:

    gcc -O3 bench.c
    ./a.out

Exported functions are defined in src/sort.singeli, and their C prototypes appear at the bottom of sort.c: the arguments for sorts are array, length, aux (or scratch buffer), and possibly aux length in bytes. These are likely to change over time.

## Overview

Singeli sort currently hybridizes the following algorithms; all are used for `sort32` and other functions use subsets. The overall structure is that the glidesort layer may call quicksort, which calls the different base cases in various situations.

- Quicksort partitioning from [fluxsort](https://github.com/scandum/fluxsort) and [crumsort](https://github.com/scandum/crumsort)
- Outer merge layer: modified [glidesort](https://github.com/orlp/glidesort) ([powersort](https://github.com/sebawild/powersort) rules made lazy to defer to quicksort if runs aren't found)
- Merging: based on [piposort](https://github.com/scandum/piposort)
- Small arrays: sorting networks as in [ipn_unstable](https://github.com/Voultapher/sort-research-rs/blob/main/src/unstable/rust_ipn.rs), extra merging and insertion following [quadsort](https://github.com/scandum/quadsort)
- Radix sort: mostly like [ska_sort_copy](https://github.com/skarupke/ska_sort)
- Counting sort: see [section](https://github.com/mlochbaum/rhsort#counting-sort) in rhsort
- [Robin Hood](https://github.com/mlochbaum/rhsort) sort

In progress, still has various issues:

- Drapesort, similar to [drop-Merge sort](https://github.com/emilk/drop-merge-sort)

Other methods to consider later:

- In-place partitioning with [pdqsort](https://github.com/orlp/pdqsort). Slower than crumsort but it does adapt to mostly-sorted data well.
- Interleaved merges and bidirectional partitioning from glidesort. These have not yet been demonstrated to improve performance relative to fluxsort, and there are indications that they slow things down on older processors in addition to bumping up code size. I'll wait for the paper explaining choices made before looking into them further.

## Guide to the source

The source code is supposed to be the place to go to get full descriptions and details. I am certain it fails in this role—particularly in places where I don't expect anyone's reading, so please complain if a part you've chosen to read is not well explained!

The general-use files:

- sort.singeli Main file: include statements, and the sorting function definitions.
- base.singeli Basic definitions to be used elsewhere. This includes operators, which are all user-defined in Singeli.
- common.singeli Other definitions that are more specific than base but may be used in multiple places.
- arith.singeli Some log and square root code to keep it out of the way.

And specific algorithms:

- quicksort.singeli
  - partition.singeli Partitioning
  - median.singeli Medians and pseudomedians for picking candidates
    - xorshift.singeli Pseudo-random number generator (PRNG) avoids patterns
- merge.singeli Merging utilities and pisort
- glide.singeli Glidesort strategy: use merges for natural runs
  - (merge.singeli)
- small.singeli Small array sorting
  - network.singeli Sorting networks for some fixed sizes
  - ins.singeli Insertion sorting
  - (merge.singeli)
- radix.singeli Radix sorts
  - prefix.singeli Prefix sums
- count.singeli Counting sort
  - (prefix.singeli)
- rh.singeli Robin Hood sort, for evenly distributed data
- dropsort.singeli (unused) Dropsorts for nearly-sorted arrays

Some quick notes on Singeli. Everything in brackets `{}` is run at compile time, so a call like `dist{dn}{U, minv, maxv}` is all inlined (`dist` is called a generator). Functions are called with parentheses and are used rarely, for things that are exported, used in many places, or recursive.

All operators are user-defined, with many picked up from standard includes `skin/c` and `skin/cext`. Some of the ones that are unfamiliar relative to C are listed below.

| Syntax      | Meaning
|-------------|---------
| `x <{dn} y` | Compare `x` and `y`, flipping ordering if `dn` is `1`
| `x -> i`    | Value at offset `i` from pointer `x`
| `x <- v`    | Store `v` at pointer `x`
| `x <-{i} v` | Store `v` at offset `i` from pointer `x`
| `x <-> y`   | Swap values at pointers `x` and `y`
| `a <~> b`   | Swap variables of variables `a` and `b`
| `T~~v`      | Cast `v` to same-width type `T`
| `T^~v`      | Promote `v` to supertype `T`
| `T<~v`      | Narrowing conversion of `v` to type `T`

Singeli sort uses a fair amount of compile-time trickery to support lots of sorting functions while keeping the code reasonably clean. Functions all support sorting in both directions (`dn` is `0` for up and `1` for down), and many of them support a sort-by operation that actually passes around a tuple of pointers: one to be sorted and others to be moved in the same pattern. A related operation is "grade", which reorders indices as the data should be ordered, and leaves the data intact (it may partially or completely sort it in aux space). A few funny operators are used to support sort-by: for example `*+T` to turn tuple type `T` into a tuple of pointer types, `*?` to avoid loading from extra pointers until the values are needed, and `>*` to compare pointer values.

| Syntax       | Meaning
|--------------|---------
| `>p`         | Get first pointer only from multi-pointer
| `p >*{dn} q` | Compare `p` and `q` by value at first pointer
| `*?p`        | Lazy load at `p`
| `p *? i`     | Lazy load at offset `i` from `p`
| `*+T`        | Tuple of pointer types
