# Colecciones y Estructuras de Datos

## [Iteración](@id lib-collections-iteration)

La iteración secuencial es implementada por los métodos [`start()`](@ref), [`done()`](@ref) y [`next()`](@ref).
El bucle `for` general:

```julia
for i = I   # or  "for i in I"
    # body
end
```

es traducido a:

```julia
state = start(I)
while !done(I, state)
    (i, state) = next(I, state)
    # body
end
```

El objeto `state` puede ser cualquier cosa, y debería ser elegido apropiadamente para cada tipo iterable. 
Ver la [sección de manual sobre la interfaz de iteración](@ref man-interface-iteration) para ms detalles 
sobre detinir un tipo iterable personalizado.

```@docs
Base.start
Base.done
Base.next
Base.iteratorsize
Base.iteratoreltype
```

Completamente implementada por:

  * `Range`
  * `UnitRange`
  * `Tuple`
  * `Number`
  * [`AbstractArray`](@ref)
  * [`IntSet`](@ref)
  * [`ObjectIdDict`](@ref)
  * [`Dict`](@ref)
  * [`WeakKeyDict`](@ref)
  * `EachLine`
  * `AbstractString`
  * [`Set`](@ref)

## Colecciones generales

```@docs
Base.isempty
Base.empty!
Base.length(::Any)
```

Completamente implementado por:

  * `Range`
  * `UnitRange`
  * `Tuple`
  * `Number`
  * [`AbstractArray`](@ref)
  * [`IntSet`](@ref)
  * [`ObjectIdDict`](@ref)
  * [`Dict`](@ref)
  * [`WeakKeyDict`](@ref)
  * `AbstractString`
  * [`Set`](@ref)

## Colecciones Iterables

```@docs
Base.in
Base.eltype
Base.indexin
Base.findin
Base.unique
Base.unique!
Base.allunique
Base.reduce(::Any, ::Any, ::Any)
Base.reduce(::Any, ::Any)
Base.foldl(::Any, ::Any, ::Any)
Base.foldl(::Any, ::Any)
Base.foldr(::Any, ::Any, ::Any)
Base.foldr(::Any, ::Any)
Base.maximum(::Any)
Base.maximum(::Any, ::Any)
Base.maximum!
Base.minimum(::Any)
Base.minimum(::Any, ::Any)
Base.minimum!
Base.extrema(::Any)
Base.extrema(::AbstractArray, ::Any)
Base.indmax
Base.indmin
Base.findmax(::Any)
Base.findmax(::AbstractArray, ::Any)
Base.findmin(::Any)
Base.findmin(::AbstractArray, ::Any)
Base.findmax!
Base.findmin!
Base.sum
Base.sum!
Base.prod
Base.prod!
Base.any(::Any)
Base.any(::AbstractArray, ::Any)
Base.any!
Base.all(::Any)
Base.all(::AbstractArray, ::Any)
Base.all!
Base.count
Base.any(::Any, ::Any)
Base.all(::Any, ::Any)
Base.foreach
Base.map
Base.map!
Base.mapreduce(::Any, ::Any, ::Any, ::Any)
Base.mapreduce(::Any, ::Any, ::Any)
Base.mapfoldl(::Any, ::Any, ::Any, ::Any)
Base.mapfoldl(::Any, ::Any, ::Any)
Base.mapfoldr(::Any, ::Any, ::Any, ::Any)
Base.mapfoldr(::Any, ::Any, ::Any)
Base.first
Base.last
Base.step
Base.collect(::Any)
Base.collect(::Type, ::Any)
Base.issubset(::Any, ::Any)
Base.filter
Base.filter!
```

## Colecciones Indexables

```@docs
Base.getindex(::Any, ::Any...)
Base.setindex!(::Any, ::Any, ::Any...)
Base.endof
```

Completamente implementado por:

  * [`Array`](@ref)
  * [`BitArray`](@ref)
  * [`AbstractArray`](@ref)
  * `SubArray`

