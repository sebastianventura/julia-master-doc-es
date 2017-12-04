# [Arrays Multi-dimensionales](@id man-multi-dim-arrays)

Julia, como la mayoría de los lenguajes informáticos técnicos, proporciona una implementación de los arrays de primera clase. La mayoría de los lenguajes informáticos técnicos prestan mucha atención a su implementación de arrays a expensas de otros contenedores. Julia no trata los arrays de manera especial. La biblioteca de arrays se ha implementado casi completamente en el propio lenguaje Julia, y deriva su rendimiento del compilador, al igual que cualquier otro código escrito en Julia. Como tal, es también posible definir tipos de arrays personalizados heredando de AbstractArray. Consulte la [sección de manual en la interfaz AbstractArray](@ref man-interface-array) para ms detalles sobre implementar un tipo array personalizado.

Un array es una colección de objetos almacenados en una cuadrícula multidimensional. En el caso más general, un array puede contener objetos de tipo `Any`. Para la mayoría de los propósitos computacionales, los arrays deben contener objetos de un tipo más específico, como  [`Float64`](@ref) o [`Int32`](@ref).

En general, a diferencia de muchos otros lenguajes informáticos técnicos, Julia no espera que los programas se escriban en un estilo vectorizado para el rendimiento. El compilador de Julia utiliza la inferencia de tipos y genera código optimizado para la indexación escalar de arrays, permitiendo que los programas se escriban en un estilo que sea conveniente y legible, sin sacrificar el rendimiento y utilizando menos memoria a veces.

En Julia, todos los argumentos a las funciones se pasan por referencia. Algunos lenguajes informáticos técnicos pasan los arrays por valor, y esto es conveniente en muchos casos. En Julia, las modificaciones hechas a los arrays de entrada dentro de una función serán visibles en la función principal. Toda la biblioteca de arrays de Julia garantiza que las entradas no sean modificadas por las funciones de biblioteca. El código de usuario, si necesita mostrar un comportamiento similar, debe tener cuidado de crear una copia de las entradas que puede modificar.

## Arrays

### Funciones Básicas

| Function               | Description                                                                      |
|:---------------------- |:-------------------------------------------------------------------------------- |
| [`eltype(A)`](@ref)    | Tipo de los elementos contenidos en `A`                                          |
| [`length(A)`](@ref)    | Número de elementos en `A`                                                       |
| [`ndims(A)`](@ref)     | Número de dimensiones de `A`                                                     |
| [`size(A)`](@ref)      | Una tupla que contien las dimensiones de `A`                                     |
| [`size(A,n)`](@ref)    | El tamaño de `A` a lo largo de una dimensión particular `n`                      |
| [`indices(A)`](@ref)   | Una tupla que contiene los índices válidos de `A`                                |
| [`indices(A,n)`](@ref) | Un rango expresando los úndices válidos a lo largo de la dimensión `n            |
| [`eachindex(A)`](@ref) | Un iterador eficiente para visitar cada posición en `A`                          |
| [`stride(A,k)`](@ref)  | La zancada (*stride*, distancia de índice lineal entre elementos adyacentes) a lo largo de la dimensión `k`. |
| [`strides(A)`](@ref)   | Una tupla de las zancadas en cada dimensión                                      |

### Construcción e Inicialización

Existen muchas funciones para construir e inicializar matrices. En la siguiente lista de tales funciones, las llamadas con un argumento `dims...` pueden tomar una sola tupla de tamaños de dimensión o una serie de tamaños de dimensión pasados como un número variable de argumentos. Muchas de estas funciones también aceptan un primea entrada `T`, que es el tipo de los elementos del array. Si este tipo es omitido se asumirá como tipo por defecto por defecto [`Float64`](@ref).

| Función                           | Descripción                                                                                                                                                                                                                    |
|:---------------------------------- |:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`Array{T}(dims...)`](@ref)        | an uninitialized dense [`Array`](@ref)                                                                                                                                                                                                       |
| [`zeros(T, dims...)`](@ref)        | an `Array` of all zeros                                                                                                                                                                                                                      |
| [`zeros(A)`](@ref)                 | an array of all zeros with the same type, element type and shape as `A`                                                                                                                                                                      |
| [`ones(T, dims...)`](@ref)         | an `Array` of all ones                                                                                                                                                                                                                       |
| [`ones(A)`](@ref)                  | an array of all ones with the same type, element type and shape as `A`                                                                                                                                                                       |
| [`trues(dims...)`](@ref)           | a [`BitArray`](@ref) with all values `true`                                                                                                                                                                                                  |
| [`trues(A)`](@ref)                 | a `BitArray` with all values `true` and the same shape as `A`                                                                                                                                                                                |
| [`falses(dims...)`](@ref)          | a `BitArray` with all values `false`                                                                                                                                                                                                         |
| [`falses(A)`](@ref)                | a `BitArray` with all values `false` and the same shape as `A`                                                                                                                                                                               |
| [`reshape(A, dims...)`](@ref)      | an array containing the same data as `A`, but with different dimensions                                                                                                                                                                      |
| [`copy(A)`](@ref)                  | copy `A`                                                                                                                                                                                                                                     |
| [`deepcopy(A)`](@ref)              | copy `A`, recursively copying its elements                                                                                                                                                                                                   |
| [`similar(A, T, dims...)`](@ref)   | an uninitialized array of the same type as `A` (dense, sparse, etc.), but with the specified element type and dimensions. The second and third arguments are both optional, defaulting to the element type and dimensions of `A` if omitted. |
| [`reinterpret(T, A)`](@ref)        | an array with the same binary data as `A`, but with element type `T`                                                                                                                                                                         |
| [`rand(T, dims...)`](@ref)         | an `Array` with random, iid [^1] and uniformly distributed values in the half-open interval ``[0, 1)``                                                                                                                                       |
| [`randn(T, dims...)`](@ref)        | an `Array` with random, iid and standard normally distributed values                                                                                                                                                                         |
| [`eye(T, n)`](@ref)                | `n`-by-`n` identity matrix                                                                                                                                                                                                                   |
| [`eye(T, m, n)`](@ref)             | `m`-by-`n` identity matrix                                                                                                                                                                                                                   |
| [`linspace(start, stop, n)`](@ref) | range of `n` linearly spaced elements from `start` to `stop`                                                                                                                                                                                 |
| [`fill!(A, x)`](@ref)              | fill the array `A` with the value `x`                                                                                                                                                                                                        |
| [`fill(x, dims...)`](@ref)         | an `Array` filled with the value `x`                                                                                                                                                                                                         |

[^1]: *iid*, independently and identically distributed.

The syntax `[A, B, C, ...]` constructs a 1-d array (vector) of its arguments. If all
arguments have a common [promotion type](@ref conversion-and-promotion) then they get
converted to that type using `convert()`.

### Concatenación

Los arrays pueden ser construídos y también concatenados usando las siguientes funciones:

| Function               | Description                                          |
|:---------------------- |:---------------------------------------------------- |
| [`cat(k, A...)`](@ref) | concatena n-d arrays a lo largo de la dimensión `k`  |
| [`vcat(A...)`](@ref)   | abreviatura para `cat(1, A...)`                      |
| [`hcat(A...)`](@ref)   | abreviatura para `cat(2, A...)`                      |

Los valores escalares pasados a estas funciones son tratados como arrays de 1 elemento.

Las funciones de concatenación se usan tan frecuentemente que tiene una sintaxis especial:

| Expression        | Calls             |
|:----------------- |:----------------- |
| `[A; B; C; ...]`  | [`vcat()`](@ref)  |
| `[A B C ...]`     | [`hcat()`](@ref)  |
| `[A B; C D; ...]` | [`hvcat()`](@ref) |

`hvcat()` concatena tanto en la dimensión 1 (con puntos y coma) como en la dos (con espacios).

### Inicializadores tipados de arrays

Se puede construir una matriz con un tipo de elemento específico utilizando la sintaxis `T[A, B, C, ...]`. Esto construirá un array 1-d con el tipo de elemento `T`, inicializado para contener los elementos `A`, `B`, `C`, etc. Por ejemplo `Any [x, y, z]` construye un array heterogéneo que puede contener cualquier valor.

La sintaxis de concatenación puede ser prefijada de forma similar con un tipo para especificar el tipo de elemento del resultado.

```jldoctest
julia> [[1 2] [3 4]]
1×4 Array{Int64,2}:
 1  2  3  4

