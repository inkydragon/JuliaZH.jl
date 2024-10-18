# [缺失值](@id missing)

<<<<<<< HEAD
Julia 支持表示统计意义上的缺失值，即某个变量在观察中没有可用值，但在理论上存在有效值的情况。缺失值由 [`missing`](@ref) 对象表示，该对象是 [`Missing`](@ref) 类型的唯一实例。`missing` 等价于 [SQL 中的 `NULL`](https://en.wikipedia.org/wiki/NULL_(SQL)) 以及 [R 中的 `NA`](https://cran.r-project.org/doc/manuals/r-release/R-lang.html#NA-handling)，并在大多数情况下表现得与它们一样。
=======
Julia provides support for representing missing values in the statistical sense.
This is for situations where no value is available for a variable in an observation,
but a valid value theoretically exists.
Missing values are represented via the [`missing`](@ref) object, which is the
singleton instance of the type [`Missing`](@ref). `missing` is equivalent to
[`NULL` in SQL](https://en.wikipedia.org/wiki/NULL_(SQL)) and
[`NA` in R](https://cran.r-project.org/doc/manuals/r-release/R-lang.html#NA-handling),
and behaves like them in most situations.
>>>>>>> cyhan/en-v1.10

## 缺失值的传播

<<<<<<< HEAD
`missing`  值会自动在标准数学运算符和函数中*传播*。对于这类函数，其某个运算对象的值的不确定性会导致其结果的不确定性。在应用中，上述情形意味着若在数学操作中包括 `missing`  值，其结果也常常返回 `missing` 值。
=======
`missing` values *propagate* automatically when passed to standard mathematical
operators and functions.
For these functions, uncertainty about the value of one of the operands
induces uncertainty about the result. In practice, this means a math operation
involving a `missing` value generally returns `missing`:
>>>>>>> cyhan/en-v1.10
```jldoctest
julia> missing + 1
missing

julia> "a" * missing
missing

julia> abs(missing)
missing
```

<<<<<<< HEAD
由于`missing` 是 Julia 中的正常对象，此传播规则仅在可实现该对象的函数中应用。这可通过定义包含 `Missing` 类的实参的特定方法，或是简单地让函数可接受此类实参，并将该它们传入已具备传播规则的函数（如标准数学运算符）中实现。在包中定义新传播规则时，应考虑缺失值的传播是否具有实际意义，并在传播有意义时定义合适的方法。在某个不包含接受 `Missing` 类实参方法的函数中传递缺失值，则抛出 [`MethodError`](@ref)的报错，正如其它类型一样。
=======
Since `missing` is a normal Julia object, this propagation rule only works
for functions which have opted in to implement this behavior. This can be
achieved by:
 - adding a specific method defined for arguments of type `Missing`,
 - accepting arguments of this type, and passing them to functions
   which propagate them (like standard math operators).
Packages should consider
whether it makes sense to propagate missing values when defining new functions,
and define methods appropriately if this is the case. Passing a `missing` value
to a function which does not have a method accepting arguments of type `Missing`
throws a [`MethodError`](@ref), just like for any other type.
>>>>>>> cyhan/en-v1.10

若希望函数不传播缺失值，可将其按照 [Missings.jl](https://github.com/JuliaData/Missings.jl) 库中的 `passmissing` 函数封装起来。例如，将 `f(x)` 封装为 `passmissing(f)(x)`。

## 相等和比较运算符

<<<<<<< HEAD
标准相等和比较运算符遵循上面给出的传播规则：如果任何操作数是 `missing`，那么结果是 `missing`。这是一些例子
=======
Standard equality and comparison operators follow the propagation rule presented
above: if any of the operands is `missing`, the result is `missing`.
Here are a few examples:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> missing == 1
missing

julia> missing == missing
missing

julia> missing < 1
missing

julia> 2 >= missing
missing
```

特别要注意，`missing == missing` 返回 `missing`，所以 `==` 不能用于测试值是否为缺失值。要测试 `x` 是否为 `missing`，请用 [`ismissing(x)`](@ref)。

<<<<<<< HEAD
特殊的比较运算符 [`isequal`](@ref) 和 [`===`](@ref) 是传播规则的例外：它们总返回一个 `Bool` 值，即使存在 `missing` 值，并认为 `missing` 与 `missing` 相等且其与任何其它值不同。因此，它们可用于测试某个值是否为 `missing`。
=======
Special comparison operators [`isequal`](@ref) and [`===`](@ref) are exceptions
to the propagation rule. They will always return a `Bool` value, even in the presence
of `missing` values, considering `missing` as equal to `missing` and as different
from any other value. They can therefore be used to test whether a value is `missing`:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> missing === 1
false

julia> isequal(missing, 1)
false

julia> missing === missing
true

julia> isequal(missing, missing)
true
```

<<<<<<< HEAD
[`isless`](@ref) 运算符是另一个例外：`missing` 被认为比任何其它值大。此运算符被用于 [`sort`](@ref)，因此 `missing` 值被放置在所有其它值之后。
=======
The [`isless`](@ref) operator is another exception: `missing` is considered
as greater than any other value. This operator is used by [`sort!`](@ref),
which therefore places `missing` values after all other values:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> isless(1, missing)
true

julia> isless(missing, Inf)
false

julia> isless(missing, missing)
false
```

## 逻辑运算符

<<<<<<< HEAD
逻辑（或布尔）运算符 [`|`](@ref)、[`&`](@ref) 和 [`xor`](@ref) 是另一种特殊情况，因为它们只有在逻辑上是必需的时传递 `missing` 值。对于这些运算符来说，结果是否不确定取决于具体操作，其遵循[*三值逻辑*](https://en.wikipedia.org/wiki/Three-valued_logic)的既定规则，这些规则也由 SQL 中的 `NULL` 以及 R 中的 `NA` 实现。这个抽象的定义实际上对应于一系列相对自然的行为，这最好通过具体的例子来解释。

让我们用逻辑「或」运算符 [`|`](@ref) 来说明这个原理。按照布尔逻辑的规则，如果其中一个操作数是 `true`，则另一个操作数对结果没影响，结果总是 `true`。
=======
Logical (or boolean) operators [`|`](@ref), [`&`](@ref) and [`xor`](@ref) are
another special case since they only propagate `missing` values when it is logically
required. For these operators, whether or not the result is uncertain, depends
on the particular operation. This follows the well-established rules of
[*three-valued logic*](https://en.wikipedia.org/wiki/Three-valued_logic) which are
implemented by e.g. `NULL` in SQL and `NA` in R. This abstract definition
corresponds to a relatively natural behavior which is best explained
via concrete examples.

Let us illustrate this principle with the logical "or" operator [`|`](@ref).
Following the rules of boolean logic, if one of the operands is `true`,
the value of the other operand does not have an influence on the result,
which will always be `true`:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> true | true
true

julia> true | false
true

julia> false | true
true
```

<<<<<<< HEAD
基于观察，我们可以得出结论，如果其中一个操作数是 `true` 而另一个是 `missing`，我们知道结果为 `true`，尽管另一个参数的实际值存在不确定性。如果我们能观察到第二个操作数的实际值，那么它只能是 `true` 或 `false`，在两种情况下结果都是 `true`。因此，在这种特殊情况下，值的缺失不会传播
=======
Based on this observation, we can conclude if one of the operands is `true`
and the other `missing`, we know that the result is `true` in spite of the
uncertainty about the actual value of one of the operands. If we had
been able to observe the actual value of the second operand, it could only be
`true` or `false`, and in both cases the result would be `true`. Therefore,
in this particular case, missingness does *not* propagate:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> true | missing
true

julia> missing | true
true
```

<<<<<<< HEAD
相反地，如果其中一个操作数是 `false`，结果可能是 `true` 或 `false`，这取决于另一个操作数的值。因此，如果一个操作数是 `missing`，那么结果也是 `missing`。
=======
On the contrary, if one of the operands is `false`, the result could be either
`true` or `false` depending on the value of the other operand. Therefore,
if that operand is `missing`, the result has to be `missing` too:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> false | true
true

julia> true | false
true

julia> false | false
false

julia> false | missing
missing

julia> missing | false
missing
```

<<<<<<< HEAD
逻辑「且」运算符 [`&`](@ref) 的行为与 `|` 运算符相似，区别在于当其中一个操作数为 `false` 时，值的缺失不会传播。例如，当第一个操作数是 `false` 时
=======
The behavior of the logical "and" operator [`&`](@ref) is similar to that of the
`|` operator, with the difference that missingness does not propagate when
one of the operands is `false`. For example, when that is the case of the first
operand:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> false & false
false

julia> false & true
false

julia> false & missing
false
```

<<<<<<< HEAD
另一方面，当其中一个操作数为 `true` 时，值的缺失会传播，例如，当第一个操作数是 `true` 时
=======
On the other hand, missingness propagates when one of the operands is `true`,
for example the first one:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> true & true
true

julia> true & false
false

julia> true & missing
missing
```

<<<<<<< HEAD
最后，逻辑「异或」运算符 [`xor`](@ref) 总传播 `missing` 值，因为两个操作数都总是对结果产生影响。还要注意，否定运算符 [`!`](@ref) 在操作数是 `missing` 时返回 `missing`，这就像其它一元运算符。
=======
Finally, the "exclusive or" logical operator [`xor`](@ref) always propagates
`missing` values, since both operands always have an effect on the result.
Also note that the negation operator [`!`](@ref) returns `missing` when the
operand is `missing`, just like other unary operators.
>>>>>>> cyhan/en-v1.10

## 流程控制和短路运算符

<<<<<<< HEAD
流程控制操作符，包括 [`if`](@ref)、[`while`](@ref) 和[三元运算符](@ref man-conditional-evaluation) `x ? y : z`，不允许缺失值。这是因为如果我们能够观察实际值，它是 `true` 还是 `false` 是不确定的，这意味着我们不知道程序应该如何运行。一旦在以下上下文中遇到 `missing` 值，就会抛出 [`TypeError`](@ref)
=======
Control flow operators including [`if`](@ref), [`while`](@ref) and the
[ternary operator](@ref man-conditional-evaluation) `x ? y : z`
do not allow for missing values. This is because of the uncertainty about whether
the actual value would be `true` or `false` if we could observe it.
This implies we do not know how the program should behave. In this case, a
[`TypeError`](@ref) is thrown as soon as a `missing` value is encountered in this context:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> if missing
           println("here")
       end
ERROR: TypeError: non-boolean (Missing) used in boolean context
```

<<<<<<< HEAD
出于同样的原因，并与上面给出的逻辑运算符相反，短路布尔运算符 [`&&`](@ref) 和 [`||`](@ref) 在当前操作数的值决定下一个操作数是否求值时不允许 `missing` 值。例如
=======
For the same reason, contrary to logical operators presented above,
the short-circuiting boolean operators [`&&`](@ref) and [`||`](@ref) do not
allow for `missing` values in situations where the value of the operand
determines whether the next operand is evaluated or not. For example:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> missing || false
ERROR: TypeError: non-boolean (Missing) used in boolean context

julia> missing && false
ERROR: TypeError: non-boolean (Missing) used in boolean context

julia> true && missing && false
ERROR: TypeError: non-boolean (Missing) used in boolean context
```

<<<<<<< HEAD
另一方面，如果无需 `missing` 值即可确定结果，则不会引发错误。代码在对 `missing` 操作数求值前短路，以及 `missing` 是最后一个操作数都是这种情况。
=======
In contrast, there is no error thrown when the result can be determined without
the `missing` values. This is the case when the code short-circuits
before evaluating the `missing` operand, and when the `missing` operand is the
last one:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> true && missing
missing

julia> false && missing
false
```

## 包含缺失值的数组

<<<<<<< HEAD
包含缺失值的数组的创建就像其它数组
=======
Arrays containing missing values can be created like other arrays:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> [1, missing]
2-element Vector{Union{Missing, Int64}}:
 1
  missing
```

<<<<<<< HEAD
如此示例所示，此类数组的元素类型为 `Union{Missing, T}`，其中 `T` 为非缺失值的类型。这简单地反映了以下事实：数组条目可以具有类型 `T`（此处为 `Int64`）或类型 `Missing`。此类数组使用高效的内存存储，其等价于一个 `Array{T}` 和一个 `Array{UInt8}` 的组合，前者保存实际值，后者表示条目类型（即它是 `Missing` 还是 `T`）。

允许缺失值的数组可以使用标准语法构造。使用 `Array{Union{Missing, T}}(missing, dims)` 来创建填充缺失值的数组：
=======
As this example shows, the element type of such arrays is `Union{Missing, T}`,
with `T` the type of the non-missing values. This reflects the fact that
array entries can be either of type `T` (here, `Int64`) or of type `Missing`.
This kind of array uses an efficient memory storage equivalent to an `Array{T}`
holding the actual values combined with an `Array{UInt8}` indicating the type
of the entry (i.e. whether it is `Missing` or `T`).

Arrays allowing for missing values can be constructed with the standard syntax.
Use `Array{Union{Missing, T}}(missing, dims)` to create arrays filled with
missing values:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> Array{Union{Missing, String}}(missing, 2, 3)
2×3 Matrix{Union{Missing, String}}:
 missing  missing  missing
 missing  missing  missing
```

!!! note
    使用 `undef` 或 `similar` 目前可能会给出一个填充有 `missing` 的数组，但这不是获得这样一个数组的正确方法。 请使用如上所示的 `missing` 构造函数。

<<<<<<< HEAD
允许但不包含 `missing` 值的数组可使用 [`convert`](@ref) 转换回不允许缺失值的数组。如果该数组包含 `missing` 值，在类型转换时会抛出 `MethodError`
=======
An array with element type allowing `missing` entries (e.g. `Vector{Union{Missing, T}}`)
which does not contain any `missing` entries can be converted to an array type that does
not allow for `missing` entries (e.g. `Vector{T}`) using
[`convert`](@ref). If the array contains `missing` values, a `MethodError` is thrown
during conversion:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> x = Union{Missing, String}["a", "b"]
2-element Vector{Union{Missing, String}}:
 "a"
 "b"

julia> convert(Array{String}, x)
2-element Vector{String}:
 "a"
 "b"

julia> y = Union{Missing, String}[missing, "b"]
2-element Vector{Union{Missing, String}}:
 missing
 "b"

julia> convert(Array{String}, y)
ERROR: MethodError: Cannot `convert` an object of type Missing to an object of type String
```
<<<<<<< HEAD
## 跳过缺失值

由于 `missing` 会随着标准数学运算符传播，归约函数会在调用的数组包含缺失值时返回 `missing`
=======

## Skipping Missing Values

Since `missing` values propagate with standard mathematical operators, reduction
functions return `missing` when called on arrays which contain missing values:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> sum([1, missing])
missing
```

<<<<<<< HEAD
在这种情况下，使用 [`skipmissing`](@ref) 即可跳过缺失值
=======
In this situation, use the [`skipmissing`](@ref) function to skip missing values:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> sum(skipmissing([1, missing]))
1
```

<<<<<<< HEAD
此函数方便地返回一个可高效滤除 `missing` 值的迭代器。因此，它可应用于所有支持迭代器的函数 
```jldoctest skipmissing; setup = :(using Statistics)
=======
This convenience function returns an iterator which filters out `missing` values
efficiently. It can therefore be used with any function which supports iterators:

```jldoctest skipmissing
>>>>>>> cyhan/en-v1.10
julia> x = skipmissing([3, missing, 2, 1])
skipmissing(Union{Missing, Int64}[3, missing, 2, 1])

julia> maximum(x)
3

julia> sum(x)
6

julia> mapreduce(sqrt, +, x)
4.146264369941973

```

<<<<<<< HEAD
通过在某数组中调用 `skipmissing` 生成的对象能以其在所属数组中的位置进行索引。对应缺失值的指标并不有效，若尝试使用之会丢出报错（它们在 `keys` 和 `eachindex` 中同样是被跳过的）。
=======
Objects created by calling `skipmissing` on an array can be indexed using indices
from the parent array. Indices corresponding to missing values are not valid for
these objects, and an error is thrown when trying to use them (they are also skipped
by `keys` and `eachindex`):

>>>>>>> cyhan/en-v1.10
```jldoctest skipmissing
julia> x[1]
3

julia> x[2]
ERROR: MissingException: the value at index (2,) is missing
[...]
```

<<<<<<< HEAD
这允许对索引进行操作的函数与`skipmissing`结合使用。搜索和查找函数尤其如此，它们返回对`skipmissing` 函数返回的对象有效的索引，这些索引也是*在父数组中*匹配条目的索引。
```jldoctest skipmissin
=======
This allows functions which operate on indices to work in combination with `skipmissing`.
This is notably the case for search and find functions. These functions return indices
valid for the object returned by `skipmissing`, and are also the indices of the
matching entries *in the parent array*:

```jldoctest skipmissing
>>>>>>> cyhan/en-v1.10
julia> findall(==(1), x)
1-element Vector{Int64}:
 4

julia> findfirst(!iszero, x)
1

julia> argmax(x)
1
```

<<<<<<< HEAD
使用 [`collect`](@ref) 提取非 `missing` 值并将它们存储在一个数组里
=======
Use [`collect`](@ref) to extract non-`missing` values and store them in an array:

>>>>>>> cyhan/en-v1.10
```jldoctest skipmissing
julia> collect(x)
3-element Vector{Int64}:
 3
 2
 1
```

## 数组上的逻辑运算

<<<<<<< HEAD
上面描述的逻辑运算符的三值逻辑也适用于针对数组的函数。因此，使用 [`==`](@ref) 运算符的数组相等性测试中，若在未知 `missing` 条目实际值时无法确定结果，就返回 `missing`。在实际应用中意味着，在待比较数组中所有非缺失值都相等，且某个或全部数组包含缺失值（也许在不同位置）时会返回 `missing`。
=======
The three-valued logic described above for logical operators is also used
by logical functions applied to arrays. Thus, array equality tests using
the [`==`](@ref) operator return `missing` whenever the result cannot be
determined without knowing the actual value of the `missing` entry. In practice,
this means `missing` is returned if all non-missing values of the compared
arrays are equal, but one or both arrays contain missing values (possibly at
different positions):

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> [1, missing] == [2, missing]
false

julia> [1, missing] == [1, missing]
missing

julia> [1, 2, missing] == [1, missing, 2]
missing
```

<<<<<<< HEAD
对于单个值，[`isequal`](@ref) 会将 `missing` 值视为与其它 `missing` 值相等但与非缺失值不同。
=======
As for single values, use [`isequal`](@ref) to treat `missing` values as equal
to other `missing` values, but different from non-missing values:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> isequal([1, missing], [1, missing])
true

julia> isequal([1, 2, missing], [1, missing, 2])
false
```

<<<<<<< HEAD
函数 [`any`](@ref) 和 [`all`](@ref) 遵循三值逻辑的规则，会在结果无法被确定时返回 `missing`。
=======
Functions [`any`](@ref) and [`all`](@ref) also follow the rules of
three-valued logic. Thus, returning `missing` when the result cannot be determined:

>>>>>>> cyhan/en-v1.10
```jldoctest
julia> all([true, missing])
missing

julia> all([false, missing])
false

julia> any([true, missing])
true

julia> any([false, missing])
missing
```
