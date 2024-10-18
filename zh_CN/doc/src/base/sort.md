# 排序及相关函数

<<<<<<< HEAD
Julia 拥有为数众多的灵活的 API，用于对已经排序的值数组进行排序和交互。默认情况下，Julia 会选择合理的算法并按标准升序进行排序：
=======
Julia has an extensive, flexible API for sorting and interacting with already-sorted arrays
of values. By default, Julia picks reasonable algorithms and sorts in ascending order:
>>>>>>> cyhan/en-v1.10

```jldoctest
julia> sort([2,3,1])
3-element Vector{Int64}:
 1
 2
 3
```

<<<<<<< HEAD
你同样可以轻松实现逆序排序：
=======
You can sort in reverse order as well:
>>>>>>> cyhan/en-v1.10

```jldoctest
julia> sort([2,3,1], rev=true)
3-element Vector{Int64}:
 3
 2
 1
```

<<<<<<< HEAD
对数组进行 in-place 排序时，要使用 `!` 版的排序函数：
=======
`sort` constructs a sorted copy leaving its input unchanged. Use the "bang" version of
the sort function to mutate an existing array:
>>>>>>> cyhan/en-v1.10

```jldoctest
julia> a = [2,3,1];

julia> sort!(a);

julia> a
3-element Vector{Int64}:
 1
 2
 3
```

<<<<<<< HEAD
你可以计算用于排列的索引，而不是直接对数组进行排序：
=======
Instead of directly sorting an array, you can compute a permutation of the array's
indices that puts the array into sorted order:
>>>>>>> cyhan/en-v1.10

```julia-repl
julia> v = randn(5)
5-element Array{Float64,1}:
  0.297288
  0.382396
 -0.597634
 -0.0104452
 -0.839027

julia> p = sortperm(v)
5-element Array{Int64,1}:
 5
 3
 4
 1
 2

julia> v[p]
5-element Array{Float64,1}:
 -0.839027
 -0.597634
 -0.0104452
  0.297288
  0.382396
```

<<<<<<< HEAD
数组可以根据对其值任意的转换结果来进行排序；
=======
Arrays can be sorted according to an arbitrary transformation of their values:
>>>>>>> cyhan/en-v1.10

```julia-repl
julia> sort(v, by=abs)
5-element Array{Float64,1}:
 -0.0104452
  0.297288
  0.382396
 -0.597634
 -0.839027
```

或者通过转换来进行逆序排序

```julia-repl
julia> sort(v, by=abs, rev=true)
5-element Array{Float64,1}:
 -0.839027
 -0.597634
  0.382396
  0.297288
 -0.0104452
```

如有必要，可以选择排序算法：

```julia-repl
julia> sort(v, alg=InsertionSort)
5-element Array{Float64,1}:
 -0.839027
 -0.597634
 -0.0104452
  0.297288
  0.382396
```