julia> Int8[[1 2] [3 4]]
1×4 Array{Int8,2}:
 1  2  3  4
```

### Comprensiones

Las comprensiones proporcionan una forma general y potente de construir arrays. La sintaxis de comprensión es similar a la notación de construcción de conjuntos en matemáticas:

```
A = [ F(x,y,...) for x=rx, y=ry, ... ]
```

El significado de esta forma es que `F(x, y, ...)` se evalúa para las variables x, y, etc. tomando cada valor en su lista de valores dada. Los valores se pueden especificar como cualquier objeto iterable, pero comúnmente serán rangos como `1:n` o `2:(n-1)`, o arrays de valores explícitos como `[1.2, 3.4, 5.7]`. El resultado es una matriz N-d densa con dimensiones que son la concatenación de las dimensiones de las variables rango `rx`, `ry`, etc. y cada evaluación `F (x, y, ...)` devuelve un escalar.

El siguiente ejemplo calcula un promedio ponderado del elemento actual y su vecino izquierdo y derecho a lo largo de una cuadrícula de 1-d. :

```julia-repl
julia> x = rand(8)
8-element Array{Float64,1}:
 0.843025
 0.869052
 0.365105
 0.699456
 0.977653
 0.994953
 0.41084
 0.809411

julia> [ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]
6-element Array{Float64,1}:
 0.736559
 0.57468
 0.685417
 0.912429
 0.8446
 0.656511
```

El tipo del array resultante depende de los tipos de los elementos calculados. Para controlar el tipo explícitamente, un tipo puede ser precedido a la comprensión. Por ejemplo, podríamos haber solicitado el resultado en precisión simple escribiendo:

```julia
Float32[ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]
```

### Expresiones Generador

Las comprensiones también se pueden escribir sin los corchetes adjuntos, produciendo un objeto conocido como generador. Este objeto puede iterarse para producir valores bajo demanda, en lugar de asignar una matriz y almacenarlos de antemano (véase [Iteración](@ref)). Por ejemplo, la siguiente expresión suma una serie sin asignar memoria:

```jldoctest
julia> sum(1/n^2 for n=1:1000)
1.6439345666815615
```

Cuando se escribe una expresión generador con múltiples dimensiones dentro de una lista de argumentos, se necesitan paréntesis para separar el generador de argumentos posteriores:

```julia-repl
julia> map(tuple, 1/(i+j) for i=1:2, j=1:2, [1:4;])
ERROR: syntax: invalid iteration specification
```

Todas las expresiones separadas por comas después de se interpretan como rangos. Añadir paréntesis nos permite añadir un tercer argumento a `map`:

```jldoctest
julia> map(tuple, (1/(i+j) for i=1:2, j=1:2), [1 3; 2 4])
2×2 Array{Tuple{Float64,Int64},2}:
 (0.5, 1)       (0.333333, 3)
 (0.333333, 2)  (0.25, 4)
