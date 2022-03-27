# Singeli sort

Algorithms in [Singeli](https://github.com/mlochbaum/Singeli), aiming for high performance and broad adaptivity for sorting CPU-native types (integers and floats). A secondary goal is a well-commented and readable codebase that explains what various methods are good for and how they're implemented to use hardware to its full potential.

Compile with:

    $ singeli src/sort.singeli -o sort.c && gcc -O3 bench.c

Singeli sort will hybridize the following algorithms. All the base mechanisms except quadsort merging have now been implemented with performance matching the C versions, so most of the remaining work is in the control code that switches between algorithms (mostly pivot candidate handling).

- Quicksort partitioning: [fluxsort](https://github.com/scandum/fluxsort), [crumsort](https://github.com/scandum/crumsort), and maybe [pdqsort]( https://github.com/orlp/pdqsort)
- Merging: [quadsort](https://github.com/scandum/quadsort)
- Small-array sorting: quadsort; maybe [tinysort](https://github.com/scandum/tinysort)
- Radix sort: mostly like [ska_sort_copy](https://github.com/skarupke/ska_sort)
- Counting sort: see [section](https://github.com/mlochbaum/rhsort#counting-sort) in rhsort
- [Robin Hood](https://github.com/mlochbaum/rhsort) sort
