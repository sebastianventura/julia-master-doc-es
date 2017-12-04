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

La sintaxis `[A, B, C, ...]` construye un array 1-dimensional (vector) a partir de sus argumentos. Si todos los argumentos tienen un [tipo de promocion](@ref conversion-and-promotion) comun entonces ellos son convertidos a este tipo usando `convert()`.

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

### [Tipos índices Soportados](@id man-supported-index-types)

En la expresión `A[I_1, I_2, ..., I_n]`, cada `I_k` puede ser un índice escalar, un array de índices escalares o un objeto que repreenta un array de índices escalares y puede ser convertido a tal mediante [`to_indices`](@ref):

1. Un índice escalar. Por defecto esto incluye:
    * Enteros no booleanos
    * `CartesianIndex{N}`s, que se comportan como una `N`-tupla de enteros abarcando múltiples dimensiones (ver abajo para ms detalles)
2. Un array de índices escalares. Esto incluye:
    * Vectores y arrays multidimensionales de enteros
    * Arrays vacíos como `[]`, que no selecciona elementos
    * `Range`s de la forma `a:c` o `a:b:c`, que seleccionan subsecciones contiguas o con salto desde `a` hasta `c` (inclusive)
    * Cualquier array de índices escalares que sea un subtipo de `AbstractArray`
    * Arrays de `CartesianIndex{N}` (ver abajo para ms detalles)
3. Un objeto que representa un array de índice escalares y puede ser convertido a tal mediante [`to_indices`](@ref). Por defecto esto incluye:
    * [`Colon()`](@ref) (`:`), qu representa todos los índices dentro de una dimensin entera o a través del array completo
    * Arrays de booleans, que seleccionan los elementos en los que sus índices son `true` indices (ver abajo para más detalles)

#### Índices Cartesianos

El objeto especial `CartesianIndex{N}` representa un índice escalar que se comporta como una `N`-tupla de enteros que abarcan multiples dimensioneas. Por ejemplo:

```jldoctest cartesianindex
julia> A = reshape(1:32, 4, 4, 2);

julia> A[3, 2, 1]
7

julia> A[CartesianIndex(3, 2, 1)] == A[3, 2, 1] == 7
true
```
Considerado solo, esto puede parecer relativamente trivial; `CartesianIndex` simplemente reúne múltiples enteros juntos en un objeto que representa un único índice multidimensional. Sin embargo, cuando se combina con otras formas de indexación e iteradores que producen `CartesianIndex`es, esto puede conducir directamente a un código muy elegante y eficiente. Ver [Iteración](@ref) a continuación, y para algunos ejemplos más avanzados, ver [esta publicación en el blog sobre algoritmos multidimensionales e iteración](https://julialang.org/blog/2016/02/iteration).

Las matrices de `CartesianIndex {N}` también son compatibles. Representan una colección de índices escalares que abarcan cada una de las dimensiones `N`, lo que permite una forma de indexación que a veces se denomina indexación puntual. Por ejemplo, permite acceder a los elementos diagonales desde la primera "página" de 'A' desde arriba:

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
Esto se puede expresar mucho más simplemente con [*dot broadcasting*] (@ref man-vectorized) y combinándolo con un índice entero normal (en lugar de extraer la primera `page` de` A` como un paso separado). Incluso se puede combinar con `:` para extraer ambas diagonales de las dos páginas al mismo tiempo:

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

    Los objetos `CartesianIndex` y los arrays de `CartesianIndex` no son compatibles con la palabra clave 
    `end` para representar el último índice de una dimensión. No use la palabra `end` en expresiones de
    indexación que puedan contener `CartesianIndex` o *arrays* en ellos.

#### Indexación Lógica

A menudo denomnada indexación lógica o indexación con una máscara lógica, la indexación mediante  una matriz booleana selecciona elementos en los índices cuyos valores son `verdaderos`. La indexación por un vector booleano `B` es efectivamente igual a la indexación por el vector de enteros que es devuelto por [`find (B)`](@ref). De forma similar, la indexación por una matriz booleana `N`-dimensional es efectivamente igual a la indexación por el vector de `CartesianIndex{N}`s donde sus valores son` true`. Un índice lógico debe ser un vector de la misma longitud que la dimensión en la que indexa, o debe ser el único índice proporcionado y debe coincidir con el tamaño y la dimensionalidad de la matriz en la que se indexa. En general, es más eficiente usar matrices booleanas como índices directamente en lugar de llamar primero a [`find()`](@ref).

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

### Iteración

Las formas recomendadas de iterar sobre un array completo son:

```julia
for a in A
    # Do something with the element a
end

for i in eachindex(A)
    # Do something with i and/or A[i]
end
```

La primera construcción se usa cuando necesitamos el valor, pero no los índices, de cada elemento. En la segunda construcción, `i` será un `Int` si `A` es un tipo array con indexacin lineal rápida; en caso contrario será un `CartesianIndex`:

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

En contraste con `for i = 1:length(A)`, iterar con `eachindex` proporciona una forma eficiente de iterar sobre cualquier tipo de array.

### Array traits

Si uno escribe un tipo [`AbstractArray`](@ref) personalizado, uno puede especificar que el tipo tiene indexación lineal rápida usando

```julia
Base.IndexStyle(::Type{<:MyArray}) = IndexLinear()
```

Esta configuración hará que la iteración `eachindex` sobre un objeto `MyArray` use enteros. Si no especifica este rasgo, se usa el valor predeterminado `IndexCartesian()`.

### Operaciones y Funciones Array y Vectorizadas

Los siguientes operadores están soportados para arrays:

1. Aritmética unaria -- `-`, `+`
2. Aritmética binaria -- `-`, `+`, `*`, `/`, `\`, `^`
3. Comparación -- `==`, `!=`, `≈` ([`isapprox`](@ref)), `≉`

La mayoría de los operadores aritméticos binarios enumerados anteriormente también funcionan con elementos cuando un argumento es escalar: `-`,` + `, y` * `cuando cualquiera de los argumentos es escalar, y `/` y `\` cuando el denominador es escalar. Por ejemplo, `[1, 2] + 3 == [4, 5]` y `[6, 4] / 2 == [3, 2]`.

Además, para permitir una conveniente vectorización de operaciones matemáticas y de otro tipo, Julia [proporciona la sintaxis punto](@ref man-vectorized) `f.(args ...)`, por ejemplo, `sin.(x)` o `min.(x, y)`, para operaciones con elementos sobre arrays o mezclas de matrices y escalares (una [Retransmisión (*broadcasting*)](@ref)); estos tienen la ventaja adicional de
"fusión" en un solo bucle cuando se combina con otras llamadas de puntos, por ejemplo, `sin.(cos.(x))`.

Además, *cada* operador binario admite una [versión de punto](@ref man-dot-operators) que se puede aplicar a matrices (y combinaciones de matrices y escalares) en tales [operaciones de retransmisión fusionadas](@ref man-vectorized), por ejemplo, `z .== sin.(x. * y)`.

Tenga en cuenta que las comparaciones como `==` operan en arrays completos, dando un solo booleano como respuesta. Use operadores de punto como `.==` para comparaciones elemento a elemento. (Para operaciones de comparación como `<`, *solo* la versión de elementos `.<` es aplicable a las matrices).

También note la diferencia entre `max.(a, b)`, que retransmitir [`max()`](@ref) elemento a elemento sobre `a` y` b`, y `maximum(a)`, que encuentra el mayor valor dentro de `a`. La misma relación se cumple para `min.(A, b)` y `minimum(a)`.

### Retransmisión

A veces es útil realizar operaciones binarias elemento por elemento en matrices de diferentes tamaños, como agregar un vector a cada columna de una matriz. Una forma ineficiente de hacer esto sería replicar el vector al tamaño de la matriz:

```julia-repl
julia> a = rand(2,1); A = rand(2,3);

julia> repmat(a,1,3)+A
2×3 Array{Float64,2}:
 1.20813  1.82068  1.25387
 1.56851  1.86401  1.67846
```

Esto es un desperdicio cuando las dimensiones son grandes, entonces Julia ofrece [`broadcast()`](@ref), que expande las dimensiones *singleton* en los argumentos del matriz para hacer coincidir la dimensión correspondiente en la otra matriz sin usar memoria extra, y aplica la función dada elemento a elemento:

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

[Los operadores con punto](@ref man-dot-operators) tales como `.+` y `.*` son equivalentes a llamadas a `broadcast` (excepto que se funden, como se describe a continuación). También hay una función [`broadcast!()`](@ref) para especificar un destino explícito (al que también se puede acceder por fusión mediante asignación `.=`), y funciones [`broadcast_getindex ()`](@ref) y [`broadcast_setindex! ()`] (@ref) que retransmiten los índices antes de indexar. Además, `f. (Args ...)` es equivalente a `broadcast(f, args ...)`, proporcionando una sintaxis conveniente para transmitir cualquier función ([sintaxis de punto](@ref man-vectorized)). "Llamadas punto" anidadas `f. (...)` (incluidas las llamadas a `.+` Etcétera) [fusibles automáticamente](@ ref man-dot-operators) en una sola llamada `broadcast`.

Además, [`broadcast ()`] (@ref) no está limitado a las matrices (ver la documentación de la función), también maneja tuplas y trata cualquier argumento que no sea una matriz, tupla o `Ref` (excepto para` Ptr` ) como un "escalar".

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

### Implementación

El tipo de matriz base en Julia es el tipo abstracto [`AbstractArray {T, N}`](@ref). Este tipo está parametrizado por el número de dimensiones `N` y el tipo de elementos `T`. [`AbstractVector`](@ref) y [`AbstractMatrix`](@ref) son alias para los casos 1-d y 2-d. Las operaciones en los objetos `AbstractArray` se definen usando operadores y funciones de alto nivel, de  manera que es independiente del almacenamiento subyacente. Estas operaciones generalmente funcionan correctamente como una alternativa para cualquier implementación de matriz específica.

El tipo `AbstractArray` incluye algo vagamente parecido a un array, y las implementaciones de este podrían ser bastante diferentes de laos arrays convencionales. Por ejemplo, los elementos se pueden calcular a petición en lugar de ser almacenados. Sin embargo, cualquier tipo concreto de `AbstractArray{T, N}` debería implementar al menos [`size(A)`](@ref) (devolviendo una tupla `Int`), [`getindex(A,i)`](@ref) y [`getindex(A, i1, ..., iN)`](@ref getindex); Las matrices mutables también deberían implementar [`setindex!()`](@ ref). Se recomienda que estas operaciones tengan complejidad temporal casi constante, o técnicamente complejidad de orden 1, ya que de lo contrario algún conjunto de funciones pueden ser inesperadamente lentas. Los tipos concretos también deberían proporcionar un método [`similar(A, T = el type(A), dims = size(A))`](@ ref), que se utiliza para asignar un conjunto similar para [`copy()`](@ref) y otras operaciones de actualización. No importa cómo se represente internamente un `AbstractArray {T, N}`, `T` es el tipo de objeto devuelto por *integer* indización (`A [1, ..., 1]`, cuando `A` no está vacío) y` N` debe ser la longitud de la tupla devuelta por [`size()`] (@ref).

`DenseArray` es un subtipo abstracto de` AbstractArray` que pretende incluir todas las matrices que están
establecido en compensaciones regulares en la memoria, y que por lo tanto se puede pasar a C externo y Fortran
funciones que esperan este diseño de memoria. Los subtipos deberían proporcionar un método [`stride (A, k)`] (@ ref)
que devuelve el "paso" de la dimensión `k`: aumentar el índice de dimensión` k` por `1` debería
aumente el índice `i` de [` getindex (A, i) `] (@ ref) por [` stride (A, k) `] (@ ref). Si una conversión de puntero
método [`Base.unsafe_convert (Ptr {T}, A)`] (@ ref), el diseño de la memoria debe corresponder
de la misma manera a estos pasos.

El tipo [`Array`](@ref) es una instancia específica de `DenseArray` donde los elementos se almacenan en orden de columnas principales (consulte notas adicionales en [Sugerencias de rendimiento](@ref man-performance-tips)). [`Vector`](@ref) y [`Matrix`](@ref) son alias para los casos 1-d y 2-d. Las operaciones específicas como la indexación escalar, la asignación y algunas otras operaciones básicas específicas del almacenamiento son todas las que deben implementarse para [`Array`](@ref), de modo que el resto de la biblioteca de matriz pueda implementarse de una forma genérica.

`SubArray` es una especialización de `AbstractArray` que realiza indexación por referencia en lugar de copiar. Un `SubArray` se crea con la función [`view()`](@ref), que se llama de la misma manera que [`getindex()`](@ref) (con una matriz y una serie de argumentos de índice) . El resultado de [`view()`](@ref) se ve igual que el resultado de [`getindex()`] (@ref), excepto que los datos se dejan en su lugar. [`view()`](@ref) almacena los vectores de índice de entrada en un objeto `SubArray`, que luego puede usarse para indexar la matriz original de forma indirecta. Al colocar la macro [`@views`](@ref) delante de una expresión o bloque de código, cualquier segmento `array[...]` en esa expresión se convertirá para crear una vista `SubArray` en su lugar.

`StridedVector` y` StridedMatrix` son alias convenientes definidos para que Julia pueda llamar a un rango más amplio de funciones BLAS y LAPACK pasándoles objetos [`Array`](@ref) o `SubArray`, y ahorrando así ineficiencias de asignación de memoria y copia.

El siguiente ejemplo calcula la descomposición QR de una pequeña sección de una matriz más grande, sin crear ningún temporario, y llamando a la función LAPACK apropiada con el tamaño de la dimensión principal adelantada y los parámetros de salto.

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

## Vectores y Matrices *Sparse*

Julia tiene soporte integrado para vectores y [matrices dispersas (*sparse*)](https://en.wikipedia.org/wiki/Sparse_matrix). Las matrices *sparse* son matrices que contienen suficientes ceros para almacenarlos en una estructura de datos especial que ahorra espacio y tiempo de ejecución, en comparación con las matrices densas.

### [Compressed Sparse Column (CSC) Sparse Matrix Storage](@id man-csc)

En Julia, las matrices dispersas se almacenan en el formato [Compressed Sparse Column (CSC)] (https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_column_.28CSC_or_CCS.29).
Las matrices *sparse* de Julia tienen el tipo [`SparseMatrixCSC{Tv, Ti}`](@ref), donde `Tv` es el tipo de los valores almacenados, y` Ti` es el tipo entero para almacenar punteros de columnas e índices de filas. La representación interna de `SparseMatrixCSC` es la siguiente:

```julia
struct SparseMatrixCSC{Tv,Ti<:Integer} <: AbstractSparseMatrix{Tv,Ti}
    m::Int                  # Number of rows
    n::Int                  # Number of columns
    colptr::Vector{Ti}      # Column i is in colptr[i]:(colptr[i+1]-1)
    rowval::Vector{Ti}      # Row indices of stored values
    nzval::Vector{Tv}       # Stored values, typically nonzeros
end
```

El almacenamiento de columnas dispersas y comprimidas facilita y agiliza el acceso a los elementos en la columna de una matriz *sparse*, mientras que el acceso a la matriz *sparse* por filas es considerablemente más lento. Las operaciones como la inserción de entradas previamente no almacenadas de una en una en la estructura de CSC tienden a ser lentas. Esto se debe a que todos los elementos de la matriz *sparse* que están más allá del punto de inserción deben moverse un lugar más.

Todas las operaciones en matrices *sparse* se implementan cuidadosamente para aprovechar la estructura de datos CSC para el rendimiento y para evitar operaciones costosas.

Si tiene datos en formato CSC desde una aplicación o biblioteca diferente, y desea importarlos en Julia, asegúrese de utilizar la indexación basada en 1. Los índices de fila en cada columna deben ser ordenados. Si su objeto `SparseMatrixCSC` contiene índices de filas sin clasificar, una forma rápida de ordenarlos es haciendo una doble transposición.

En algunas aplicaciones, es conveniente almacenar valores cero explícitos en `SparseMatrixCSC`. Estas *son* aceptadas por funciones en `Base` (pero no hay garantía de que se conservarán en las operaciones de mutación). Tales ceros explícitamente almacenados son tratados como no estructurales por muchas rutinas. La función [`nnz()`](@ref) devuelve la cantidad de elementos almacenados explícitamente en la estructura de datos dispersos, incluidos los no-ceros estructurales. Para contar el número exacto de nozeros numéricos, use [`countnz()`](@ref), que inspecciona todos los elementos almacenados de un parásito
matriz. [`dropzeros()`](@ref), y el in-place [`dropzeros!()`](@ref), se puede usar para eliminar ceros almacenados de la matriz dispersa.

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

Los vectores dispersos se almacenan en un formato de columna dispersa análoga a comprimida para matrices dispersas. En Julia, los vectores dispersos tienen el tipo [`SparseVector {Tv, Ti}`] (@ ref) donde `Tv` es el tipo de los valores almacenados y` Ti` el tipo entero para los índices. La representación interna es la siguiente:

```julia
struct SparseVector{Tv,Ti<:Integer} <: AbstractSparseVector{Tv,Ti}
    n::Int              # Length of the sparse vector
    nzind::Vector{Ti}   # Indices of stored values
    nzval::Vector{Tv}   # Stored values, typically nonzeros
end
```

En cuanto a [`SparseMatrixCSC`](@ref), el tipo` SparseVector` también puede contener ceros almacenados explícitamente. (Consulte [Almacenamiento de matriz *sparse*] (@ ref man-csc).).

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

### Operaciones con matrices *sparse*

Las operaciones aritméticas en matrices *sparse* también funcionan como lo hacen en matrices densas. La indexación de, la asignación en y la concatenación de matrices *sparse* funcionan de la misma manera que las matrices densas. Las operaciones de indexación, especialmente la asignación, son costosas, cuando se llevan a cabo un elemento a la vez. En muchos casos, puede ser mejor convertir la matriz dispersa en formato `(I,J,V)` usando [`findnz()`](@ref), manipular los valores o la estructura en los vectores densos `(I,J,V) `, y luego reconstruir la matriz *sparse*.

### Correspondencia de métodos densos y *sparse*

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
