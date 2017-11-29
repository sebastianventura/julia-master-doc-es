# Interfaces

Un montón de la potencia y extensibilidad de Julia viene de una colección de interfaces informales. Extendiendo unos pocos métodos específicos para que trabajen para un tipo personalizado, los objetos de este tipo no sólo reciben estas funcionalidades, sino que son también capaces de ser usados en otros métodos que han sido escritos para ser construidos genéricamente sobre esos comportamientos.

## [Iteración](@id man-interface-iteration)

| Métodos requeridos               |                        | Breve descripción                                                                     |
|:------------------------------ |:---------------------- |:------------------------------------------------------------------------------------- |
| `start(iter)`                  |                        | Devuelve el estado inicial de iteración                                                   |
| `next(iter, state)`            |                        | Devuelve el ítem actual y pone `state` en el siguiente                                          |
| `done(iter, state)`            |                        | Comprueba si quedan más ítems                                                |
| **Métodos opcionales importantes** | **Definiciones por defecto** | **Breve descripción**                                                                 |
| `iteratorsize(IterType)`       | `HasLength()`          | Uno de `HasLength()`,`HasShape()`,`IsInfinite()`, o `sizeUnknown()`, según convenga |
| `iteratoreltype(IterType)`     | `HasEltype()`          | Uno de `EltypeUnknown()` o `HasEltype()`, según convenga                              |
| `eltype(IterType)`             | `Any`                  | El tipo de los ítems devueltos por `next()`                                               |
| `length(iter)`                 | (*indefinido*)          | El número de ítems, si es conocido                                                        |
| `size(iter, [dim...])`         | (*indefinido*)          | El número de ítems en cada dimensión, si es conocido                                      |

| Valro devuelto por `iteratorsize(IterType)` | Métodos requeridos |
|:------------------------------------------ |:------------------------------------------ |
| `HasLength()`                              | `length(iter)`                             |
| `HasShape()`                               | `length(iter)`  and `size(iter, [dim...])` |
| `IsInfinite()`                             | (*ninguno*)                                   |
| `SizeUnknown()`                            | (*ninguno*)                                   |

| Valro devuelto por `iteratoreltype(IterType)` | Métodos requeridos  |
|:-------------------------------------------- |:------------------ |
| `HasEltype()`                                | `eltype(IterType)` |
| `EltypeUnknown()`                            | (*ninguno*)           |

La iteración secuencial es implementada mediante los métodos [`start()`](@ref), [`done()`](@ref), y [`next()`](@ref). En lugar de mutar objetos cuando se itera sobre ellos, Julia proporciona estos tres métodos que llevaqn la traza del estado de la iteración externamente al objeto. El método `start(iter)` devuvelve el estado inicial para un objeto iterable `iterd`. Este estado se pasa a lo largo de `done(iter, state)` que chequea si quedan más elementos, y `next(iter, state)` que devuelve una tupla que contiene el elemento y el estado actuales. El objeto `state`puede ser cualquier cosa, y suele ser considerado un detalle de implementación privado al objeto iterable.

Cualquier objeto que defina estos tres métodos es iterable y puede ser usado en las [muchas funciones que se basan en la iteración](@ref lib-collections-iteration). También puede ser usado directamente en un bucle for ya que la sintaxis:

```julia
for i in iter   # or  "for i = iter"
    # body
end
```

es traducida por:

```julia
state = start(iter)
while !done(iter, state)
    (i, state) = next(iter, state)
    # body
end
```

Un ejemplo sencillo es una secuencia iterable de cuadrados de número con una longitd definida:

```jldoctest squaretype
julia> struct Squares
           count::Int
       end

julia> Base.start(::Squares) = 1

julia> Base.next(S::Squares, state) = (state*state, state+1)

julia> Base.done(S::Squares, state) = state > S.count

julia> Base.eltype(::Type{Squares}) = Int # Note that this is defined for the type

julia> Base.length(S::Squares) = S.count
```

With only [`start`](@ref), [`next`](@ref), and [`done`](@ref) definitions, the `Squares` type is already pretty powerful.
We can iterate over all the elements:

```jldoctest squaretype
julia> for i in Squares(7)
           println(i)
       end
1
4
9
16
25
36
49
```

Podemos usar muchos de los métodos predefinidos que trabajan con iterables, como [`in()`](@ref), [`mean()`](@ref) y [`std()`](@ref):

```jldoctest squaretype
julia> 25 in Squares(10)
true

julia> mean(Squares(100))
3383.5

julia> std(Squares(100))
3024.355854282583
```