```

Los rangos en generadores y comprensiones pueden depender de rangos anteriores escribiendo varias palabras clave `for`:

```jldoctest
julia> [(i,j) for i=1:3 for j=1:i]
6-element Array{Tuple{Int64,Int64},1}:
 (1, 1)
 (2, 1)
 (2, 2)
 (3, 1)
 (3, 2)
 (3, 3)
```

En estos casos, el resultado es siempre 1-d.

Los valores generados se pueden filtrar usando la palabra clave if:

```jldoctest
julia> [(i,j) for i=1:3 for j=1:i if i+j == 4]
2-element Array{Tuple{Int64,Int64},1}:
 (2, 2)
 (3, 1)
```

### [Indexación](@id man-array-indexing)

La sintaxis general para indexar en un array n-dimensional `A` es:

```
X = A[I_1, I_2, ..., I_n]
```

donde cada `I_k` puede ser un entero escalar, un array de enteros o cualquier otro [índice soportado](@ref man-supported-index-types). Esto incluye [`Colon`](@ref) (`:`) para seleccionar todos los índices dentro de la dimensión completa, rangos de la forma `a:c` o `a:b:c` para seleccionar subsecciones contiguas o con salto, y arrays de booleans para seleccionar elementos en sus índices `true`.

Si todos los índices fueran escalares, entonces el resultado `X` es un solo elemento del array `A`. De lo contrario, `X` es un array con el mismo número de dimensiones que la suma de las dimensionalidades de todos los índices.

Si todos los índices son vectores, por ejemplo, entonces la forma de `X` sería `(length(I_1), length(I_2), ..., length(I_n))`, donde las ubicaciones `(i_1, i_2, ..., i_n)` de `X` contienen el valor `A[I_1[i_1], I_2[i_2], ..., I_n[i_n]]`. Si `I_1` se cambia por un array bidimensional, entonces `X` se vuelve un `n+1`-dimensional array de forma `(size(I_1, 1), size(I_1, 2), length(I_2), ..., length(I_n))`. La matriz añade una dimensión. La ubicación `(i_1, i_2, i_3, ..., i_{n+1})` contiene el valor en `A[I_1[i_1, i_2], I_2[i_3], ..., I_n[i_{n+1}]]`. Todas las dimensiones indexadas con escalares se eliminan. Por ejemplo, el resultado de `A[2, I, 3]` es un array de tamaño `size(I)`. Su i-ésimo elemento es poblado por `A[2, I[i], 3]`.

Como parte especial de esta sintaxis, se puede usar la palabra clave `end` para representar el último índice de cada dimensión dentro de los corchetes de indexación, según lo determinado por el tamaño del array más interno indexado. La sintaxis de indexación sin la palabra `end` es equivalente a una llamada a `getindex`:

```
X = getindex(A, I_1, I_2, ..., I_n)
```

Ejemplo:

```jldoctest
julia> x = reshape(1:16, 4, 4)
4×4 Base.ReshapedArray{Int64,2,UnitRange{Int64},Tuple{}}:
 1  5   9  13
 2  6  10  14
 3  7  11  15
 4  8  12  16

julia> x[2:3, 2:end-1]
2×2 Array{Int64,2}:
 6  10
 7  11

julia> x[1, [2 3; 4 1]]
2×2 Array{Int64,2}:
  5  9
 13  1
```

Los rangos vacío de la forma `n:n-1` se suelen usar para indicar la localización inter-index entre `n-1` y `n`. Por ejemplo, la función [`searchsorted()`](@ref) usa esta convención para indicar el punto de inserción de un valor no encontrados en un array ordenado:

```jldoctest
julia> a = [1,2,5,6,7];

julia> searchsorted(a, 3)
3:2
```

### Asignación

La sintaxis general para asignar valores en un array `A` es:
```
A[I_1, I_2, ..., I_n] = X
```

donde cada `I_k` puede ser un índice escalar, un array de enteros o cualquier otro [índice soportado](@ref man-supported-index-types). Esto incluye [`Colon`](@ref) (`:`) para seleccionar todos los índices dentro de la dimensión completa, rangos de la forma `a:c` o `a:b:c` para seleccionar subsecciones contiguas o con salto, y arrays de booleans para seleccionar elementos en sus índices `true`.

Si `X` es un array, debe tener el mismo número de elementos que el producto de las longitudes de los índices `prod(length(I_1), length(I_2), ..., length(I_n))`. El valor en la localización `I_1[i_1], I_2[i_2], ..., I_n[i_n]`de `A` es sobreescrito con el valor `X[i_1, i_2, ..., i_n]`. Si `X` no es un array, su valor es escrito a todas las localizaciones referenciadas de `A`.

Justo como en [Indexación](@ref man-array-indexing), la palabra clave `end` puede utilizarse para representar el último índice de cada dimensión dentro de los corchetes de los índices, como queda determinado por el tamaño del array en el que se está siendo asignado. La sintaxis de la asignacin indwexada sin la palabra clave `end` es equivalente a llamar a la función [`setindex!()`](@ref):

```
setindex!(A, X, I_1, I_2, ..., I_n)
```

Ejemplo:

```jldoctest
julia> x = collect(reshape(1:9, 3, 3))
3×3 Array{Int64,2}:
 1  4  7
 2  5  8
 3  6  9

