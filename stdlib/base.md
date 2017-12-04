# Essentials

## Introducción

La librería estándar de Julia contiene un rango de funciones y maros apropiadas para realizar 
computación científica y numérica, pero es también tan amplia como la de muchos lenguajes de 
programación de propósito general. También hay funcionalidad adicional disponible en una 
colección creciente de paquetes disponibles. Las funciones están agrupadas abajo por temas.

Algunas notas generales:

* Excepto para las funciones en los módulos predefinidos (`Pkg`, `Collections`, `Test` 
y `Profile`), todas las funciones documentadas aquí están disponibles directamente 
para ser usadas en programas.
* Para usar funciones de módulos, usar `import Module` para importar el módulo, y `Module.fn(x)` 
para usar las funciones.
* Alternativamente `using Module` importará todas las funciones exportadas por el módulo en 
el espacio de nombres actual.
* Por convenio, los nombres de funciones que acaban con un signo de admiración (`!`) 
modifican sus argumentos. Algunas funciones tienen las dos versiones (con y sin modificación 
de los argumentos).

## Arrancando

```@docs
Base.exit
Base.quit
Base.atexit
Base.atreplinit
Base.isinteractive
Base.whos
Base.summarysize
Base.edit(::AbstractString, ::Integer)
Base.edit(::Any)
Base.@edit
Base.less(::AbstractString)
Base.less(::Any)
Base.@less
Base.clipboard(::Any)
Base.clipboard()
Base.reload
Base.require
Base.compilecache
Base.__precompile__
Base.include
Base.include_string
Base.include_dependency
Base.Docs.apropos
Base.which(::Any, ::Any)
Base.which(::Symbol)
Base.@which
Base.methods
Base.methodswith
Base.@show
Base.versioninfo
Base.workspace
ans
```

## Todos los Objetos

```@docs
Core.:(===)
Core.isa
Base.isequal(::Any, ::Any)
Base.isequal(::Nullable, ::Nullable)
Base.isless
Base.isless(::Nullable, ::Nullable)
Base.ifelse
Base.lexcmp
Base.lexless
Core.typeof
Core.tuple
Base.ntuple
Base.object_id
Base.hash
Base.finalizer
Base.finalize
Base.copy
Base.deepcopy
Core.isdefined
Base.@isdefined
Base.convert
Base.promote
Base.oftype
Base.widen
Base.identity
```

## Tipos

```@docs
Base.supertype
Core.:(<:)
Base.:(>:)
Base.subtypes
Base.typemin
Base.typemax
Base.realmin
Base.realmax
Base.maxintfloat
Base.sizeof(::Type)
Base.eps(::Type{<:AbstractFloat})
Base.eps(::AbstractFloat)
Base.promote_type
Base.promote_rule
Core.getfield
Core.setfield!
Base.fieldoffset
Core.fieldtype
Base.isimmutable
Base.isbits
Base.isleaftype
Base.typejoin
Base.typeintersect
Base.Val
Base.Enums.@enum
Base.instances
```

## Funciones Genéricas

```@docs
Core.Function
Base.method_exists
Core.applicable
Core.invoke
Base.invokelatest
Base.:(|>)
Base.:(∘)
```

## Sintaxis

```@docs
Core.eval
Base.@eval
Base.evalfile
Base.esc
Base.@inbounds
Base.@inline
Base.@noinline
Base.@nospecialize
Base.gensym
Base.@gensym
Base.@polly
Base.parse(::Any, ::Any)
Base.parse(::Any)
```

## *Nullables*

```@docs
Base.Nullable
Base.get(::Nullable, ::Any)
Base.isnull
Base.unsafe_get
```

## Sistema

```@docs
Base.run
Base.spawn
Base.DevNull
Base.success
Base.process_running
Base.process_exited
Base.kill(::Base.Process, ::Integer)
Base.Sys.set_process_title
Base.Sys.get_process_title
Base.readandwrite
Base.ignorestatus
Base.detach
Base.Cmd
Base.setenv
Base.withenv
Base.pipeline(::Any, ::Any, ::Any, ::Any...)
Base.pipeline(::Base.AbstractCmd)
Base.Libc.gethostname
Base.getipaddr
Base.Libc.getpid
Base.Libc.time()
Base.time_ns
Base.tic
Base.toc
Base.toq
Base.@time
Base.@timev
Base.@timed
Base.@elapsed
Base.@allocated
Base.EnvHash
Base.ENV
Base.Sys.isunix
Base.Sys.isapple
Base.Sys.islinux
Base.Sys.isbsd
Base.Sys.iswindows
Base.Sys.windows_version
Base.@static
```

## Errores

```@docs
Base.error
Core.throw
Base.rethrow
Base.backtrace
Base.catch_backtrace
Base.assert
Base.@assert
Base.ArgumentError
Base.AssertionError
Core.BoundsError
Base.DimensionMismatch
Core.DivideError
Core.DomainError
Base.EOFError
Core.ErrorException
Core.InexactError
Core.InterruptException
Base.KeyError
Base.LoadError
Base.MethodError
Base.NullException
Core.OutOfMemoryError
Core.ReadOnlyMemoryError
Core.OverflowError
Base.ParseError
Base.ProcessExitedException
Core.StackOverflowError
Base.SystemError
Core.TypeError
Core.UndefRefError
Core.UndefVarError
Base.InitError
Base.retry
Base.ExponentialBackOff
```

## Eventos

```@docs
Base.Timer(::Function, ::Real, ::Real)
Base.Timer
Base.AsyncCondition
Base.AsyncCondition(::Function)
```

## Reflexión

```@docs
Base.module_name
Base.module_parent
Base.@__MODULE__
Base.fullname
Base.names
Core.nfields
Base.fieldnames
Base.fieldname
Base.fieldcount
Base.datatype_module
Base.datatype_name
Base.isconst
Base.function_name
Base.function_module(::Function)
Base.function_module(::Any, ::Any)
Base.functionloc(::Any, ::Any)
Base.functionloc(::Method)
Base.@functionloc
```

## Interioridades

```@docs
Base.gc
Base.gc_enable
Base.macroexpand
Base.@macroexpand
Base.@macroexpand1
Base.expand
Base.code_lowered
Base.@code_lowered
Base.code_typed
Base.@code_typed
Base.code_warntype
Base.@code_warntype
Base.code_llvm
Base.@code_llvm
Base.code_native
Base.@code_native
Base.precompile
```
