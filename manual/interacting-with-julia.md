# Interactuando con Julia

Julia viene con un REPL (read-eval-print loop) interactivo de línea de comando  integrado en el ejecutable `julia`. Además de permitir una evaluación rápida y fácil de las declaraciones de Julia, tiene un historial de búsqueda, finalización de pestañas, muchas combinaciones útiles de teclas, ayuda dedicada y modos de shell. El REPL se puede iniciar simplemente llamando a `julia` sin argumentos o haciendo doble clic en el ejecutable:

```
$ julia
               _
   _       _ _(_)_     |  A fresh approach to technical computing
  (_)     | (_) (_)    |  Documentation: https://docs.julialang.org
   _ _   _| |_  __ _   |  Type "?help" for help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 0.6.0-dev.2493 (2017-01-31 18:53 UTC)
 _/ |\__'_|_|_|\__'_|  |  Commit c99e12c* (0 days old master)
|__/                   |  x86_64-linux-gnu

julia>
```

Para salir de la sesión interactiva, escriba `^D` - la tecla de control junto con la tecla `d` en una línea en blanco - o escriba `quit()` seguido de la tecla return o enter. El REPL te saluda con un banner y un prompt `julia>`.

## Los distintos modos de prompt

### El modo Juliano

El REPL tiene cuatro modos principales de operación. El primero y más común es el prompt Juliano. Es el modo de operación predeterminado; cada nueva línea inicialmente comienza con `julia>`. Es aquí donde puede ingresar las expresiones de Julia. Pulsando *return* o *enter* después de haber ingresado una expresión completa, se evaluará la entrada y se mostrará el resultado de la última expresión.

```jldoctest
julia> string(1 + 2)
"3"
```
Hay varias funciones útiles únicas para el trabajo interactivo. Además de mostrar el resultado, el REPL también vincula el resultado a la variable `ans`. Un punto y coma final en la línea se puede utilizar como un indicador para suprimir mostrar el resultado.

```jldoctest
julia> string(3 * 4);

julia> ans
"12"
```

En el modo Julia, el REPL es compatible con algo llamado *pegado rápido*. Esto se activa al pegar texto que comienza con `julia>` en REPL. En ese caso, solo las expresiones que comienzan con `julia>` se analizan, otras se eliminan. Esto hace que sea posible pegar un trozo de código que ha sido copiado de una sesión REPL sin tener que eliminar los prompts y las salidas. Esta característica está habilitada de forma predeterminada, pero se puede desactivar o habilitar a voluntad con `Base.REPL.enable_promptpaste(:: Bool)`. Si está habilitado, puedes probar pegando el bloque de código arriba de este párrafo directamente en REPL. Esta función no funciona en el símbolo del sistema estándar de Windows debido a su limitación para detectar cuándo se produce un pegado.

### Modo Ayuda

Cuando el cursor está al principio de la línea, el aviso se puede cambiar a un modo de ayuda escribiendo `?`. Julia intentará imprimir ayuda o documentación para todo lo ingresado en modo de ayuda:

```julia-repl
julia> ? # upon typing ?, the prompt changes (in place) to: help?>

help?> string
search: string String stringmime Cstring Cwstring RevString randstring bytestring SubString

  string(xs...)

  Create a string from any values using the print function.
```

Las macros, los tipos y las variables también se pueden consultar:

```
help?> @time
  @time

  A macro to execute an expression, printing the time it took to execute, the number of allocations,
  and the total number of bytes its execution caused to be allocated, before returning the value of the
  expression.

  See also @timev, @timed, @elapsed, and @allocated.

help?> AbstractString
search: AbstractString AbstractSparseMatrix AbstractSparseVector AbstractSet

  No documentation found.

  Summary:

  abstract AbstractString <: Any

  Subtypes:

  Base.Test.GenericString
  DirectIndexString
  String
```

El modo de ayuda puede salir presionando la tecla de retroceso al comienzo de la línea.