Parcialmente implementado por:

  * `Range`
  * `UnitRange`
  * `Tuple`
  * `AbstractString`
  * [`Dict`](@ref)
  * [`ObjectIdDict`](@ref)
  * [`WeakKeyDict`](@ref)

## Colecciones asociativas

[`Dict`](@ref) es la colección asociativa estándar. Su implementación usa [`hash()`](@ref)
como función de hashing para la clave, e [`isequal()`](@ref) para determinar la igualdad. 
Si redefine estas dos funciones en un tipo personalizado sobreescribiran como se almacenan
dichos tipos en una tabla hash.

[`ObjectIdDict`](@ref) es una tabla hash especial donde las claves son siempre identidades de objeto.

[`WeakKeyDict`] (@ ref) es una implementación de tabla hash donde las claves son referencias 
débiles a los objetos y, por lo tanto, permiten recolección de basura recogida incluso cuando se 
referencian en una tabla hash.

[`Dict`](@ref)s se pueden crear pasando pares de objetos construidos con `=>() `a un constructor [`Dict`](@ref): `Dict ("A"=> 1," B "=> 2)`. Esta llamada intentará inferir información de tipo de las claves y valores (es decir, este ejemplo crea un `Dict{String, Int64}`). Para especificar explícitamente los tipos, use la sintaxis `Dict{KeyType,ValueType}(...)`. Por ejemplo, `Dict{String,Int32}(" A "=> 1," B "=> 2)`.

Las colecciones asociativas pueden también ser creadas con generadores. Por ejemplo, `Dict(i => f(i) for i = 1:10)`.

Dado un diccionario `D`, la sintaxis` D[x]` devuelve el valor de la clave `x` (si existe) o arroja un error, y `D[x] = y` almacena el par de clave-valor `x => y` en `D` (reemplazando cualquier valor existente por la clave` x`). Múltiples argumentos para `D [...]` se convierten a tuplas; por ejemplo, la sintaxis `D[x,y]` es equivalente a `D[(x,y)]`, es decir, se refiere al valor introducido por la tupla `(x,y)`.

```@docs
Base.Dict
Base.ObjectIdDict
Base.WeakKeyDict
Base.haskey
Base.get(::Any, ::Any, ::Any)
Base.get
Base.get!(::Any, ::Any, ::Any)
Base.get!(::Function, ::Any, ::Any)
Base.getkey
Base.delete!
Base.pop!(::Any, ::Any, ::Any)
Base.keys
Base.values
Base.merge
Base.merge!(::Associative, ::Associative...)
Base.merge!(::Function, ::Associative, ::Associative...)
Base.sizehint!
Base.keytype
Base.valtype
```

Completamente implementado por:

  * [`ObjectIdDict`](@ref)
  * [`Dict`](@ref)
  * [`WeakKeyDict`](@ref)

Parcialmente implementado por:

  * [`IntSet`](@ref)
  * [`Set`](@ref)
  * [`EnvHash`](@ref Base.EnvHash)
  * [`Array`](@ref)
  * [`BitArray`](@ref)

## Colecciones tipo Conjunto

```@docs
Base.Set
Base.IntSet
Base.union
Base.union!
Base.intersect
Base.setdiff
Base.setdiff!
Base.symdiff
Base.symdiff!(::IntSet, ::Integer)
Base.symdiff!(::IntSet, ::Any)
Base.symdiff!(::IntSet, ::IntSet)
Base.intersect!
Base.issubset
```

Completamente implementado por:

  * [`IntSet`](@ref)
  * [`Set`](@ref)

Parcialmente implementado por:

  * [`Array`](@ref)

## Dequeues

```@docs
Base.push!
Base.pop!(::Any)
Base.unshift!
Base.shift!
Base.insert!
Base.deleteat!
Base.splice!
Base.resize!
Base.append!
Base.prepend!
```

Completamente implementado por:

  * `Vector` (a.k.a. 1-dimensional [`Array`](@ref))
  * `BitVector` (a.k.a. 1-dimensional [`BitArray`](@ref))