Hay unos pocos más métodos que se pueden extender para dar a Julia más información sobre esta colección iterable. Se sabe que todos los elementos en una secuencia `Squares` serán `Int`. Extendiendo el método  [`eltype()`](@ref) method, se puede proporcionar esta información a Julia y ayudarlo a hacer código más especializado en métodos más complicados. También se sabe el número de elementos de esa secuencia, por loq ue también se pude extender [`length()`](@ref).

Ahora, cuando pedimos a Julia que [`collect()`](@ref) todos los elementos en un array ella puede preasignar un `Vector{Int}` en la parte derecha de la expresión, en lugar de ir poniendo a ciegas mediante [`push!`](@ref)ing cada elemento en un `Vector{Any}`.

```jldoctest squaretype
julia> collect(Squares(10))' # transposed to save space
1×10 RowVector{Int64,Array{Int64,1}}:
 1  4  9  16  25  36  49  64  81  100
```

Aunque podemos confiar en las implementaciones genéricas, podemos también extender métodos específicos donde sepamos que hay un algoritmo más simple. Por ejemplo, he aquí una fórmula para sobreescribir la versión iterativa para una solución más eficiente:

```jldoctest squaretype
julia> Base.sum(S::Squares) = (n = S.count; return n*(n+1)*(2n+1)÷6)

julia> sum(Squares(1803))
1955361914
```

Este es un patrón muy común a través de la librería estándar de Julia: un pequeño conjunto de métodos requeridos definen una interfaz informal que permite muchos comportamientos muy atractivos. En algunos casos, los tipos que quieran especializar esos comportamientos extra cuando saben que existe un algoritmo más eficiente que podrán usar en su caso específico.

## Indexación

| Métodos a implementar | Breve descripción                   |
|:--------------------  |:----------------------------------- |
| `getindex(X, i)`      | `X[i]`, acceso indexado a elemento  | 
| `setindex!(X, v, i)`  | `X[i] = v`, asignación indexada     |
| `endof(X)`            | El último índice, usado en `X[end]` |

Para el iterable `Squares` anterior, podemos calcular fácilmente el i-ésimo elemento de la secuencia elevándolo al cuadrado. Pordemos exponer esto como una expresión de indexación `S[i]`. Para optar a ese comportamiento, `Squares` sólo tieen que definir [`getindex()`](@ref):

```jldoctest squaretype
julia> function Base.getindex(S::Squares, i::Int)
           1 <= i <= S.count || throw(BoundsError(S, i))
           return i*i
       end

julia> Squares(100)[23]
529
```

Adicionalmente, para soportar la sintaxis `S[end]`, debemos definir [`endof()`](@ref) para especificar el último índice válido::

```jldoctest squaretype
julia> Base.endof(S::Squares) = length(S)

julia> Squares(23)[end]
529
```

Tenga en cuenta, sin embargo, que lo anterior *sólo* define [`getindex()`](@ref) con un índice entero. Indexar con cualquier cosa que no sea un `Int` lanzará un  [`MethodError`](@ref) diciendo que no había ningún método coincidente. Para soportar la indexación con intervalos o vectores de `Int`s, se deben escribir métodos separados:

```jldoctest squaretype
julia> Base.getindex(S::Squares, i::Number) = S[convert(Int, i)]

julia> Base.getindex(S::Squares, I) = [S[i] for i in I]

julia> Squares(10)[[3,4.,5]]
3-element Array{Int64,1}:
  9
 16
 25
```

Aunque que esto está comenzando a soportar más de las [operaciones de indexación soportadas por algunos de los tipos incorporados](@ref man-array-indexing), todavía hay un buen número de comportamientos ausentes. Esta secuencia `Squares` está empezando a parecer más y más como un vector, ya que hemos añadido comportamientos a la misma. En lugar de definir todos estos comportamientos nosotros mismos, podemos definirlos oficialmente como un subtipo de un [`AbstractArray`](@ref).

## [Abstract Arrays](@id man-interface-array)

