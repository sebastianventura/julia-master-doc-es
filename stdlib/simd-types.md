# Soporte SIMD

El tipo `VecElement{T}` está pensado para construir librerías de operaciones SIMD operations. El uso práctico de él requiere usar `llvmcall`. El tipo está definido como:

```julia
struct VecElement{T}
    value::T
end
```

Él tiene una regla de compilación especial: una tupla homogénea de `VecElement{T}` se corresponde con un tipo `vector` LLVM cuando `T` un tipo de bits primitivo y la longitud de la tupla está en el conjunto {2-6,8-10,16}.

En `-O3`, el compilador *podría* automáticamente vectorizar operaciones sobre tales tuplas. Por ejemplo, el siguiente programa, cuando se compila con `julia -O3` genera dos instrucciones de adición SIMD (`addps`) sobre los sistemas x86:

```julia
const m128 = NTuple{4,VecElement{Float32}}

function add(a::m128, b::m128)
    (VecElement(a[1].value+b[1].value),
     VecElement(a[2].value+b[2].value),
     VecElement(a[3].value+b[3].value),
     VecElement(a[4].value+b[4].value))
end

triple(c::m128) = add(add(c,c),c)

code_native(triple,(m128,))
```

Sin embargo, dado que no se puede confiar en la vectorización automática, el uso futuro se realizará principalmente a través de bibliotecas que usen `llvmcall`.
