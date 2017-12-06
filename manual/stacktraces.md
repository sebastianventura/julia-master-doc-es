# Stack Traces

El módulo `StackTraces` proporciona simples seguimientos de pila que son legibles para los humanos y fáciles de usar mediante programación.

## Viendo un rastro de pila

La función principal utilizada para obtener un seguimiento de pila es [`stacktrace()`](@ref):

```julia-repl
julia> stacktrace()
4-element Array{StackFrame,1}:
 eval(::Module, ::Any) at boot.jl:236
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
```

Llamar a [`stacktrace()`](@ref) devuelve un vector de [`StackFrame`](@ref)s. Para facilitar el uso, el alias [`StackTrace`](@ref) se puede usar en lugar de `Vector{StackFrame}`. (Los ejemplos con `[...]` indican que la salida puede variar dependiendo de cómo se ejecuta el código).

```julia-repl
julia> example() = stacktrace()
example (generic function with 1 method)

julia> example()
5-element Array{StackFrame,1}:
 example() at REPL[1]:1
 eval(::Module, ::Any) at boot.jl:236
[...]

julia> @noinline child() = stacktrace()
child (generic function with 1 method)

julia> @noinline parent() = child()
parent (generic function with 1 method)

julia> grandparent() = parent()
grandparent (generic function with 1 method)

julia> grandparent()
7-element Array{StackFrame,1}:
 child() at REPL[3]:1
 parent() at REPL[4]:1
 grandparent() at REPL[5]:1
[...]
```

Tenga en cuenta que cuando llama a [`stacktrace()`](@ref) normalmente verá un marco con `eval (...) en boot.jl`. Al invocar [`stacktrace()`](@ref) desde el REPL, también tendrá algunos fotogramas adicionales en la pila de `REPL.jl`, que generalmente se ve así:

```julia-repl
julia> example() = stacktrace()
example (generic function with 1 method)

julia> example()
5-element Array{StackFrame,1}:
 example() at REPL[1]:1
 eval(::Module, ::Any) at boot.jl:236
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
```

## Extracting useful information

Cada [`StackFrame`](@ref) contiene el nombre de la función, el nombre del archivo, el número de línea, la información lambda, un indicador que indica si el marco ha sido insertado, un indicador que indica si es una función C (por defecto las funciones C no aparecen en el seguimiento de la pila) y una representación entera del puntero devuelto por [`backtrace()`](@ref):

```julia-repl
julia> top_frame = stacktrace()[1]
eval(::Module, ::Any) at boot.jl:236

julia> top_frame.func
:eval

julia> top_frame.file
Symbol("./boot.jl")

julia> top_frame.line
236

julia> top_frame.linfo
Nullable{Core.MethodInstance}(MethodInstance for eval(::Module, ::Any))

julia> top_frame.inlined
false

julia> top_frame.from_c
false
```

```julia-repl
julia> top_frame.pointer
0x00007f390d152a59
```

Esto hace que la información de seguimiento de pila esté disponible programáticamente para el registro, el manejo de errores y más.

## Manejo de Errores

Si bien tener acceso fácil a la información sobre el estado actual de la pila de llamadas puede ser útil en muchos lugares, la aplicación más obvia es la gestión de errores y la depuración.

```julia-repl
julia> @noinline bad_function() = undeclared_variable
bad_function (generic function with 1 method)

julia> @noinline example() = try
           bad_function()
       catch
           stacktrace()
       end
example (generic function with 1 method)

julia> example()
5-element Array{StackFrame,1}:
 example() at REPL[2]:4
 eval(::Module, ::Any) at boot.jl:236
[...]
```

Puede observar que en el ejemplo anterior, el primer marco apila puntos en la línea 4, donde se llama a [`stacktrace()`](@ref), en lugar de a la línea 2, donde se llama *bad_function* y el marco de `bad_function` falta por completo. Esto es comprensible, dado que [`stacktrace()`](@ref) se llama desde el contexto de *catch*. Si bien en este ejemplo es bastante fácil encontrar el origen real del error, en casos complejos, rastrear el origen del error no es trivial.

Esto se puede remediar llamando a [`catch_stacktrace()`](@ref) en lugar de [`stacktrace ()`] (@ref). En lugar de devolver la información de la pila de llamadas para el contexto actual, [`catch_stacktrace()`](@ref) devuelve la información de la pila para el contexto de la excepción más reciente:

```julia-repl
julia> @noinline bad_function() = undeclared_variable
bad_function (generic function with 1 method)

julia> @noinline example() = try
           bad_function()
       catch
           catch_stacktrace()
       end
example (generic function with 1 method)

julia> example()
6-element Array{StackFrame,1}:
 bad_function() at REPL[1]:1
 example() at REPL[2]:2
[...]
```

Nótese que la traza de la pila indica ahora el número de línea apropiado y el marco perdido.

```julia-repl
julia> @noinline child() = error("Whoops!")
child (generic function with 1 method)

julia> @noinline parent() = child()
parent (generic function with 1 method)

julia> @noinline function grandparent()
           try
               parent()
           catch err
               println("ERROR: ", err.msg)
               catch_stacktrace()
           end
       end
grandparent (generic function with 1 method)

julia> grandparent()
ERROR: Whoops!
7-element Array{StackFrame,1}:
 child() at REPL[1]:1
 parent() at REPL[2]:1
 grandparent() at REPL[3]:3
[...]
```

## Comparación con [`backtrace()`](@ref)

Una llamada a [`backtrace ()`] (@ ref) devuelve un vector de `Ptr{Void}`, que puede pasarse luego a [`stacktrace()`](@ref) para la traducción:

```julia-repl
julia> trace = backtrace()
21-element Array{Ptr{Void},1}:
 Ptr{Void} @0x00007f10049d5b2f
 Ptr{Void} @0x00007f0ffeb4d29c
 Ptr{Void} @0x00007f0ffeb4d2a9
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f10049a92be
 Ptr{Void} @0x00007f10049a823a
 Ptr{Void} @0x00007f10049a9fb0
 Ptr{Void} @0x00007f10049aa718
 Ptr{Void} @0x00007f10049c0d5e
 Ptr{Void} @0x00007f10049a3286
 Ptr{Void} @0x00007f0ffe9ba3ba
 Ptr{Void} @0x00007f0ffe9ba3d0
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f0ded34583d
 Ptr{Void} @0x00007f0ded345a87
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f0ded34308f
 Ptr{Void} @0x00007f0ded343320
 Ptr{Void} @0x00007f1004993fe7
 Ptr{Void} @0x00007f10049aeb67
 Ptr{Void} @0x0000000000000000

julia> stacktrace(trace)
5-element Array{StackFrame,1}:
 backtrace() at error.jl:46
 eval(::Module, ::Any) at boot.jl:236
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
```

Observe que el vector devuelto por [`backtrace()`](@ref) tenía 21 punteros, mientras que el vector devuelto por [`stacktrace ()`](@ref) solo tiene 5. Esto es porque, de forma predeterminada, [`stacktrace()`](@ref) elimina cualquier función C de nivel inferior de la pila. Si desea incluir cuadros de pila de llamadas C, puede hacerlo así:

```julia-repl
julia> stacktrace(trace, true)
27-element Array{StackFrame,1}:
 jl_backtrace_from_here at stackwalk.c:103
 backtrace() at error.jl:46
 backtrace() at sys.so:?
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 do_call at interpreter.c:75
 eval at interpreter.c:215
 eval_body at interpreter.c:519
 jl_interpret_toplevel_thunk at interpreter.c:664
 jl_toplevel_eval_flex at toplevel.c:592
 jl_toplevel_eval_in at builtins.c:614
 eval(::Module, ::Any) at boot.jl:236
 eval(::Module, ::Any) at sys.so:?
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 eval_user_input(::Any, ::Base.REPL.REPLBackend) at REPL.jl:66
 ip:0x7f1c707f1846
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 macro expansion at REPL.jl:97 [inlined]
 (::Base.REPL.##1#2{Base.REPL.REPLBackend})() at event.jl:73
 ip:0x7f1c707ea1ef
 jl_call_method_internal at julia_internal.h:248 [inlined]
 jl_apply_generic at gf.c:2215
 jl_apply at julia.h:1411 [inlined]
 start_task at task.c:261
 ip:0xffffffffffffffff
```

Los punteros individuales devueltos por [`backtrace()`](@ref) se pueden traducir a [`StackFrame`](@ref) s pasándolos a [`StackTraces.lookup()`](@ref):

```julia-repl
julia> pointer = backtrace()[1];

julia> frame = StackTraces.lookup(pointer)
1-element Array{StackFrame,1}:
 jl_backtrace_from_here at stackwalk.c:103

julia> println("The top frame is from $(frame[1].func)!")
The top frame is from jl_backtrace_from_here!
```
