# Empezando

La instalación de Julia es sencilla, ya sea utilizando binarios precompilados o compilando desde la fuente. Descargue e instale Julia siguiendo las instrucciones en [https://julialang.org/downloads/](https://julialang.org/downloads/).

La forma más fácil de aprender y experimentar con Julia es iniciando una sesión interactiva (también conocida como *read-eval-print loop* o "REPL") haciendo doble clic en el ejecutable de Julia o ejecutando `julia` desde la línea de órdenes:

```
$ julia
               _
   _       _ _(_)_     |  A fresh approach to technical computing
  (_)     | (_) (_)    |  Documentation: https://docs.julialang.org
   _ _   _| |_  __ _   |  Type "?help" for help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 0.6.0-dev+2440 (2016-02-01 02:22 UTC)
 _/ |\__'_|_|_|\__'_|  |  Commit 2bb94d6 (11 days old master)
|__/                   |  x86_64-apple-darwin13.1.0

julia> 1 + 2
3

julia> ans
3
```

Para salir de la sesión interactiva, escriba `^D` (la tecla de control junto con la tecla `D`) o escriba
`quit()`. Cuando se ejecuta en modo interactivo, Julia muestra un banner y solicita al usuario la entrada. 
Una vez que el usuario ha introducido una expresión completa, como `1 + 2`, y pulsa *Enter*, la sesión 
interactiva evalúa la expresión y muestra su valor. Si se introduce una expresión en una sesión interactiva 
con un punto y coma al final, no se muestra su valor. La variable `ans` está enlazada al valor de la última 
expresión evaluada, sea mostrada o no. La variable `ans` sólo está enlazada a las sesiones interactivas, 
no cuando el código Julia se ejecuta de otras maneras.

Para evaluar expresiones escritas en un archivo de origen `file.jl`, escriba `include ("file.jl")`.

Para ejecutar código en un archivo de forma no interactiva, puede darlo como el primer argumento al mandato Julia:

```
$ julia script.jl arg1 arg2...
```

Como indica el ejemplo, los siguientes argumentos de línea de órdenes de Julia se toman como argumentos de 
línea de comandos al programa `script.jl` del programa, pasado a través de la constante global `ARGS`. El 
nombre del propio *script* se pasa como la variable global `PROGRAM_FILE`. Tenga en cuenta que `ARGS` 
también se establece cuando se da el código de script usando la opción `-e` en la línea de órdenes (vea 
la salida de ayuda de `julia` más abajo) pero `PROGRAM_FILE` estará vacío. Por ejemplo, para imprimir 
los argumentos que se le dan a un script, puede hacer esto:

```
$ julia -e 'println(PROGRAM_FILE); for x in ARGS; println(x); end' foo bar

foo
bar
```

O puede poner ese código en un script y ejecutarlo:

```
$ echo 'println(PROGRAM_FILE); for x in ARGS; println(x); end' > script.jl
$ julia script.jl foo bar
script.jl
foo
bar
```

El delimitador `--` puede usarse para separar argumentos en línea de mandatos al fichero del 
*script* a los argumentos de Julia:

```
$ julia --color=yes -O -- foo.jl arg1 arg2..
```

Julia se puede iniciar en modo paralelo con las opciones `-p` o `--machinefile`. `-p n` pondrá en 
marcha un `n` procesos *worker* adicionales, mientras que `--machinefile archivo` iniciará un 
*worker* para cada línea en el archivo de archivo. Las máquinas definidas en el archivo deben ser 
accesibles a través de un login ssh sin contraseña, con Julia instalado en la misma ubicación que 
el host actual. Cada definición de máquina toma la forma 
`[count *] [user @] host [: port] [bind_addr [: port]]`. El valor por defecto de `user` es el 
usuario actual, y el de `port` el puerto ssh estándar. Las variables opcionales
`bind_to` `bind_addr` `[: port]` especifican la dirección IP y el puerto que otros *workers* 
deberían usar para conectarse a este *worker*.

Si tiene código que desea ejecutar cada vez que Julia se inicia, puede ponerlo en `~/.juliarc.jl`:

```
$ echo 'println("Greetings! 你好! 안녕하세요?")' > ~/.juliarc.jl
$ julia
Greetings! 你好! 안녕하세요?

...
```

Hay varias formas de ejecutar el código Julia y proporcionar opciones, similares a las disponibles para los 
programas `perl` y `ruby:

```
julia [switches] -- [programfile] [args...]
 -v, --version             Display version information
 -h, --help                Print this message

 -J, --sysimage <file>     Start up with the given system image file
 --precompiled={yes|no}    Use precompiled code from system image if available
 --compilecache={yes|no}   Enable/disable incremental precompilation of modules
 -H, --home <dir>          Set location of `julia` executable
 --startup-file={yes|no}   Load ~/.juliarc.jl
 --handle-signals={yes|no} Enable or disable Julia's default signal handlers

 -e, --eval <expr>         Evaluate <expr>
 -E, --print <expr>        Evaluate and show <expr>
 -L, --load <file>         Load <file> immediately on all processors

 -p, --procs {N|auto}      Integer value N launches N additional local worker processes
                           "auto" launches as many workers as the number of local cores
 --machinefile <file>      Run processes on hosts listed in <file>

 -i                        Interactive mode; REPL runs and isinteractive() is true
 -q, --quiet               Quiet startup (no banner)
 --color={yes|no}          Enable or disable color text
 --history-file={yes|no}   Load or save history

 --compile={yes|no|all|min}Enable or disable JIT compiler, or request exhaustive compilation
 -C, --cpu-target <target> Limit usage of cpu features up to <target>
 -O, --optimize={0,1,2,3}  Set the optimization level (default is 2 if unspecified or 3 if specified as -O)
 -g, -g <level>            Enable / Set the level of debug info generation (default is 1 if unspecified or 2 if specified as -g)
 --inline={yes|no}         Control whether inlining is permitted (overrides functions declared as @inline)
 --check-bounds={yes|no}   Emit bounds checks always or never (ignoring declarations)
 --math-mode={ieee,fast}   Disallow or enable unsafe floating point optimizations (overrides @fastmath declaration)

 --depwarn={yes|no|error}  Enable or disable syntax and method deprecation warnings ("error" turns warnings into errors)

 --output-o name           Generate an object file (including system image data)
 --output-ji name          Generate a system image data file (.ji)
 --output-bc name          Generate LLVM bitcode (.bc)
 --output-incremental=no   Generate an incremental output file (rather than complete)

 --code-coverage={none|user|all}, --code-coverage
                           Count executions of source lines (omitting setting is equivalent to "user")
 --track-allocation={none|user|all}, --track-allocation
                           Count bytes allocated by each source line
```

## Resources

Adems de este manual, hay otros recursos que pueden ayudar a los usuarios nuevos cuanto empiezan 
con Julia:

  * [Julia and IJulia cheatsheet](http://math.mit.edu/~stevenj/Julia-cheatsheet.pdf)
  * [Learn Julia in a few minutes](https://learnxinyminutes.com/docs/julia/)
  * [Learn Julia the Hard Way](https://github.com/chrisvoncsefalvay/learn-julia-the-hard-way)
  * [Julia by Example](http://samuelcolvin.github.io/JuliaByExample/)
  * [Hands-on Julia](https://github.com/dpsanders/hands_on_julia)
  * [Tutorial for Homer Reid's numerical analysis class](http://homerreid.dyndns.org/teaching/18.330/JuliaProgramming.shtml)
  * [An introductory presentation](https://raw.githubusercontent.com/ViralBShah/julia-presentations/master/Fifth-Elephant-2013/Fifth-Elephant-2013.pdf)
  * [Videos from the Julia tutorial at MIT](https://julialang.org/blog/2013/03/julia-tutorial-MIT)
  * [YouTube videos from the JuliaCons](https://www.youtube.com/user/JuliaLanguage/playlists)