julia> x[1:2, 2:3] = -1
-1

julia> x
3×3 Array{Int64,2}:
 1  -1  -1
 2  -1  -1
 3   6   9
```

### [Supported index types](@id man-supported-index-types)

In the expression `A[I_1, I_2, ..., I_n]`, each `I_k` may be a scalar index, an
array of scalar indices, or an object that represents an array of scalar
indices and can be converted to such by [`to_indices`](@ref):

1. A scalar index. By default this includes:
    * Non-boolean integers
    * `CartesianIndex{N}`s, which behave like an `N`-tuple of integers spanning multiple dimensions (see below for more details)
2. An array of scalar indices. This includes:
    * Vectors and multidimensional arrays of integers
    * Empty arrays like `[]`, which select no elements
    * `Range`s of the form `a:c` or `a:b:c`, which select contiguous or strided subsections from `a` to `c` (inclusive)
    * Any custom array of scalar indices that is a subtype of `AbstractArray`
    * Arrays of `CartesianIndex{N}` (see below for more details)
3. An object that represents an array of scalar indices and can be converted to such by [`to_indices`](@ref). By default this includes:
    * [`Colon()`](@ref) (`:`), which represents all indices within an entire dimension or across the entire array
    * Arrays of booleans, which select elements at their `true` indices (see below for more details)

#### Cartesian indices

The special `CartesianIndex{N}` object represents a scalar index that behaves
like an `N`-tuple of integers spanning multiple dimensions.  For example:

```jldoctest cartesianindex
julia> A = reshape(1:32, 4, 4, 2);

julia> A[3, 2, 1]
7

julia> A[CartesianIndex(3, 2, 1)] == A[3, 2, 1] == 7
true
```

Considered alone, this may seem relatively trivial; `CartesianIndex` simply
gathers multiple integers together into one object that represents a single
multidimensional index. When combined with other indexing forms and iterators
that yield `CartesianIndex`es, however, this can lead directly to very elegant
and efficient code. See [Iteration](@ref) below, and for some more advanced
examples, see [this blog post on multidimensional algorithms and
iteration](https://julialang.org/blog/2016/02/iteration).

Arrays of `CartesianIndex{N}` are also supported. They represent a collection
of scalar indices that each span `N` dimensions, enabling a form of indexing
that is sometimes referred to as pointwise indexing. For example, it enables
accessing the diagonal elements from the first "page" of `A` from above:

```jldoctest cartesianindex
julia> page = A[:,:,1]
4×4 Array{Int64,2}:
 1  5   9  13
 2  6  10  14
 3  7  11  15
 4  8  12  16

julia> page[[CartesianIndex(1,1),
             CartesianIndex(2,2),
             CartesianIndex(3,3),
             CartesianIndex(4,4)]]
4-element Array{Int64,1}:
  1
  6
 11
 16
```

This can be expressed much more simply with [dot broadcasting](@ref man-vectorized)
and by combining it with a normal integer index (instead of extracting the
first `page` from `A` as a separate step). It can even be combined with a `:`
to extract both diagonals from the two pages at the same time:

```jldoctest cartesianindex
julia> A[CartesianIndex.(indices(A, 1), indices(A, 2)), 1]
4-element Array{Int64,1}:
  1
  6
 11
 16

julia> A[CartesianIndex.(indices(A, 1), indices(A, 2)), :]
4×2 Array{Int64,2}:
  1  17
  6  22
 11  27
 16  32
```

!!! warning

    `CartesianIndex` and arrays of `CartesianIndex` are not compatible with the
    `end` keyword to represent the last index of a dimension. Do not use `end`
    in indexing expressions that may contain either `CartesianIndex` or arrays thereof.

#### Logical indexing

Often referred to as logical indexing or indexing with a logical mask, indexing
by a boolean array selects elements at the indices where its values are `true`.
Indexing by a boolean vector `B` is effectively the same as indexing by the
vector of integers that is returned by [`find(B)`](@ref). Similarly, indexing
by a `N`-dimensional boolean array is effectively the same as indexing by the
vector of `CartesianIndex{N}`s where its values are `true`. A logical index
must be a vector of the same length as the dimension it indexes into, or it
must be the only index provided and match the size and dimensionality of the
array it indexes into. It is generally more efficient to use boolean arrays as
indices directly instead of first calling [`find()`](@ref).

```jldoctest
julia> x = reshape(1:16, 4, 4)
4×4 Base.ReshapedArray{Int64,2,UnitRange{Int64},Tuple{}}:
 1  5   9  13
 2  6  10  14
 3  7  11  15
 4  8  12  16