<<<<<<< HEAD
所有与排序和顺序相关的函数依赖于“小于”关系，该关系定义了要操纵的值的总顺序。默认情况下会调用 `isless` 函数，但可以通过 `lt` 关键字指定关系。
=======
All the sorting and order related functions rely on a "less than" relation defining a
[strict weak order](https://en.wikipedia.org/wiki/Weak_ordering#Strict_weak_orderings)
on the values to be manipulated. The `isless` function is invoked by default, but the
relation can be specified via the `lt` keyword, a function that takes two array elements
and returns `true` if and only if the first argument is "less than" the second. See
[`sort!`](@ref) and [Alternate Orderings](@ref) for more information.
>>>>>>> cyhan/en-v1.10

## 排序函数

```@docs
Base.sort!
Base.sort
Base.sortperm
Base.InsertionSort
Base.MergeSort
Base.QuickSort
Base.PartialQuickSort
Base.Sort.sortperm!
Base.Sort.sortslices
```

## 排列顺序相关的函数

```@docs
Base.issorted
Base.Sort.searchsorted
Base.Sort.searchsortedfirst
Base.Sort.searchsortedlast
Base.Sort.insorted
Base.Sort.partialsort!
Base.Sort.partialsort
Base.Sort.partialsortperm
Base.Sort.partialsortperm!
```

## 排序算法

<<<<<<< HEAD
目前，Julia Base 中有四种可用的排序算法：
=======
There are currently four sorting algorithms publicly available in base Julia:
>>>>>>> cyhan/en-v1.10

  * [`InsertionSort`](@ref)
  * [`QuickSort`](@ref)
  * [`PartialQuickSort(k)`](@ref)
  * [`MergeSort`](@ref)

<<<<<<< HEAD
`InsertionSort` 是一个在 `QuickSort` 中使用的时间复杂度为 O(n^2) 的稳定的排序算法，它通常在 `n` 比较小的时候才具有较高的效率。

`QuickSort` 是一个内置并且非常快，但是不稳定的时间复杂度为 O(n log n）的排序算法，例如即使数组两个元素相等的，它们排序之后的顺序也可能和在原数组中顺序不一致。`QuickSort` 是内置的包括整数和浮点数在内的数字值的默认排序算法。

`PartialQuickSort(k)` 类似于 `QuickSort`，但是如果 `k` 是一个整数，输出数组只排序到索引 `k`，如果 `k` 是 `OrdinalRange`，则输出数组排在 `k` 范围内。 例如：
=======
By default, the `sort` family of functions uses stable sorting algorithms that are fast
on most inputs. The exact algorithm choice is an implementation detail to allow for
future performance improvements. Currently, a hybrid of `RadixSort`, `ScratchQuickSort`,
`InsertionSort`, and `CountingSort` is used based on input type, size, and composition.
Implementation details are subject to change but currently available in the extended help
of `??Base.DEFAULT_STABLE` and the docstrings of internal sorting algorithms listed there.
>>>>>>> cyhan/en-v1.10

You can explicitly specify your preferred algorithm with the `alg` keyword
(e.g. `sort!(v, alg=PartialQuickSort(10:20))`) or reconfigure the default sorting algorithm
for custom types by adding a specialized method to the `Base.Sort.defalg` function.
For example, [InlineStrings.jl](https://github.com/JuliaStrings/InlineStrings.jl/blob/v1.3.2/src/InlineStrings.jl#L903)
defines the following method:
```julia
Base.Sort.defalg(::AbstractArray{<:Union{SmallInlineStrings, Missing}}) = InlineStringSort
```

<<<<<<< HEAD
`MergeSort` 是一个时间复杂度为 O(n log n) 的稳定但是非 in-place 的算法，它需要一个大小为输入数组一般的临时数组——同时也不像 `QuickSort` 一样快。`MergeSort` 是非数值型数据的默认排序算法。

默认排序算法的选择是基于它们的快速稳定，或者 *appear* 之类的。对于数值类型，实际上选择了 `QuickSort`，因为在这种情况下，它更快，与稳定排序没有区别(除非数组以某种方式记录了突变)

Julia选择默认排序算法的机制是通过 `Base.Sort.defalg` 来实现的，其允许将特定算法注册为特定数组的所有排序函数中的默认值。例如，这有两个默认算法 [`sort.jl`](https://github.com/JuliaLang/julia/blob/master/base/sort.jl):

```julia
defalg(v::AbstractArray) = MergeSort
defalg(v::AbstractArray{<:Number}) = QuickSort
```

对于数值型数组，选择非稳定的默认排序算法的原则是稳定的排序算法没有必要的（例如：但两个值相比较时相等且不可区分时）。

## Alternate orderings

By default, `sort` and related functions use [`isless`](@ref) to compare two
elements in order to determine which should come first. The
[`Base.Order.Ordering`](@ref) abstract type provides a mechanism for defining
alternate orderings on the same set of elements. Instances of `Ordering` define
a [total order](https://en.wikipedia.org/wiki/Total_order) on a set of elements,
so that for any elements `a`, `b`, `c` the following hold:

* Exactly one of the following is true: `a` is less than `b`, `b` is less than
  `a`, or `a` and `b` are equal (according to [`isequal`](@ref)).
* The relation is transitive - if `a` is less than `b` and `b` is less than `c`
  then `a` is less than `c`.

The [`Base.Order.lt`](@ref) function works as a generalization of `isless` to
test whether `a` is less than `b` according to a given order.
=======
!!! compat "Julia 1.9"
    The default sorting algorithm (returned by `Base.Sort.defalg`) is guaranteed to
    be stable since Julia 1.9. Previous versions had unstable edge cases when
    sorting numeric arrays.

## Alternate Orderings

By default, `sort`, `searchsorted`, and related functions use [`isless`](@ref) to compare
two elements in order to determine which should come first. The
[`Base.Order.Ordering`](@ref) abstract type provides a mechanism for defining alternate
orderings on the same set of elements: when calling a sorting function like
`sort!`, an instance of `Ordering` can be provided with the keyword argument `order`.

Instances of `Ordering` define an order through the [`Base.Order.lt`](@ref)
function, which works as a generalization of `isless`.
This function's behavior on custom `Ordering`s must satisfy all the conditions of a
[strict weak order](https://en.wikipedia.org/wiki/Weak_ordering#Strict_weak_orderings).
See [`sort!`](@ref) for details and examples of valid and invalid `lt` functions.
>>>>>>> cyhan/en-v1.10

```@docs
Base.Order.Ordering
Base.Order.lt
Base.Order.ord
Base.Order.Forward
Base.Order.ReverseOrdering
Base.Order.Reverse
Base.Order.By
Base.Order.Lt
Base.Order.Perm
```
