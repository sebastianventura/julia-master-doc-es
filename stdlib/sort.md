# Ordenación y Funciones Relacionadas

Julia tiene una API amplia y flexible para ordenar e interactuar con matrices de valores ya ordenados. Por defecto, Julia selecciona algoritmos y ordenaciones razonables en orden ascendente estándar:

```jldoctest
julia> sort([2,3,1])
3-element Array{Int64,1}:
 1
 2
 3
```

Uno también puede ordenar en orden inverso:

```jldoctest
julia> sort([2,3,1], rev=true)
3-element Array{Int64,1}:
 3
 2
 1
```

Para ordenar un array en el lugar, use la versión con admiración de la función de ordenación:

```jldoctest
julia> a = [2,3,1];

julia> sort!(a);

julia> a
3-element Array{Int64,1}:
 1
 2
 3
```

En lugar de ordenar un array directamente, podemos computar una preemutación de los índices del array que ponen el array en un orden determinado:

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

Los arrays pueden ser ordenados fácilmente de acuerdo a una transformacin arbitraria de sus valores:

```julia-repl
julia> sort(v, by=abs)
5-element Array{Float64,1}:
 -0.0104452
  0.297288
  0.382396
 -0.597634
 -0.839027
```

O en orden reverso mediante una transformación

```julia-repl
julia> sort(v, by=abs, rev=true)
5-element Array{Float64,1}:
 -0.839027
 -0.597634
  0.382396
  0.297288
 -0.0104452
```

Si es necesario, puede elegirse el algoritmo de ordenación:

```julia-repl
julia> sort(v, alg=InsertionSort)
5-element Array{Float64,1}:
 -0.839027
 -0.597634
 -0.0104452
  0.297288
  0.382396
```

Todas las funciones de ordenación y relacionadas con orden se basan en una relación "menor que" que define un orden total sobre los valores que van a manipularse. La función `isless` es la invocada por defecto, pero la relación puede ser especificada mediante la palabra clave `lt`.

## Funciones de Ordenación

```@docs
Base.sort!
Base.sort
Base.sortperm
Base.Sort.sortperm!
Base.Sort.sortrows
Base.Sort.sortcols
```

## Funciones relacionadas con Orden

```@docs
Base.issorted
Base.Sort.searchsorted
Base.Sort.searchsortedfirst
Base.Sort.searchsortedlast
Base.Sort.select!
Base.Sort.select
Base.Sort.selectperm
Base.Sort.selectperm!
```

## Algoritmos de Ordenación

Actualmente hay cuatro algoritmos de ordenación disponibles en Julia base:

  * `InsertionSort`
  * `QuickSort`
  * `PartialQuickSort(k)`
  * `MergeSort`

`InsertionSort` es un algoritmo de ordenación estable cuyo coste es O(n^2). Es eficiente para `n` muy pequeños, y es usado internamente por `QuickSort`.

`QuickSort` es un algoritmo de ordenación que es *in-place* muy rápido pero no estable (es decir, los elementos que son considerados iguales no permanecerán en el mismo orden en que se encontraban originalmente en el array antes de ser ordenados. Su coste computacional es O(n log n). `QuickSort` es el algoritmo por defecto para valores numéricos, incluyendo enteros y punto flotante.

`PartialQuickSort(k)` es similar a `QuickSort`, pero el array de salida es sólo ordenado hasta el índice 
`k` si `k` es un entero, o en el rango de `k` si `k` es un `OrdinalRange`. Por ejemplo:

```julia
x = rand(1:500, 100)
k = 50
k2 = 50:100
s = sort(x; alg=QuickSort)
ps = sort(x; alg=PartialQuickSort(k))
qs = sort(x; alg=PartialQuickSort(k2))
map(issorted, (s, ps, qs))             # => (true, false, false)
map(x->issorted(x[1:k]), (s, ps, qs))  # => (true, true, false)
map(x->issorted(x[k2]), (s, ps, qs))   # => (true, false, true)
s[1:k] == ps[1:k]                      # => true
s[k2] == qs[k2]                        # => true
```

`MergeSort` es un algoritmo de ordenación estable, pero no *in-place* (requiere un array temporal de la mitad del tamaño del array de entrada), de coste O(n log n) y no suele ser tan rapido como `QuickSort`. Es el algoritmo por defecto para datos no numéricos.

Los algoritmos de clasificación por defecto se eligen sobre la base de que son rápidos y estables, o *parezcan* serlo. Para los tipos numéricos, de hecho, se selecciona `QuickSort` ya que es más rápido e indistinguible en este caso de un tipo estable (a menos que la matriz registre sus mutaciones de alguna manera). La propiedad de estabilidad tiene un costo no despreciable, por lo que si no la necesita, puede especificar explícitamente su algoritmo preferido, p. `sort!(v, alg=QuickSort)`.

El mecanismo por el cual Julia selecciona los algoritmos de clasificación predeterminados se implementa a través de la función `Base.Sort.defalg`. Permite que un algoritmo particular se registre como el predeterminado en todas las funciones de ordenación para arrays específicos. Por ejemplo, aquí están los dos métodos predeterminados de [`sort.jl`] (https://github.com/JuliaLang/julia/blob/master/base/sort.jl):

The mechanism by which Julia picks default sorting algorithms is implemented via the `Base.Sort.defalg`
function. It allows a particular algorithm to be registered as the default in all sorting functions
for specific arrays. For example, here are the two default methods from [`sort.jl`](https://github.com/JuliaLang/julia/blob/master/base/sort.jl):

```julia
defalg(v::AbstractArray) = MergeSort
defalg(v::AbstractArray{<:Number}) = QuickSort
```

En cuanto a los arrays numéricos, la elección de un algoritmo predeterminado no estable para los tipos de array para los cuales la noción de ordenación estable no tiene sentido (es decir, cuando dos valores que comparan iguales no se pueden distinguir) puede tener sentido.