julia> x[[false, true, true, false], :]
2×4 Array{Int64,2}:
 2  6  10  14
 3  7  11  15

julia> mask = map(ispow2, x)
4×4 Array{Bool,2}:
  true  false  false  false
  true  false  false  false
 false  false  false  false
  true   true  false   true

julia> x[mask]
5-element Array{Int64,1}:
  1
  2
  4
  8
 16
```

### Iteration

The recommended ways to iterate over a whole array are

```julia
for a in A
    # Do something with the element a
end

for i in eachindex(A)
    # Do something with i and/or A[i]
end
```

The first construct is used when you need the value, but not index, of each element. In the second
construct, `i` will be an `Int` if `A` is an array type with fast linear indexing; otherwise,
it will be a `CartesianIndex`:

```jldoctest
julia> A = rand(4,3);

julia> B = view(A, 1:3, 2:3);

julia> for i in eachindex(B)
           @show i
       end
i = CartesianIndex{2}((1, 1))
i = CartesianIndex{2}((2, 1))
i = CartesianIndex{2}((3, 1))
i = CartesianIndex{2}((1, 2))
i = CartesianIndex{2}((2, 2))
i = CartesianIndex{2}((3, 2))
```

In contrast with `for i = 1:length(A)`, iterating with `eachindex` provides an efficient way to
iterate over any array type.

### Array traits

If you write a custom [`AbstractArray`](@ref) type, you can specify that it has fast linear indexing using

```julia
Base.IndexStyle(::Type{<:MyArray}) = IndexLinear()
```

This setting will cause `eachindex` iteration over a `MyArray` to use integers. If you don't
specify this trait, the default value `IndexCartesian()` is used.

### Array and Vectorized Operators and Functions

The following operators are supported for arrays:

1. Unary arithmetic -- `-`, `+`
2. Binary arithmetic -- `-`, `+`, `*`, `/`, `\`, `^`
3. Comparison -- `==`, `!=`, `≈` ([`isapprox`](@ref)), `≉`

Most of the binary arithmetic operators listed above also operate elementwise
when one argument is scalar: `-`, `+`, and `*` when either argument is scalar,
and `/` and `\` when the denominator is scalar. For example, `[1, 2] + 3 == [4, 5]`
and `[6, 4] / 2 == [3, 2]`.

Additionally, to enable convenient vectorization of mathematical and other operations,
Julia [provides the dot syntax](@ref man-vectorized) `f.(args...)`, e.g. `sin.(x)`
or `min.(x,y)`, for elementwise operations over arrays or mixtures of arrays and
scalars (a [Broadcasting](@ref) operation); these have the additional advantage of
"fusing" into a single loop when combined with other dot calls, e.g. `sin.(cos.(x))`.

Also, *every* binary operator supports a [dot version](@ref man-dot-operators)
that can be applied to arrays (and combinations of arrays and scalars) in such
[fused broadcasting operations](@ref man-vectorized), e.g. `z .== sin.(x .* y)`.

Note that comparisons such as `==` operate on whole arrays, giving a single boolean
answer. Use dot operators like `.==` for elementwise comparisons. (For comparison
operations like `<`, *only* the elementwise `.<` version is applicable to arrays.)

Also notice the difference between `max.(a,b)`, which `broadcast`s [`max()`](@ref)
elementwise over `a` and `b`, and `maximum(a)`, which finds the largest value within
`a`. The same relationship holds for `min.(a,b)` and `minimum(a)`.

### Broadcasting

It is sometimes useful to perform element-by-element binary operations on arrays of different
sizes, such as adding a vector to each column of a matrix. An inefficient way to do this would
be to replicate the vector to the size of the matrix:

```julia-repl
julia> a = rand(2,1); A = rand(2,3);

julia> repmat(a,1,3)+A
2×3 Array{Float64,2}:
 1.20813  1.82068  1.25387
 1.56851  1.86401  1.67846
```

This is wasteful when dimensions get large, so Julia offers [`broadcast()`](@ref), which expands
singleton dimensions in array arguments to match the corresponding dimension in the other array
without using extra memory, and applies the given function elementwise:

```julia-repl
julia> broadcast(+, a, A)
2×3 Array{Float64,2}:
 1.20813  1.82068  1.25387
 1.56851  1.86401  1.67846

julia> b = rand(1,2)
1×2 Array{Float64,2}:
 0.867535  0.00457906

julia> broadcast(+, a, b)
2×2 Array{Float64,2}:
 1.71056  0.847604
 1.73659  0.873631
```

[Dotted operators](@ref man-dot-operators) such as `.+` and `.*` are equivalent
to `broadcast` calls (except that they fuse, as described below). There is also a
[`broadcast!()`](@ref) function to specify an explicit destination (which can also
be accessed in a fusing fashion by `.=` assignment), and functions [`broadcast_getindex()`](@ref)
and [`broadcast_setindex!()`](@ref) that broadcast the indices before indexing. Moreover, `f.(args...)`
is equivalent to `broadcast(f, args...)`, providing a convenient syntax to broadcast any function
([dot syntax](@ref man-vectorized)). Nested "dot calls" `f.(...)` (including calls to `.+` etcetera)
[automatically fuse](@ref man-dot-operators) into a single `broadcast` call.

Additionally, [`broadcast()`](@ref) is not limited to arrays (see the function documentation),
it also handles tuples and treats any argument that is not an array, tuple or `Ref` (except for `Ptr`) as a "scalar".

```jldoctest
julia> convert.(Float32, [1, 2])
2-element Array{Float32,1}:
 1.0
 2.0