| Methods to implement                            |                                          | Brief description                                                                     |
|:----------------------------------------------- |:---------------------------------------- |:------------------------------------------------------------------------------------- |
| `size(A)`                                       |                                          | Devuelve una tupla que contiene las dimensiones de `A`                                      |
| `getindex(A, i::Int)`                           |                                          | (if `IndexLinear`) Linear scalar indexing                                              |
| `getindex(A, I::Vararg{Int, N})`                |                                          | (if `IndexCartesian`, where `N = ndims(A)`) N-dimensional scalar indexing                 |
| `setindex!(A, v, i::Int)`                       |                                          | (if `IndexLinear`) Scalar indexed assignment                                           |
| `setindex!(A, v, I::Vararg{Int, N})`            |                                          | (if `IndexCartesian`, where `N = ndims(A)`) N-dimensional scalar indexed assignment       |
| **Optional methods**                            | **Default definition**                   | **Brief description**                                                                 |
| `IndexStyle(::Type)`                            | `IndexCartesian()`                       | Returns either `IndexLinear()` or `IndexCartesian()`. See the description below.      |
| `getindex(A, I...)`                             | defined in terms of scalar `getindex()`  | [Multidimensional and nonscalar indexing](@ref man-array-indexing)                    |
| `setindex!(A, I...)`                            | defined in terms of scalar `setindex!()` | [Multidimensional and nonscalar indexed assignment](@ref man-array-indexing)          |
| `start()`/`next()`/`done()`                     | defined in terms of scalar `getindex()`  | Iteration                                                                             |
| `length(A)`                                     | `prod(size(A))`                          | Number of elements                                                                    |
| `similar(A)`                                    | `similar(A, eltype(A), size(A))`         | Return a mutable array with the same shape and element type                           |
| `similar(A, ::Type{S})`                         | `similar(A, S, size(A))`                 | Return a mutable array with the same shape and the specified element type             |
| `similar(A, dims::NTuple{Int})`                 | `similar(A, eltype(A), dims)`            | Return a mutable array with the same element type and size *dims*                     |
| `similar(A, ::Type{S}, dims::NTuple{Int})`      | `Array{S}(dims)`                         | Return a mutable array with the specified element type and size                       |
| **Non-traditional indices**                     | **Default definition**                   | **Brief description**                                                                 |
| `indices(A)`                                    | `map(OneTo, size(A))`                    | Return the `AbstractUnitRange` of valid indices                                       |
| `Base.similar(A, ::Type{S}, inds::NTuple{Ind})` | `similar(A, S, Base.to_shape(inds))`     | Return a mutable array with the specified indices `inds` (see below)                  |
| `Base.similar(T::Union{Type,Function}, inds)`   | `T(Base.to_shape(inds))`                 | Return an array similar to `T` with the specified indices `inds` (see below)          |

Si un tipo se define como subtipo de `AbstractArray`, hereda un conjunto muy grande de comportamientos ricos, incluyendo la iteración y la indexación multidimensional construida sobre el acceso de un solo elemento. Consulte la [página de manual sobre arrays](@ref man-multi-dim-arrays) y la [sección de la biblioteca estándar](@ref lib-arrays) para más métodos soportados.

Una parte clave en la definición de un subtipo de `AbstractArray` es [`IndexStyle`](@ref).Dado que la indexación es una parte tan importante de una matriz y que a menudo se produce en los bucles en caliente, es importante que tanto la indexación y la asignación indexada sean lo más eficientes posible. Las estructuras de datos array se suelen definir de dos maneras: o bien accede de forma más eficaz a sus elementos utilizando sólo un índice (indexación lineal) o  accede intrínsecamente a los elementos con índices especificados para cada dimensión. Estas dos modalidades son identificadas por Julia como `IndexLinear()` e `IndexCartesian()`. La conversión de un índice lineal en subíndices de indexación múltiples suele ser muy costosa, por lo que esto proporciona un mecanismo basado en tratos para permitir un código genérico eficiente para todos los tipos de matriz.

Esta distinción determina qué métodos de indexación escalar debe definir cada tipo. Los arrays `IndexLinear()` son sencillos: sólo definen `getindex(A::ArrayType, i::Int)`. Cuando el array se indexa posteriormmente con un conjunto multidimensional de índices, el método de respaldo `getindex(A::AbstractArray, I...)()` convierte eficientemente los indices en un indice lineal y luego llama al metodo anterior. Los arrays `IndexCartesian()`, por otra parte,  requieren que se definan métodos para cada dimensionalidad soportada con `ndims(A)` índices Int. Por ejemplo, el tipo [`SparseMatrixCSC`](@ref) incorporado sólo admite dos dimensiones, por lo que sólo define `getindex(A::SparseMatrixCSC, i::Int, j::Int)`. Lo mismo sucede para `setindex!()`.

Volviendo a la secuencia de cuadrados de arriba, podríamos definirla como un subtipo de un `AbstractArray{Int, 1}`:

```jldoctest squarevectype
julia> struct SquaresVector <: AbstractArray{Int, 1}
           count::Int
       end

julia> Base.size(S::SquaresVector) = (S.count,)

julia> Base.IndexStyle(::Type{<:SquaresVector}) = IndexLinear()

julia> Base.getindex(S::SquaresVector, i::Int) = i*i
```

Note que es muy importante especificar los dos parámetros del `AbstractArray`; la primera define el tipo de elemento  [`eltype()`](@ref), y la segunda el número de dimensiones [`ndims()`](@ref). Este supertipo y sus tres métodos son todo lo que hace falta para que `SquaresVector` sea un array iterable, indexable y completamente funcional:

```jldoctest squarevectype
julia> s = SquaresVector(7)
7-element SquaresVector:
  1
  4
  9
 16
 25
 36
 49

julia> s[s .> 20]
3-element Array{Int64,1}:
 25
 36
 49

julia> s \ [1 2; 3 4; 5 6; 7 8; 9 10; 11 12; 13 14]
1×2 Array{Float64,2}:
 0.305389  0.335329

julia> s ⋅ s # dot(s, s)
4676
```

Un ejemplo un poco más complicado. definamos nuestro propio tipo array *sparse* N-dimensional "de juguete", construído encima de [`Dict`](@ref):

```jldoctest squarevectype
julia> struct SparseArray{T,N} <: AbstractArray{T,N}
           data::Dict{NTuple{N,Int}, T}
           dims::NTuple{N,Int}
       end

julia> SparseArray{T}(::Type{T}, dims::Int...) = SparseArray(T, dims);

julia> SparseArray{T,N}(::Type{T}, dims::NTuple{N,Int}) = SparseArray{T,N}(Dict{NTuple{N,Int}, T}(), dims);

julia> Base.size(A::SparseArray) = A.dims

julia> Base.similar(A::SparseArray, ::Type{T}, dims::Dims) where {T} = SparseArray(T, dims)

julia> Base.getindex(A::SparseArray{T,N}, I::Vararg{Int,N}) where {T,N} = get(A.data, I, zero(T))

julia> Base.setindex!(A::SparseArray{T,N}, v, I::Vararg{Int,N}) where {T,N} = (A.data[I] = v)
```

Observe que se trata de un array  `IndexCartesian` array, , por lo que debemos definir manualmente [`getindex()`](@ref) y [`setindex!()`](@ref) en la dimensionalidad de la matriz. En este caso, a diferencia de en `SquaresVector`, somos capaces de definir  [`setindex!()`](@ref) y, en consecuencia, podemos mutar el array:

```jldoctest squarevectype
julia> A = SparseArray(Float64, 3, 3)
3×3 SparseArray{Float64,2}:
 0.0  0.0  0.0
 0.0  0.0  0.0
 0.0  0.0  0.0

julia> fill!(A, 2)
3×3 SparseArray{Float64,2}:
 2.0  2.0  2.0
 2.0  2.0  2.0
 2.0  2.0  2.0

julia> A[:] = 1:length(A); A
3×3 SparseArray{Float64,2}:
 1.0  4.0  7.0
 2.0  5.0  8.0
 3.0  6.0  9.0
```

El resultado de la indexación de un `AbstractArray` puede ser en sí mismo un array (por ejemplo, al indexar por un rango). Los métodos de respaldo de `AbstractArray` utilizan [`similar()`](@ref) para asignar un `Array` del tamaño y tipo de elemento apropiados, que se rellena usando el método de indexación básico descrito anteriormente. Sin embargo, al implementar un *wrapper* de array, a menudo deseamos que el resultado sea también un *wrapper*:

```jldoctest squarevectype
julia> A[1:2,:]
2×3 SparseArray{Float64,2}:
 1.0  4.0  7.0
 2.0  5.0  8.0
```

En este ejemplo esto se logra mediante la definición de `Base.similar{T}(A::SparseArray, :: Type{T}, dims::Dims)` para crear la matriz wrapped apropiada. (Tenga en cuenta que aunque `similar` soporta formas de 1 y 2 argumentos, en la mayoría de los casos sólo necesita especializar el formulario de 3 argumentos). Para que esto funcione es importante que `SparseArray` sea mutable (soporte `setindex!`). Definir `similar()`, `getindex()` y `setindex!()` para `SparseArray` también hace posible copiar el array mediante [`copy()`](@ref):

```jldoctest squarevectype
julia> copy(A)
3×3 SparseArray{Float64,2}:
 1.0  4.0  7.0
 2.0  5.0  8.0
 3.0  6.0  9.0
```

Además de todos los métodos iterables e indexables de arriba, estos tipos también pueden interactuar entre sí y utilizar todos los métodos definidos en la biblioteca estándar para `AbstractArrays`:

```jldoctest squarevectype
julia> A[SquaresVector(3)]
3-element SparseArray{Float64,1}:
 1.0
 4.0
 9.0

julia> dot(A[:,1],A[:,2])
32.0
```

Si está definiendo un tipo de array que permite la indexación no tradicional (índices que comienzan en algo distinto de 1), debe especializar `indices`. También debe especializarse [`similar`](@ref) para que el argumento `dims` (normalmente una tupla de tamaños `Dims`) pueda aceptar objetos `AbstractUnitRange`, tal vez rango-tipos `Ind` de su propio diseño. Para obtener más información, vea [Arrays con índices personalizados](@ref).
