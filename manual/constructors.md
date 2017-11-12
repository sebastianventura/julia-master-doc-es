# [Constructors](@id man-constructors)

Los constructores [^1] son funciones que crean nuevos objetos (específicamente instancias de tipos compuestos). En Julia, los objetos también sirven como funciones constructor: ellos crean instancias de sí mismos cuando se aplican a una dupla de argumentos como una función. Esto se mencionó brevemente cuando se habló de tipos compuestos. Por ejemplo:

```jldoctest footype
julia> struct Foo
           bar
           baz
       end

julia> foo = Foo(1, 2)
Foo(1, 2)

julia> foo.bar
1

julia> foo.baz
2
```

Para muchos tipos, formar nuevos objetos enlazando valores de campo juntos es todo se necesita para crear instancias. Hay, sin embargo, casos donde se requiere más funcionalidad cuando se crean objetos compuestos. Algunas invariantes deben ser forzadas, bien chequeando argumentos o transformándolos. Las [estructuras de datos recursivas](https://en.wikipedia.org/wiki/Recursion_%28computer_science%29#Recursive_data_structures_.28structural_recursion.29),
especialmente aquellas que pueden ser auto referenciadas frecuentemente, no pueden construirse limpiamente sin que primero sean creadas en un estado incompleto y después sean alteradas programáticamente para ser completadas, como un paso separado de la creación del objeto. Algunas veces, es conveniente ser capaz de construir objetos con menos o diferentes tipos de parámetros que el número de campos que tiene. El sistema Julia para construcción de objetos cubre estos casos y más.

[^1]:
    Nomenclature: while the term "constructor" generally refers to the entire function which constructs
    objects of a type, it is common to abuse terminology slightly and refer to specific constructor
    methods as "constructors". In such situations, it is generally clear from context that the term
    is used to mean "constructor method" rather than "constructor function", especially as it is often
    used in the sense of singling out a particular method of the constructor from all of the others.

## Métodos constructores externos

Un constructor es como cualquier otro función en Julia en que es su comportamiento global está definido por el comportamiento combinado de sus métodos. Según esto, se puede añadir funcionalidad a ún constructor simplemente definiendo nuevos métodos. Por ejemplo, supóngase que se desea añadir un método constructor para objetos `Foo` que tomar un argumento y usa el valor dado para los dos campos que presentan `baz` y `bar`. Esto es sencillo::

```jldoctest footype
julia> Foo(x) = Foo(x,x)
Foo

julia> Foo(1)
Foo(1, 1)
```

Podría también añadirse un constructor `Foo` sin argumentos que proporciona valores por defecto para los campos `bar` y `baz`:

```jldoctest footype
julia> Foo() = Foo(0)
Foo

julia> Foo()
Foo(0, 0)
```

Aquí, el método constructor sin argumentos llama al método constructor con un argumento, que a su vez llama al método constructor de dos argumentos proporcionado automáticamente. Por razones que se aclararán pronto, los metodos constructor adicionales declarados como métodos formales como éstos se denominan *métodos constructores externos*. Los métodos constructores externos sólo puede crear una nueva instancia llamando a otro método constructor, tal como los  proporcionados automáticamente por defecto.

## Inner Constructor Methods

Métodos constructores internos

Aunque los constructores externos resuelven con éxito el problema de proporcionar métodos adicionales para construir objetos, ellos fallan en los otros dos casos de uso mencionados en la introducción este capítulo: forzar invariantes y permitir la construcción de objetos autorreferenciales. Para estos problemas, se necesitan los *métodos constructores internos*. Un método constructor interno es parecido a uno externo, con dos diferencias:

1. Se declara dentro del bloque de la declaración del tipo, el lugar donde fuera como los métodos normales.
2. Tiene acceso a una función especial, existente totalmente, llamada `new` que crea objetos del tipo del bloque.

Poner ejemplo, supóngase que uno quiere declarar un tipo que almacene un par de elementos reales, sujetos a la restricción de que el primer número no es mayor que el segundo. Uno podría declararlo así:

```jldoctest pairtype
julia> struct OrderedPair
           x::Real
           y::Real
           OrderedPair(x,y) = x > y ? error("out of order") : new(x,y)
       end

```

Ahora sólo pueden construirse objetos `OrderedPair` tales que `x <= y`:

```jldoctest pairtype
julia> OrderedPair(1, 2)
OrderedPair(1, 2)

julia> OrderedPair(2,1)
ERROR: out of order
Stacktrace:
 [1] OrderedPair(::Int64, ::Int64) at ./none:4
```

If the type were declared `mutable`, you could reach in and directly change the field values to
violate this invariant, but messing around with an object's internals uninvited is considered poor form.
You (or someone else) can also provide additional outer constructor methods at any later point, but
once a type is declared, there is no way to add more inner constructor methods. Since outer constructor
methods can only create objects by calling other constructor methods, ultimately, some inner constructor
must be called to create an object. This guarantees that all objects of the declared type must come into
existence by a call to one of the inner constructor methods provided with the type, thereby giving
some degree of enforcement of a type's invariants.

If any inner constructor method is defined, no default constructor method is provided: it is presumed
that you have supplied yourself with all the inner constructors you need. The default constructor
is equivalent to writing your own inner constructor method that takes all of the object's fields
as parameters (constrained to be of the correct type, if the corresponding field has a type),
and passes them to `new`, returning the resulting object:

```jldoctest
julia> struct Foo
           bar
           baz
           Foo(bar,baz) = new(bar,baz)
       end

```

This declaration has the same effect as the earlier definition of the `Foo` type without an explicit
inner constructor method. The following two types are equivalent -- one with a default constructor,
the other with an explicit constructor:

```jldoctest
julia> struct T1
           x::Int64
       end

julia> struct T2
           x::Int64
           T2(x) = new(x)
       end

julia> T1(1)
T1(1)

julia> T2(1)
T2(1)

julia> T1(1.0)
T1(1)

julia> T2(1.0)
T2(1)
```

It is considered good form to provide as few inner constructor methods as possible: only those
taking all arguments explicitly and enforcing essential error checking and transformation. Additional
convenience constructor methods, supplying default values or auxiliary transformations, should
be provided as outer constructors that call the inner constructors to do the heavy lifting. This
separation is typically quite natural.

## Incomplete Initialization

The final problem which has still not been addressed is construction of self-referential objects,
or more generally, recursive data structures. Since the fundamental difficulty may not be immediately
obvious, let us briefly explain it. Consider the following recursive type declaration:

```jldoctest selfrefer
julia> mutable struct SelfReferential
           obj::SelfReferential
       end

```

This type may appear innocuous enough, until one considers how to construct an instance of it.
If `a` is an instance of `SelfReferential`, then a second instance can be created by the call:

```julia-repl
julia> b = SelfReferential(a)
```

But how does one construct the first instance when no instance exists to provide as a valid value
for its `obj` field? The only solution is to allow creating an incompletely initialized instance
of `SelfReferential` with an unassigned `obj` field, and using that incomplete instance as a valid
value for the `obj` field of another instance, such as, for example, itself.

To allow for the creation of incompletely initialized objects, Julia allows the `new` function
to be called with fewer than the number of fields that the type has, returning an object with
the unspecified fields uninitialized. The inner constructor method can then use the incomplete
object, finishing its initialization before returning it. Here, for example, we take another crack
at defining the `SelfReferential` type, with a zero-argument inner constructor returning instances
having `obj` fields pointing to themselves:

```jldoctest selfrefer2
julia> mutable struct SelfReferential
           obj::SelfReferential
           SelfReferential() = (x = new(); x.obj = x)
       end

```

We can verify that this constructor works and constructs objects that are, in fact, self-referential:

```jldoctest selfrefer2
julia> x = SelfReferential();

julia> x === x
true

julia> x === x.obj
true

julia> x === x.obj.obj
true
```

Although it is generally a good idea to return a fully initialized object from an inner constructor,
incompletely initialized objects can be returned:

```jldoctest incomplete
julia> mutable struct Incomplete
           xx
           Incomplete() = new()
       end

julia> z = Incomplete();
```

While you are allowed to create objects with uninitialized fields, any access to an uninitialized
reference is an immediate error:

```jldoctest incomplete
julia> z.xx
ERROR: UndefRefError: access to undefined reference
```

This avoids the need to continually check for `null` values. However, not all object fields are
references. Julia considers some types to be "plain data", meaning all of their data is self-contained
and does not reference other objects. The plain data types consist of primitive types (e.g. `Int`)
and immutable structs of other plain data types. The initial contents of a plain data type is
undefined:

```julia-repl
julia> struct HasPlain
           n::Int
           HasPlain() = new()
       end

julia> HasPlain()
HasPlain(438103441441)
```

Arrays of plain data types exhibit the same behavior.

You can pass incomplete objects to other functions from inner constructors to delegate their completion:

```jldoctest
julia> mutable struct Lazy
           xx
           Lazy(v) = complete_me(new(), v)
       end
```

As with incomplete objects returned from constructors, if `complete_me` or any of its callees
try to access the `xx` field of the `Lazy` object before it has been initialized, an error will
be thrown immediately.

## Parametric Constructors

Parametric types add a few wrinkles to the constructor story. Recall from [Parametric Types](@ref)
that, by default, instances of parametric composite types can be constructed either with explicitly
given type parameters or with type parameters implied by the types of the arguments given to the
constructor. Here are some examples:

```jldoctest parametric
julia> struct Point{T<:Real}
           x::T
           y::T
       end

julia> Point(1,2) ## implicit T ##
Point{Int64}(1, 2)

julia> Point(1.0,2.5) ## implicit T ##
Point{Float64}(1.0, 2.5)

julia> Point(1,2.5) ## implicit T ##
ERROR: MethodError: no method matching Point(::Int64, ::Float64)
Closest candidates are:
  Point(::T<:Real, !Matched::T<:Real) where T<:Real at none:2

julia> Point{Int64}(1, 2) ## explicit T ##
Point{Int64}(1, 2)

julia> Point{Int64}(1.0,2.5) ## explicit T ##
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int64}, ::Float64) at ./float.jl:680
 [2] Point{Int64}(::Float64, ::Float64) at ./none:2

julia> Point{Float64}(1.0, 2.5) ## explicit T ##
Point{Float64}(1.0, 2.5)

julia> Point{Float64}(1,2) ## explicit T ##
Point{Float64}(1.0, 2.0)
```

As you can see, for constructor calls with explicit type parameters, the arguments are converted
to the implied field types: `Point{Int64}(1,2)` works, but `Point{Int64}(1.0,2.5)` raises an
[`InexactError`](@ref) when converting `2.5` to [`Int64`](@ref). When the type is implied
by the arguments to the constructor call, as in `Point(1,2)`, then the types of the
arguments must agree -- otherwise the `T` cannot be determined -- but any pair of real
arguments with matching type may be given to the generic `Point` constructor.

What's really going on here is that `Point`, `Point{Float64}` and `Point{Int64}` are all different
constructor functions. In fact, `Point{T}` is a distinct constructor function for each type `T`.
Without any explicitly provided inner constructors, the declaration of the composite type `Point{T<:Real}`
automatically provides an inner constructor, `Point{T}`, for each possible type `T<:Real`, that
behaves just like non-parametric default inner constructors do. It also provides a single general
outer `Point` constructor that takes pairs of real arguments, which must be of the same type.
This automatic provision of constructors is equivalent to the following explicit declaration:

```jldoctest parametric2
julia> struct Point{T<:Real}
           x::T
           y::T
           Point{T}(x,y) where {T<:Real} = new(x,y)
       end

julia> Point(x::T, y::T) where {T<:Real} = Point{T}(x,y);
```

Notice that each definition looks like the form of constructor call that it handles.
The call `Point{Int64}(1,2)` will invoke the definition `Point{T}(x,y)` inside the
`type` block.
The outer constructor declaration, on the other hand, defines a
method for the general `Point` constructor which only applies to pairs of values of the same real
type. This declaration makes constructor calls without explicit type parameters, like `Point(1,2)`
and `Point(1.0,2.5)`, work. Since the method declaration restricts the arguments to being of the
same type, calls like `Point(1,2.5)`, with arguments of different types, result in "no method"
errors.

Suppose we wanted to make the constructor call `Point(1,2.5)` work by "promoting" the integer
value `1` to the floating-point value `1.0`. The simplest way to achieve this is to define the
following additional outer constructor method:

```jldoctest parametric2
julia> Point(x::Int64, y::Float64) = Point(convert(Float64,x),y);
```

This method uses the [`convert()`](@ref) function to explicitly convert `x` to [`Float64`](@ref)
and then delegates construction to the general constructor for the case where both arguments are
[`Float64`](@ref). With this method definition what was previously a [`MethodError`](@ref) now
successfully creates a point of type `Point{Float64}`:

```jldoctest parametric2
julia> Point(1,2.5)
Point{Float64}(1.0, 2.5)

julia> typeof(ans)
Point{Float64}
```

However, other similar calls still don't work:

```jldoctest parametric2
julia> Point(1.5,2)
ERROR: MethodError: no method matching Point(::Float64, ::Int64)
Closest candidates are:
  Point(::T<:Real, !Matched::T<:Real) where T<:Real at none:1
```

For a more general way to make all such calls work sensibly, see [Conversion and Promotion](@ref conversion-and-promotion).
At the risk of spoiling the suspense, we can reveal here that all it takes is the following outer
method definition to make all calls to the general `Point` constructor work as one would expect:

```jldoctest parametric2
julia> Point(x::Real, y::Real) = Point(promote(x,y)...);
```

The `promote` function converts all its arguments to a common type -- in this case [`Float64`](@ref).
With this method definition, the `Point` constructor promotes its arguments the same way that
numeric operators like [`+`](@ref) do, and works for all kinds of real numbers:

```jldoctest parametric2
julia> Point(1.5,2)
Point{Float64}(1.5, 2.0)

julia> Point(1,1//2)
Point{Rational{Int64}}(1//1, 1//2)

julia> Point(1.0,1//2)
Point{Float64}(1.0, 0.5)
```

Thus, while the implicit type parameter constructors provided by default in Julia are fairly strict,
it is possible to make them behave in a more relaxed but sensible manner quite easily. Moreover,
since constructors can leverage all of the power of the type system, methods, and multiple dispatch,
defining sophisticated behavior is typically quite simple.

## Case Study: Rational

Perhaps the best way to tie all these pieces together is to present a real world example of a
parametric composite type and its constructor methods. To that end, here is the (slightly modified) beginning of [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl),
which implements Julia's [Rational Numbers](@ref):

```jldoctest rational
julia> struct OurRational{T<:Integer} <: Real
           num::T
           den::T
           function OurRational{T}(num::T, den::T) where T<:Integer
               if num == 0 && den == 0
                    error("invalid rational: 0//0")
               end
               g = gcd(den, num)
               num = div(num, g)
               den = div(den, g)
               new(num, den)
           end
       end

julia> OurRational(n::T, d::T) where {T<:Integer} = OurRational{T}(n,d)
OurRational

julia> OurRational(n::Integer, d::Integer) = OurRational(promote(n,d)...)
OurRational

julia> OurRational(n::Integer) = OurRational(n,one(n))
OurRational

julia> //(n::Integer, d::Integer) = OurRational(n,d)
// (generic function with 1 method)

julia> //(x::OurRational, y::Integer) = x.num // (x.den*y)
// (generic function with 2 methods)

julia> //(x::Integer, y::OurRational) = (x*y.den) // y.num
// (generic function with 3 methods)

julia> //(x::Complex, y::Real) = complex(real(x)//y, imag(x)//y)
// (generic function with 4 methods)

julia> //(x::Real, y::Complex) = x*y'//real(y*y')
// (generic function with 5 methods)

julia> function //(x::Complex, y::Complex)
           xy = x*y'
           yy = real(y*y')
           complex(real(xy)//yy, imag(xy)//yy)
       end
// (generic function with 6 methods)
```

The first line -- `struct OurRational{T<:Integer} <: Real` -- declares that `OurRational` takes one
type parameter of an integer type, and is itself a real type. The field declarations `num::T`
and `den::T` indicate that the data held in a `OurRational{T}` object are a pair of integers of type
`T`, one representing the rational value's numerator and the other representing its denominator.

Now things get interesting. `OurRational` has a single inner constructor method which checks that
both of `num` and `den` aren't zero and ensures that every rational is constructed in "lowest
terms" with a non-negative denominator. This is accomplished by dividing the given numerator and
denominator values by their greatest common divisor, computed using the `gcd` function. Since
`gcd` returns the greatest common divisor of its arguments with sign matching the first argument
(`den` here), after this division the new value of `den` is guaranteed to be non-negative. Because
this is the only inner constructor for `OurRational`, we can be certain that `OurRational` objects are
always constructed in this normalized form.

`OurRational` also provides several outer constructor methods for convenience. The first is the "standard"
general constructor that infers the type parameter `T` from the type of the numerator and denominator
when they have the same type. The second applies when the given numerator and denominator values
have different types: it promotes them to a common type and then delegates construction to the
outer constructor for arguments of matching type. The third outer constructor turns integer values
into rationals by supplying a value of `1` as the denominator.

Following the outer constructor definitions, we have a number of methods for the [`//`](@ref)
operator, which provides a syntax for writing rationals. Before these definitions, [`//`](@ref)
is a completely undefined operator with only syntax and no meaning. Afterwards, it behaves just
as described in [Rational Numbers](@ref) -- its entire behavior is defined in these few lines.
The first and most basic definition just makes `a//b` construct a `OurRational` by applying the
`OurRational` constructor to `a` and `b` when they are integers. When one of the operands of [`//`](@ref)
is already a rational number, we construct a new rational for the resulting ratio slightly differently;
this behavior is actually identical to division of a rational with an integer.
Finally, applying
[`//`](@ref) to complex integral values creates an instance of `Complex{OurRational}` -- a complex
number whose real and imaginary parts are rationals:

```jldoctest rational
julia> ans = (1 + 2im)//(1 - 2im);

julia> typeof(ans)
Complex{OurRational{Int64}}

julia> ans <: Complex{OurRational}
false
```

Thus, although the [`//`](@ref) operator usually returns an instance of `OurRational`, if either
of its arguments are complex integers, it will return an instance of `Complex{OurRational}` instead.
The interested reader should consider perusing the rest of [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl):
it is short, self-contained, and implements an entire basic Julia type.

## [Constructors and Conversion](@id constructors-and-conversion)

Constructors `T(args...)` in Julia are implemented like other callable objects: methods are added
to their types. The type of a type is `Type`, so all constructor methods are stored in the method
table for the `Type` type. This means that you can declare more flexible constructors, e.g. constructors
for abstract types, by explicitly defining methods for the appropriate types.

However, in some cases you could consider adding methods to `Base.convert` *instead* of defining
a constructor, because Julia falls back to calling [`convert()`](@ref) if no matching constructor
is found. For example, if no constructor `T(args...) = ...` exists `Base.convert(::Type{T}, args...) = ...`
is called.

`convert` is used extensively throughout Julia whenever one type needs to be converted to another
(e.g. in assignment, [`ccall`](@ref), etcetera), and should generally only be defined (or successful)
if the conversion is lossless.  For example, `convert(Int, 3.0)` produces `3`, but `convert(Int, 3.2)`
throws an `InexactError`.  If you want to define a constructor for a lossless conversion from
one type to another, you should probably define a `convert` method instead.

On the other hand, if your constructor does not represent a lossless conversion, or doesn't represent
"conversion" at all, it is better to leave it as a constructor rather than a `convert` method.
For example, the `Array{Int}()` constructor creates a zero-dimensional `Array` of the type `Int`,
but is not really a "conversion" from `Int` to an `Array`.

## Outer-only constructors

As we have seen, a typical parametric type has inner constructors that are called when type parameters
are known; e.g. they apply to `Point{Int}` but not to `Point`. Optionally, outer constructors
that determine type parameters automatically can be added, for example constructing a `Point{Int}`
from the call `Point(1,2)`. Outer constructors call inner constructors to do the core work of
making an instance. However, in some cases one would rather not provide inner constructors, so
that specific type parameters cannot be requested manually.

For example, say we define a type that stores a vector along with an accurate representation of
its sum:

```jldoctest
julia> struct SummedArray{T<:Number,S<:Number}
           data::Vector{T}
           sum::S
       end

julia> SummedArray(Int32[1; 2; 3], Int32(6))
SummedArray{Int32,Int32}(Int32[1, 2, 3], 6)
```

The problem is that we want `S` to be a larger type than `T`, so that we can sum many elements
with less information loss. For example, when `T` is [`Int32`](@ref), we would like `S` to
be [`Int64`](@ref). Therefore we want to avoid an interface that allows the user to construct
instances of the type `SummedArray{Int32,Int32}`. One way to do this is to provide a
constructor only for `SummedArray`, but inside the `type` definition block to suppress
generation of default constructors:

```jldoctest
julia> struct SummedArray{T<:Number,S<:Number}
           data::Vector{T}
           sum::S
           function SummedArray(a::Vector{T}) where T
               S = widen(T)
               new{T,S}(a, sum(S, a))
           end
       end

julia> SummedArray(Int32[1; 2; 3], Int32(6))
ERROR: MethodError: no method matching SummedArray(::Array{Int32,1}, ::Int32)
Closest candidates are:
  SummedArray(::Array{T,1}) where T at none:5
```

This constructor will be invoked by the syntax `SummedArray(a)`. The syntax `new{T,S}` allows
specifying parameters for the type to be constructed, i.e. this call will return a `SummedArray{T,S}`.
`new{T,S}` can be used in any constructor definition, but for convenience the parameters
to `new{}` are automatically derived from the type being constructed when possible.