julia> ceil.((UInt8,), [1.2 3.4; 5.6 6.7])
2×2 Array{UInt8,2}:
 0x02  0x04
 0x06  0x07

julia> string.(1:3, ". ", ["First", "Second", "Third"])
3-element Array{String,1}:
 "1. First"
 "2. Second"
 "3. Third"
```

### Implementation

The base array type in Julia is the abstract type [`AbstractArray{T,N}`](@ref). It is parametrized by
the number of dimensions `N` and the element type `T`. [`AbstractVector`](@ref) and [`AbstractMatrix`](@ref) are
aliases for the 1-d and 2-d cases. Operations on `AbstractArray` objects are defined using higher
level operators and functions, in a way that is independent of the underlying storage. These operations
generally work correctly as a fallback for any specific array implementation.

The `AbstractArray` type includes anything vaguely array-like, and implementations of it might
be quite different from conventional arrays. For example, elements might be computed on request
rather than stored. However, any concrete `AbstractArray{T,N}` type should generally implement
at least [`size(A)`](@ref) (returning an `Int` tuple), [`getindex(A,i)`](@ref) and [`getindex(A,i1,...,iN)`](@ref getindex);
mutable arrays should also implement [`setindex!()`](@ref). It is recommended that these operations
have nearly constant time complexity, or technically Õ(1) complexity, as otherwise some array
functions may be unexpectedly slow. Concrete types should also typically provide a [`similar(A,T=eltype(A),dims=size(A))`](@ref)
method, which is used to allocate a similar array for [`copy()`](@ref) and other out-of-place
operations. No matter how an `AbstractArray{T,N}` is represented internally, `T` is the type of
object returned by *integer* indexing (`A[1, ..., 1]`, when `A` is not empty) and `N` should be
the length of the tuple returned by [`size()`](@ref).

`DenseArray` is an abstract subtype of `AbstractArray` intended to include all arrays that are
laid out at regular offsets in memory, and which can therefore be passed to external C and Fortran
functions expecting this memory layout. Subtypes should provide a method [`stride(A,k)`](@ref)
that returns the "stride" of dimension `k`: increasing the index of dimension `k` by `1` should
increase the index `i` of [`getindex(A,i)`](@ref) by [`stride(A,k)`](@ref). If a pointer conversion
method [`Base.unsafe_convert(Ptr{T}, A)`](@ref) is provided, the memory layout should correspond
in the same way to these strides.

The [`Array`](@ref) type is a specific instance of `DenseArray` where elements are stored in column-major
order (see additional notes in [Performance Tips](@ref man-performance-tips)). [`Vector`](@ref) and [`Matrix`](@ref) are aliases for
the 1-d and 2-d cases. Specific operations such as scalar indexing, assignment, and a few other
basic storage-specific operations are all that have to be implemented for [`Array`](@ref), so
that the rest of the array library can be implemented in a generic manner.

`SubArray` is a specialization of `AbstractArray` that performs indexing by reference rather than
by copying. A `SubArray` is created with the [`view()`](@ref) function, which is called the same
way as [`getindex()`](@ref) (with an array and a series of index arguments). The result of [`view()`](@ref)
looks the same as the result of [`getindex()`](@ref), except the data is left in place. [`view()`](@ref)
stores the input index vectors in a `SubArray` object, which can later be used to index the original
array indirectly.  By putting the [`@views`](@ref) macro in front of an expression or
block of code, any `array[...]` slice in that expression will be converted to
create a `SubArray` view instead.

`StridedVector` and `StridedMatrix` are convenient aliases defined to make it possible for Julia
to call a wider range of BLAS and LAPACK functions by passing them either [`Array`](@ref) or
`SubArray` objects, and thus saving inefficiencies from memory allocation and copying.

The following example computes the QR decomposition of a small section of a larger array, without
creating any temporaries, and by calling the appropriate LAPACK function with the right leading
dimension size and stride parameters.

```julia-repl
julia> a = rand(10,10)
10×10 Array{Float64,2}:
 0.561255   0.226678   0.203391  0.308912   …  0.750307  0.235023   0.217964
 0.718915   0.537192   0.556946  0.996234      0.666232  0.509423   0.660788
 0.493501   0.0565622  0.118392  0.493498      0.262048  0.940693   0.252965
 0.0470779  0.736979   0.264822  0.228787      0.161441  0.897023   0.567641
 0.343935   0.32327    0.795673  0.452242      0.468819  0.628507   0.511528
 0.935597   0.991511   0.571297  0.74485    …  0.84589   0.178834   0.284413
 0.160706   0.672252   0.133158  0.65554       0.371826  0.770628   0.0531208
 0.306617   0.836126   0.301198  0.0224702     0.39344   0.0370205  0.536062
 0.890947   0.168877   0.32002   0.486136      0.096078  0.172048   0.77672
 0.507762   0.573567   0.220124  0.165816      0.211049  0.433277   0.539476

julia> b = view(a, 2:2:8,2:2:4)
4×2 SubArray{Float64,2,Array{Float64,2},Tuple{StepRange{Int64,Int64},StepRange{Int64,Int64}},false}:
 0.537192  0.996234
 0.736979  0.228787
 0.991511  0.74485
 0.836126  0.0224702