### [Modo Shell](@id man-shell-mode)

Del mismo modo que el modo de ayuda es útil para acceder rápidamente a la documentación, otra tarea común es utilizar el shell del sistema para ejecutar los comandos del sistema. Así como `?` Ingresó al modo de ayuda cuando está al principio de la línea, un punto y coma (`;`) ingresará al modo shell. Y puede salir presionando el retroceso al comienzo de la línea.

```julia-repl
julia> ; # upon typing ;, the prompt changes (in place) to: shell>

shell> echo hello
hello
```

### Modos de búsqueda

En todos los modos anteriores, las líneas ejecutadas se guardan en un archivo de historial, que se puede buscar. Para iniciar una búsqueda incremental a través del historial anterior, escriba `^R` - la tecla de control junto con la tecla `r`. El aviso cambiará a ```(reverse-i-search)` ': ```, y al escribir, la consulta de búsqueda aparecerá en las comillas. El resultado más reciente que coincida con la consulta se actualizará dinámicamente a la derecha de los dos puntos a medida que se escriba más. Para encontrar un resultado anterior usando la misma consulta, simplemente escriba `^R` de nuevo.

Del mismo modo que `^ R` es una búsqueda inversa,` ^ S` es una búsqueda directa, con el indicador ```(i-search)` ': ```. Los dos pueden usarse conjuntamente para avanzar entre los resultados de coincidencia anterior o siguiente, respectivamente.

## Asociaciones de teclas

El REPL de Julia hace un gran uso de las asociaciones de teclas. Varias combinaciones de teclas de control ya se han introducido anteriormente (`^D` para salir,` ^R` y `^S` para buscar), pero hay muchas más. Además de la tecla de control, también hay enlaces de meta-clave. Estos varían según la plataforma, pero la mayoría de los terminales utilizan de forma predeterminada alt- u opción-, mantenida presionada con una tecla para enviar la meta-clave (o puede configurarse para hacerlo).

| Keybinding          | Description                                                                                |
|:------------------- |:------------------------------------------------------------------------------------------ |
| **Program control** |                                                                                            |
| `^D`                | Exit (when buffer is empty)                                                                |
| `^C`                | Interrupt or cancel                                                                        |
| `^L`                | Clear console screen                                                                       |
| Return/Enter, `^J`  | New line, executing if it is complete                                                      |
| meta-Return/Enter   | Insert new line without executing it                                                       |
| `?` or `;`          | Enter help or shell mode (when at start of a line)                                         |
| `^R`, `^S`          | Incremental history search, described above                                                |
| **Cursor movement** |                                                                                            |
| Right arrow, `^F`   | Move right one character                                                                   |
| Left arrow, `^B`    | Move left one character                                                                    |
| Home, `^A`          | Move to beginning of line                                                                  |
| End, `^E`           | Move to end of line                                                                        |
| `^P`                | Change to the previous or next history entry                                               |
| `^N`                | Change to the next history entry                                                           |
| Up arrow            | Move up one line (or to the previous history entry)                                        |
| Down arrow          | Move down one line (or to the next history entry)                                          |
| Page-up             | Change to the previous history entry that matches the text before the cursor               |
| Page-down           | Change to the next history entry that matches the text before the cursor                   |
| `meta-F`            | Move right one word                                                                        |
| `meta-B`            | Move left one word                                                                         |
| `meta-<`            | Change to the first history entry                                                          |
| `meta->`            | Change to the last history entry                                                           |
| **Editing**         |                                                                                            |
| Backspace, `^H`     | Delete the previous character                                                              |
| Delete, `^D`        | Forward delete one character (when buffer has text)                                        |
| meta-Backspace      | Delete the previous word                                                                   |
| `meta-D`            | Forward delete the next word                                                               |
| `^W`                | Delete previous text up to the nearest whitespace                                          |
| `^K`                | "Kill" to end of line, placing the text in a buffer                                        |
| `^Y`                | "Yank" insert the text from the kill buffer                                                |
| `^T`                | Transpose the characters about the cursor                                                  |
| `^Q`                | Write a number in REPL and press `^Q` to open editor at corresponding stackframe or method |


