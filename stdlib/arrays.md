# [Arrays](@id lib-arrays)

## Constructores y Tipos

```@docs
Core.AbstractArray
Base.AbstractVector
Base.AbstractMatrix
Core.Array
Core.Array(::Any)
Base.Vector
Base.Vector(::Any)
Base.Matrix
Base.Matrix(::Any, ::Any)
Base.getindex(::Type, ::Any...)
Base.zeros
Base.ones
Base.BitArray
Base.BitArray(::Integer...)
Base.BitArray(::Any)
Base.trues
Base.falses
Base.fill
Base.fill!
Base.similar(::AbstractArray)
Base.similar(::Any, ::Tuple)
Base.eye
Base.linspace
Base.logspace
Base.Random.randsubseq
Base.Random.randsubseq!
```

## Funciones básicas

```@docs
Base.ndims
Base.size
Base.indices(::Any)
Base.indices(::AbstractArray, ::Any)
Base.length(::AbstractArray)
Base.eachindex
Base.linearindices
Base.IndexStyle
Base.countnz
Base.conj!
Base.stride
Base.strides
Base.ind2sub
Base.sub2ind
Base.LinAlg.checksquare
```

## Retransmisión y Vectorización

Ver también la [sintaxis de puntos para vectorizar funciones] (@ref man-vectorized); por ejemplo, `f. (args ...)` llama implícitamente a `broadcast(f, args...) `. En lugar de confiar en los métodos "vectorizados" de funciones como `sin` para operar en arrays, debe usar `sin.(A)` para vectorizar a través de `broadcast`.

```@docs
Base.broadcast
Base.Broadcast.broadcast!
Base.@__dot__
Base.Broadcast.broadcast_getindex
Base.Broadcast.broadcast_setindex!
```

## Indexación y Asignación

```@docs
Base.getindex(::AbstractArray, ::Any...)
Base.setindex!(::AbstractArray, ::Any, ::Any...)
Base.copy!(::AbstractArray, ::CartesianRange, ::AbstractArray, ::CartesianRange)
Base.isassigned
Base.Colon
Base.CartesianIndex
Base.CartesianRange
Base.to_indices
Base.checkbounds
Base.checkindex
```

## Vistas (SubArrays y otros tipos de vistas)

```@docs
Base.view
Base.@view
Base.@views
Base.parent
Base.parentindexes
Base.slicedim
Base.reinterpret
Base.reshape
Base.squeeze
Base.vec
```

## Concatenación and permutación

```@docs
Base.cat
Base.vcat
Base.hcat
Base.hvcat
Base.flipdim
Base.circshift
Base.circshift!
Base.circcopy!
Base.contains(::Function, ::Any, ::Any)
Base.find(::Any)
Base.find(::Function, ::Any)
Base.findn
Base.findnz
Base.findfirst(::Any)
Base.findfirst(::Any, ::Any)
Base.findfirst(::Function, ::Any)
Base.findlast(::Any)
Base.findlast(::Any, ::Any)
Base.findlast(::Function, ::Any)
Base.findnext(::Any, ::Integer)
Base.findnext(::Function, ::Any, ::Integer)
Base.findnext(::Any, ::Any, ::Integer)
Base.findprev(::Any, ::Integer)
Base.findprev(::Function, ::Any, ::Integer)
Base.findprev(::Any, ::Any, ::Integer)
Base.permutedims
Base.permutedims!
Base.PermutedDimsArray
Base.promote_shape
```

## Funciones de Arrays

```@docs
Base.accumulate(::Any, ::Any, ::Integer)
Base.accumulate!
Base.cumprod
Base.cumprod!
Base.cumsum
Base.cumsum!
Base.cumsum_kbn
Base.crc32c
Base.LinAlg.diff
Base.LinAlg.gradient
Base.repeat(::AbstractArray)
Base.rot180
Base.rotl90
Base.rotr90
Base.reducedim
Base.mapreducedim
Base.mapslices
Base.sum_kbn
```

## Combinatoria

```@docs
Base.Random.randperm
Base.Random.randperm!
Base.invperm
Base.isperm
Base.permute!(::Any, ::AbstractVector)
Base.ipermute!
Base.Random.randcycle
Base.Random.randcycle!
Base.Random.shuffle
Base.Random.shuffle!
Base.reverse
Base.reverseind
Base.reverse!
```

## BitArrays

[`BitArray`](@ref)s son matrices booleanas "compactas" eficientes en el uso del espacio, que almacenan un bit por valor booleano. Se pueden usar de forma similar a los arrays `Array{Bool}` (que almacenan un byte por valor booleano), y se pueden convertir a/desde este último a través de `Array(bitarray)` y `BitArray(array)`, respectivamente.

```@docs
Base.flipbits!
Base.rol!
Base.rol
Base.ror!
Base.ror
```

## [Matrices y Vectores *Sparse*](@id stdlib-sparse-arrays)

Los vectores y las matrices *sparse* soportan ampliamente el mismo conjunto de operaciones que sus contrapartidas densas. las siguientes funcioens son específicas para arrays *sparse*.

```@docs
Base.SparseArrays.SparseVector
Base.SparseArrays.SparseMatrixCSC
Base.SparseArrays.sparse
Base.SparseArrays.sparsevec
Base.SparseArrays.issparse
Base.full
Base.SparseArrays.nnz
Base.SparseArrays.spzeros
Base.SparseArrays.spones
Base.SparseArrays.speye(::Type, ::Integer, ::Integer)
Base.SparseArrays.speye(::SparseMatrixCSC)
Base.SparseArrays.spdiagm
Base.SparseArrays.sprand
Base.SparseArrays.sprandn
Base.SparseArrays.nonzeros
Base.SparseArrays.rowvals
Base.SparseArrays.nzrange
Base.SparseArrays.dropzeros!(::SparseMatrixCSC, ::Bool)
Base.SparseArrays.dropzeros(::SparseMatrixCSC, ::Bool)
Base.SparseArrays.dropzeros!(::SparseVector, ::Bool)
Base.SparseArrays.dropzeros(::SparseVector, ::Bool)
Base.SparseArrays.permute
Base.permute!{Tv, Ti, Tp <: Integer, Tq <: Integer}(::SparseMatrixCSC{Tv,Ti}, ::SparseMatrixCSC{Tv,Ti}, ::AbstractArray{Tp,1}, ::AbstractArray{Tq,1})
```