julia> (q,r) = qr(b);

julia> q
4×2 Array{Float64,2}:
 -0.338809   0.78934
 -0.464815  -0.230274
 -0.625349   0.194538
 -0.527347  -0.534856

julia> r
2×2 Array{Float64,2}:
 -1.58553  -0.921517
  0.0       0.866567
```

## Sparse Vectors and Matrices

Julia has built-in support for sparse vectors and
[sparse matrices](https://en.wikipedia.org/wiki/Sparse_matrix). Sparse arrays are arrays
that contain enough zeros that storing them in a special data structure leads to savings
in space and execution time, compared to dense arrays.

### [Compressed Sparse Column (CSC) Sparse Matrix Storage](@id man-csc)

In Julia, sparse matrices are stored in the [Compressed Sparse Column (CSC) format](https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_column_.28CSC_or_CCS.29).
Julia sparse matrices have the type [`SparseMatrixCSC{Tv,Ti}`](@ref), where `Tv` is the
type of the stored values, and `Ti` is the integer type for storing column pointers and
row indices. The internal representation of `SparseMatrixCSC` is as follows:

```julia
struct SparseMatrixCSC{Tv,Ti<:Integer} <: AbstractSparseMatrix{Tv,Ti}
    m::Int                  # Number of rows
    n::Int                  # Number of columns
    colptr::Vector{Ti}      # Column i is in colptr[i]:(colptr[i+1]-1)
    rowval::Vector{Ti}      # Row indices of stored values
    nzval::Vector{Tv}       # Stored values, typically nonzeros
end
```

The compressed sparse column storage makes it easy and quick to access the elements in the column
of a sparse matrix, whereas accessing the sparse matrix by rows is considerably slower. Operations
such as insertion of previously unstored entries one at a time in the CSC structure tend to be slow. This is
because all elements of the sparse matrix that are beyond the point of insertion have to be moved
one place over.

All operations on sparse matrices are carefully implemented to exploit the CSC data structure
for performance, and to avoid expensive operations.

If you have data in CSC format from a different application or library, and wish to import it
in Julia, make sure that you use 1-based indexing. The row indices in every column need to be
sorted. If your `SparseMatrixCSC` object contains unsorted row indices, one quick way to sort
them is by doing a double transpose.

In some applications, it is convenient to store explicit zero values in a `SparseMatrixCSC`. These
*are* accepted by functions in `Base` (but there is no guarantee that they will be preserved in
mutating operations). Such explicitly stored zeros are treated as structural nonzeros by many
routines. The [`nnz()`](@ref) function returns the number of elements explicitly stored in the
sparse data structure, including structural nonzeros. In order to count the exact number of
numerical nonzeros, use [`countnz()`](@ref), which inspects every stored element of a sparse
matrix. [`dropzeros()`](@ref), and the in-place [`dropzeros!()`](@ref), can be used to
remove stored zeros from the sparse matrix.

```jldoctest
julia> A = sparse([1, 2, 3], [1, 2, 3], [0, 2, 0])
3×3 SparseMatrixCSC{Int64,Int64} with 3 stored entries:
  [1, 1]  =  0
  [2, 2]  =  2
  [3, 3]  =  0

julia> dropzeros(A)
3×3 SparseMatrixCSC{Int64,Int64} with 1 stored entry:
  [2, 2]  =  2
```

### Sparse Vector Storage

Sparse vectors are stored in a close analog to compressed sparse column format for sparse
matrices. In Julia, sparse vectors have the type [`SparseVector{Tv,Ti}`](@ref) where `Tv`
is the type of the stored values and `Ti` the integer type for the indices. The internal
representation is as follows:

```julia
struct SparseVector{Tv,Ti<:Integer} <: AbstractSparseVector{Tv,Ti}
    n::Int              # Length of the sparse vector
    nzind::Vector{Ti}   # Indices of stored values
    nzval::Vector{Tv}   # Stored values, typically nonzeros
end
```

As for [`SparseMatrixCSC`](@ref), the `SparseVector` type can also contain explicitly
stored zeros. (See [Sparse Matrix Storage](@ref man-csc).).

### Sparse Vector and Matrix Constructors

La forma más sencilla de crear matrices dispersas es usar funciones equivalentes a las funciones [`zeros()`](@ref) y [`eye()`](@ref) que proporciona Julia para trabajar con matrices densas. Para producir matrices *sparse* en su lugar, puede usar los mismos nombres con el prefijo `sp`:

```jldoctest
julia> spzeros(3)
3-element SparseVector{Float64,Int64} with 0 stored entries

julia> speye(3,5)
3×5 SparseMatrixCSC{Float64,Int64} with 3 stored entries:
  [1, 1]  =  1.0
  [2, 2]  =  1.0
  [3, 3]  =  1.0
```

La función [`sparse()`](@ref) suele ser una forma útil de construir matrices *sparse*. Por ejemplo, para construir una matriz *sparse*, podemos ingresar un vector `I` de índices de fila, un vector` J` de índices de columna, y un vector `V` de valores almacenados (esto también se conoce como [formato COO (coordenada)](https://en.wikipedia.org/wiki/Sparse_matrix#Coordinate_list_.28COO.29)). `esparse (I,J,V)` construye una matriz *sparse* tal que `S[I[k], J[k]] = V[k]`. El constructor vectorial *sparse* equivalente es [`sparsevec`](@ref), que toma el vector de índice (fila)` I` y el vector `V` con los valores almacenados y construye un vector disperso `R` tal que `R[I[k]] = V[k]`.

```jldoctest sparse_function
julia> I = [1, 4, 3, 5]; J = [4, 7, 18, 9]; V = [1, 2, -5, 3];