### Personalizando asociaciones de teclas

Las combinaciones de teclas en el REPL de Julia pueden personalizarse completamente según las preferencias de un usuario pasando un diccionario a `REPL.setup_interface()`. Las claves de este diccionario pueden ser caracteres o cadenas. La tecla `'*'` se refiere a la acción predeterminada. Las asociaciones de control y carácter `x` se indican con` "^x" `. Meta plus `x` se puede escribir` "\\ Mx" `. Los valores del mapa de teclas personalizado deben ser ``nothing`` (lo que indica que la entrada debe ignorarse) o las funciones que aceptan la firma `(PromptState, AbstractREPL, Char)`. La función `REPL.setup_interface()` debe invocarse antes de que se inicialice REPL, registrando la operación con `atreplinit()`. Por ejemplo, para enlazar las teclas de flecha hacia arriba y hacia abajo para moverse a través del historial sin búsqueda de prefijos, uno podría poner el siguiente código en `.juliarc.jl`:

```julia
import Base: LineEdit, REPL

const mykeys = Dict{Any,Any}(
    # Up Arrow
    "\e[A" => (s,o...)->(LineEdit.edit_move_up(s) || LineEdit.history_prev(s, LineEdit.mode(s).hist)),
    # Down Arrow
    "\e[B" => (s,o...)->(LineEdit.edit_move_up(s) || LineEdit.history_next(s, LineEdit.mode(s).hist))
)

function customize_keys(repl)
    repl.interface = REPL.setup_interface(repl; extra_repl_keymap = mykeys)
end

atreplinit(customize_keys)
```

Users should refer to `LineEdit.jl` to discover the available actions on key input.

## Uso de Tab para completar expresiones

Tanto en los moddos Juliano como de ayuda del REPL, uno puede entrar los primeros caracteres de una función o tipo y luego pulsar la tecla del tabulador para obtener una lista de posibles coincidencias:

```julia-repl
julia> stri[TAB]
stride     strides     string      stringmime  strip

julia> Stri[TAB]
StridedArray    StridedMatrix    StridedVecOrMat  StridedVector    String
```

La tecla tabulador puede también usarse para sustituir los símbolos matemáticos de LaTeX con sus equivalentes Unicode, y obtener también una lista de coincidencias LaTeX:

```julia-repl
julia> \pi[TAB]
julia> π
π = 3.1415926535897...

julia> e\_1[TAB] = [1,0]
julia> e₁ = [1,0]
2-element Array{Int64,1}:
 1
 0

julia> e\^1[TAB] = [1 0]
julia> e¹ = [1 0]
1×2 Array{Int64,2}:
 1  0

julia> \sqrt[TAB]2     # √ is equivalent to the sqrt() function
julia> √2
1.4142135623730951

julia> \hbar[TAB](h) = h / 2\pi[TAB]
julia> ħ(h) = h / 2π
ħ (generic function with 1 method)

julia> \h[TAB]
\hat              \hermitconjmatrix  \hkswarow          \hrectangle
\hatapprox        \hexagon           \hookleftarrow     \hrectangleblack
\hbar             \hexagonblack      \hookrightarrow    \hslash
\heartsuit        \hksearow          \house             \hspace

julia> α="\alpha[TAB]"   # LaTeX completion also works in strings
julia> α="α"
```

En la sección [Unicode Input](@ref) del manual puede encontrarse una lista completa de finalizaciones usando tabulador.

En modo shell también puede usarse el tabulador para completar cadenas relativas a caminos de fichero:

```julia-repl
julia> path="/[TAB]"
.dockerenv  .juliabox/   boot/        etc/         lib/         media/       opt/         root/        sbin/        sys/         usr/
.dockerinit bin/         dev/         home/        lib64/       mnt/         proc/        run/         srv/         tmp/         var/
shell> /[TAB]
.dockerenv  .juliabox/   boot/        etc/         lib/         media/       opt/         root/        sbin/        sys/         usr/
.dockerinit bin/         dev/         home/        lib64/       mnt/         proc/        run/         srv/         tmp/         var/
```

Esta funcionalidad puede ayudar con la investigación de los métodos disponibles que coinciden con los argumentos de entrada:

```julia-repl
julia> max([TAB] # All methods are displayed, not shown here due to size of the list

julia> max([1, 2], [TAB] # All methods where `Vector{Int}` matches as first argument
max(x, y) in Base at operators.jl:215
max(a, b, c, xs...) in Base at operators.jl:281

julia> max([1, 2], max(1, 2), [TAB] # All methods matching the arguments.
max(x, y) in Base at operators.jl:215
max(a, b, c, xs...) in Base at operators.jl:281
```

Las palabras clave (*keywords*) son también mostradas en los métodos sugeridos, ver la segunda línea después de `;` donde `limit` y `keep` son argumentos palabras clave:

```julia-repl
julia> split("1 1 1", [TAB]
split(str::AbstractString) in Base at strings/util.jl:302
split(str::T, splitter; limit, keep) where T<:AbstractString in Base at strings/util.jl:277
```

La acción de completar los métodos usa inferencia de tipos y puede por tanto ver si los argumentos coinciden incluso si los argumentos son la salida de funcionees La función necesita se estable de tipos para que la acción de completar sea capaz de borrar los métodos no coincidentes.

La acción de completar puede también ayudar a completar campos:

```julia-repl
julia> Pkg.a[TAB]
add       available
```

También pueden completarse los campos para la salida de funciones:

```julia-repl
julia> split("","")[1].[TAB]
endof  offset  string
```

La accin de completar campos para la salida de funciones usa inferencia de tipos, y sólo puede sugerir campos si la función es estable en los tipos.

## Customizing Colors

Los colores utilizados por Julia y REPL se pueden personalizar también. Para cambiar el color del *prompt* de Julia, puede agregar algo como lo siguiente a su `.juliarc.jl`, que se debe colocar dentro del directorio de inicio:

```julia
function customize_colors(repl)
    repl.prompt_color = Base.text_colors[:cyan]
end

atreplinit(customize_colors)
```

Las teclas de color disponibles se pueden ver escribiendo `Base.text_colors` en el modo de ayuda de REPL. Además, los números enteros 0 a 255 se pueden usar como teclas de color para terminales con soporte de 256 colores.

También puede cambiar los colores para la ayuda y las instrucciones del shell e ingresar y contestar el texto configurando el campo apropiado de `repl` en la función `customize_colors` arriba (respectivamente, `help_color`, `shell_color`, `input_color`, y `answer_color`). Para los dos últimos, asegúrese de que el campo `envcolors` también esté configurado a false.

También es posible aplicar el formato en negrita mediante el uso de `Base.text_colors[: bold]` como color. Por ejemplo, para imprimir las respuestas en letra negrita, se puede usar lo siguiente como `.juliarc.jl`:

```julia
function customize_colors(repl)
    repl.envcolors = false
    repl.answer_color = Base.text_colors[:bold]
end

atreplinit(customize_colors)
```

También puede personalizarse el color utilizado para presentar mensajes de advertencia e información estableciendo las variables de entorno apropiadas. Por ejemplo, para generar mensajes de error, de advertencia y de información respectivamente en magenta, amarillo y cian, puede agregar lo siguiente a su archivo `.juliarc.jl`:

```julia
ENV["JULIA_ERROR_COLOR"] = :magenta
ENV["JULIA_WARN_COLOR"] = :yellow
ENV["JULIA_INFO_COLOR"] = :cyan
```
