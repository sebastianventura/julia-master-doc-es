# [Números](@id lib-numbers)

## Tipos Numéricos Estándar

### Tipos Numéricos Abstractos

```@docs
Core.Number
Core.Real
Core.AbstractFloat
Core.Integer
Core.Signed
Core.Unsigned
```

### Tipos Numéricos Concretos

```@docs
Core.Float16
Core.Float32
Core.Float64
Base.BigFloat
Core.Bool
Core.Int8
Core.UInt8
Core.Int16
Core.UInt16
Core.Int32
Core.UInt32
Core.Int64
Core.UInt64
Core.Int128
Core.UInt128
Base.BigInt
Base.Complex
Base.Rational
Base.Irrational
```

## Formatos de Datos

```@docs
Base.bin
Base.hex
Base.dec
Base.oct
Base.base
Base.digits
Base.digits!
Base.bits
Base.parse(::Type, ::Any, ::Any)
Base.tryparse
Base.big
Base.signed
Base.unsigned
Base.float(::Any)
Base.Math.significand
Base.Math.exponent
Base.complex(::Complex)
Base.bswap
Base.num2hex
Base.hex2num
Base.hex2bytes
Base.bytes2hex
```

## Constantes y Funciones de Números Generales

```@docs
Base.one
Base.oneunit
Base.zero
Base.pi
Base.im
Base.eu
Base.catalan
Base.eulergamma
Base.golden
Base.Inf
Base.Inf32
Base.Inf16
Base.NaN
Base.NaN32
Base.NaN16
Base.issubnormal
Base.isfinite
Base.isinf
Base.isnan
Base.iszero
Base.isone
Base.nextfloat
Base.prevfloat
Base.isinteger
Base.isreal
Core.Float32(::Any)
Core.Float64(::Any)
Base.GMP.BigInt(::Any)
Base.MPFR.BigFloat(::Any)
Base.Rounding.rounding
Base.Rounding.setrounding(::Type, ::Any)
Base.Rounding.setrounding(::Function, ::Type, ::RoundingMode)
Base.Rounding.get_zero_subnormals
Base.Rounding.set_zero_subnormals
```

### Enteros

```@docs
Base.count_ones
Base.count_zeros
Base.leading_zeros
Base.leading_ones
Base.trailing_zeros
Base.trailing_ones
Base.isodd
Base.iseven
```

## BigFloats

El tipo [`BigFloat`](@ref) implementa el punto flotante de precisión arbitraria usando la librería [GNU MPFR library](http://www.mpfr.org/).

```@docs
Base.precision
Base.MPFR.precision(::Type{BigFloat})
Base.MPFR.setprecision
Base.MPFR.BigFloat(x, prec::Int)
BigFloat(x::Union{Integer, AbstractFloat, String}, rounding::RoundingMode)
Base.MPFR.BigFloat(x, prec::Int, rounding::RoundingMode)
Base.MPFR.BigFloat(x::String)
```

## Números Aleatorios

La generación de números aleatorios en Julia utiliza la [librería Mersenne Twister](http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/SFMT/#dSFMT) a través de objetos `MersenneTwister`. Julia tiene un RNG global que es usado por defecto. Pueden conectarse otros tipos RNG heredando del tipo `AbstractRNG`; ellos pueden ser usados entonces para tener multiples flujos de numeros aleatorios. Ademas de `MersenneTwister`, Julia proporciona el tipo RNG `RandomDevice` que es un *wrapper* sobre la entropía proporcionada por el SO.

La mayoría de las funciones relacionadas con la generación aleatoria aceptan un `AbstractRNG` opcional como primer argumento,`rng`, que se predetermina al global si no se proporciona. Además, algunos de ellos aceptan opcionalmente especificaciones de dimensión `dims ...` (que pueden darse como una tupla) para generar matrices de valores aleatorios.

Un RNG de tipo `MersenneTwister` o `RandomDevice` puede generar números aleatorios de los siguientes tipos: [`Float16`](@ref), [`Float32`](@ref), [`Float64`](@ref), [`Bool`](@ref), [`Int8`](@ref), [`UInt8`](@ref), [`Int16`](@ref), [`UInt16`](@ref), [`Int32`](@ref), [`UInt32`](@ref), [`Int64`](@ref), [`UInt64`](@ref), [`Int128`](@ref), [`UInt128`](@ref), [`BigInt`](@ref)
(o números complejos de estos tipos). Los números aleatorio en punto flotante son generados uniformemente en ``[0, 1)``. Como `BigInt` representa números sin límite, el intervalo debe ser especificado (por ejemplo, `rand(big(1:6))`).

```@docs
Base.Random.srand
Base.Random.MersenneTwister
Base.Random.RandomDevice
Base.Random.rand
Base.Random.rand!
Base.Random.bitrand
Base.Random.randn
Base.Random.randn!
Base.Random.randexp
Base.Random.randexp!
Base.Random.randjump
```
