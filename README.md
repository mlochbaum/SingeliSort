# Singeli sort

Algorithms in [Singeli](https://github.com/mlochbaum/Singeli), aiming for high performance and broad adaptivity for sorting CPU-native types (integers and floats). A secondary goal is a well-commented and readable codebase that explains what various methods are good for and how they're implemented to use hardware to its full potential. Will probably end up using SIMD if available to speed up a few things, but this is primarily intended to be a portable rather than SIMD-first sort.

Compile with:

    $ singeli src/sort.singeli -o sort.c && gcc -O3 bench.c

There's also a pre-compiled copy of sort.c in `compiled/sort.c`. It may not always be up to date.

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

- In-place partitioning with [pdqsort](https://github.com/orlp/pdqsort). Slower than crumsort but it does adapt to mostly-sorted data well. Kills patterns.
- Other glidesort methods: interleaved merges and bidirectional partitioning, small-array sort. These have not yet been demonstrated to improve performance relative to fluxsort, and there are indications that they slow things down on older processors in addition to bumping up code size. I'll wait for the paper explaining choices made before looking into them further.