julia> S = sparse(I,J,V)
5×18 SparseMatrixCSC{Int64,Int64} with 4 stored entries:
  [1 ,  4]  =  1
  [4 ,  7]  =  2
  [5 ,  9]  =  3
  [3 , 18]  =  -5

julia> R = sparsevec(I,V)
5-element SparseVector{Int64,Int64} with 4 stored entries:
  [1]  =  1
  [3]  =  -5
  [4]  =  2
  [5]  =  3
```

La inversa de las funciones [`sparse()`](@ref) y [`sparsevec`](@ref) es [`findnz()`](@ref), que recupera las entradas utilizadas para crear la matriz *sparse*. También hay una función [`findn`](@ref) que solo devuelve los vectores índice.

```jldoctest sparse_function
julia> findnz(S)
([1, 4, 5, 3], [4, 7, 9, 18], [1, 2, 3, -5])

julia> findn(S)
([1, 4, 5, 3], [4, 7, 9, 18])

julia> findnz(R)
([1, 3, 4, 5], [1, -5, 2, 3])

julia> findn(R)
4-element Array{Int64,1}:
 1
 3
 4
 5
```

Otra forma de crear una matriz *sparse* es convertir una matriz densa en una matriz *sparse* usando la función [`sparse()`](@ ref):

```jldoctest
julia> sparse(eye(5))
5×5 SparseMatrixCSC{Float64,Int64} with 5 stored entries:
  [1, 1]  =  1.0
  [2, 2]  =  1.0
  [3, 3]  =  1.0
  [4, 4]  =  1.0
  [5, 5]  =  1.0

julia> sparse([1.0, 0.0, 1.0])
3-element SparseVector{Float64,Int64} with 2 stored entries:
  [1]  =  1.0
  [3]  =  1.0
```

Puede ir en la otra dirección usando el constructor [`Array`](@ref). La función [`issparse()`](@ref) se puede usar para consultar si una matriz es dispersa.

```jldoctest
julia> issparse(speye(5))
true
```

### Sparse matrix operations

Las operaciones aritméticas en matrices *sparse* también funcionan como lo hacen en matrices densas. La indexación de, la asignación en y la concatenación de matrices *sparse* funcionan de la misma manera que las matrices densas. Las operaciones de indexación, especialmente la asignación, son costosas, cuando se llevan a cabo un elemento a la vez. En muchos casos, puede ser mejor convertir la matriz dispersa en formato `(I,J,V)` usando [`findnz()`](@ref), manipular los valores o la estructura en los vectores densos `(I,J,V) `, y luego reconstruir la matriz *sparse*.

### Correspondence of dense and sparse methods

La siguiente tabla proporciona una correspondencia entre los métodos incorporados en matrices *sparse* y sus métodos correspondientes en tipos de matriz densa. En general, los métodos que generan matrices *sparse* difieren de sus contrapartes densas en que la matriz resultante sigue el mismo patrón de dispersión que una matriz *sparse* dada `S`, o que la matriz *sparse* resultante tiene densidad `d`, es decir, cada elemento de matriz tiene una probabilidad `d` de ser diferente de cero.

Los detalles se pueden encontrar en la sección [Vectores y Matrices *Sparse*](@ ref stdlib-sparse-arrays) de la referencia de biblioteca estándar.

| Sparse                     | Densa                  | Descripción                                                                                                                                                           |
|:-------------------------- |:---------------------- |:--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`spzeros(m,n)`](@ref)     | [`zeros(m,n)`](@ref)   | Creates a *m*-by-*n* matrix of zeros. ([`spzeros(m,n)`](@ref) is empty.)                                                                                              |
| [`spones(S)`](@ref)        | [`ones(m,n)`](@ref)    | Creates a matrix filled with ones. Unlike the dense version, [`spones()`](@ref) has the same sparsity pattern as *S*.                                                 |
| [`speye(n)`](@ref)         | [`eye(n)`](@ref)       | Creates a *n*-by-*n* identity matrix.                                                                                                                                 |
| [`full(S)`](@ref)          | [`sparse(A)`](@ref)    | Interconverts between dense and sparse formats.                                                                                                                       |
| [`sprand(m,n,d)`](@ref)    | [`rand(m,n)`](@ref)    | Creates a *m*-by-*n* random matrix (of density *d*) with iid non-zero elements distributed uniformly on the half-open interval ``[0, 1)``.                            |
| [`sprandn(m,n,d)`](@ref)   | [`randn(m,n)`](@ref)   | Creates a *m*-by-*n* random matrix (of density *d*) with iid non-zero elements distributed according to the standard normal (Gaussian) distribution.                  |
| [`sprandn(m,n,d,X)`](@ref) | [`randn(m,n,X)`](@ref) | Creates a *m*-by-*n* random matrix (of density *d*) with iid non-zero elements distributed according to the *X* distribution. (Requires the `Distributions` package.) |
