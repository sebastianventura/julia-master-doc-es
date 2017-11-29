# [Módulos](@id modules)

Los módulos en Julia son espacio de trabajo de variables separados, es decir, ellos introducen un nuevo ámbito global. Ellos están delimitados sintácticamente dentro de `module Nombre ... end`. Los módulos nos permiten crear definiciones de nivel superior (o  variables globales) sin preocuparnos sobre conflictos de nombres cuando estamos usando nuestro código con cualquier otro. Dentro de un módulo, podemos controlar qué nombres de otros módulos son visibles (vía importación) y especificar cúales de nuestros nombres queremos que sean públicos (vía exportación). 

El siguiente ejemplo muestra las principales características de los módulos. No está destinado para ser ejecutado, sino que se muestra para propoósitos ilustrativos:

```julia
module MyModule
using Lib

using BigLib: thing1, thing2

import Base.show

importall OtherLib

export MyType, foo

struct MyType
    x
end

bar(x) = 2x
foo(a::MyType) = bar(a.x) + 1

show(io::IO, a::MyType) = print(io, "MyType $(a.x)")
end
```

Notese que el estilo es no indentar el cuerpo del módulo, ya que esto llevaría a que el fichero completo estuviera indentado.

Este módulo define un tipo `MyType` y dos funciones. Las funciones `foo` y `MyType` son exportadas y, en consecuencia, estará disponibles para ser importadas en otros módulos. La función `bar` es privada a `MyModule`.

La instrucción `using Lib` significa que un módulo llamado `Lib` estará disponible para resover nombres cuando se necesite. Cuando se encuentra que una variable no tiene definición en el módulo actual, el sistema la buscará entre las variables exportadas por `Lib` y la importará si la encuentra allí. Esto significa que todos los usos de esta global dentro del módulo actual resoverá a la definición de esta variable en `Lib`.

La instrucción `using BigLib: thing1, thing2` es una abreviación sintáctica de `using BigLib.thing1, BigLib.thing2`.

La palabra clave `import` soporta la misma sintaxis que `using`, pero sólo opera sobre un solo nombre cada vez. Ella no añade módulos para ser buscados de la forma que lo hace `using`. También difiere de `using` en que las funciones deben ser importandas mediante `import` para ser extendidas con nuevos métodos.

En el jemplo anterior `MyModule` deseamos añadir un método a la función estándar `show`, por lo que tenemos que escribir `import Base.show`. Las funciones cuyos nombres son sólo visibles via `using` no pueden ser extendidas.

La palabra clave `importall` importa explícitamente todos los nombres exportados por el módulo especificado, con si se hubiera usado `import` individualmente sobre cada uno de ellos.

Una vez que una variable se ha hecho visible vía `using` o `import`, un módulo puede no crear su propia variable con el mismo nombre. Las variables importadas son de solo lectura; asignar a una variable global siempre afecta a una variable propiedad del módulo actual, o si no causar un error.

## Resumen de uso de los módulos

Para cargar un módulo, pueden usarse dos palabras clave principales: `using` e `import`. Para comprender sus diferencias, considérese el siguiente ejemplo:

```julia
module MyModule

export x, y

x() = "x"
y() = "y"
p() = "p"

end
```

En este módulo exportamos las funciones `x` e `y` (con la palabra clave `export`) y también tenemos la función no exportada `p`. Hay varias diferentes formas de cargar el mósulo y sus funciones internas en el espacio de trabajo actual:

| Mandato de importación                  | Qué se introduce en el ámbito                                                      | Disponible para extensión de método              |
|:------------------------------- |:------------------------------------------------------------------------------- |:------------------------------------------- |
| `using MyModule`                | All `export`ed names (`x` and `y`), `MyModule.x`, `MyModule.y` and `MyModule.p` | `MyModule.x`, `MyModule.y` and `MyModule.p` |
| `using MyModule.x, MyModule.p`  | `x` and `p`                                                                     |                                             |
| `using MyModule: x, p`          | `x` and `p`                                                                     |                                             |
| `import MyModule`               | `MyModule.x`, `MyModule.y` and `MyModule.p`                                     | `MyModule.x`, `MyModule.y` and `MyModule.p` |
| `import MyModule.x, MyModule.p` | `x` and `p`                                                                     | `x` and `p`                                 |
| `import MyModule: x, p`         | `x` and `p`                                                                     | `x` and `p`                                 |
| `importall MyModule`            | All `export`ed names (`x` and `y`)                                              | `x` and `y`                                 |

### Módulos y ficheros 

Los ficheros y los nombres de ficheros no están relacionados con los módulos. Los módulos están asociados sólo con las expresiones `module`. Uno puede tener múltiples ficheros por módulo, y múltiples módulos por fichero:

```julia
module Foo

include("file1.jl")
include("file2.jl")

end
```

Incluir el mismo código en módulos diferentes porporciona un comportamiento similar a la mezcla. Uno podría usar esto para ejecutar el mismo código con diferentes definiciones base, por ejemplo, código de test ejecutando una versión "segura" de algunos operadores:

```julia
module Normal
include("mycode.jl")
end

module Testing
include("safe_operators.jl")
include("mycode.jl")
end
```

### Módulos estándar

Hay tres módulos estándar importantes: Main, Core y Base.

`Main` es el módulo de nivel superior, y Julia arranca con Main fijado como módulo actual. Las variables definidas en el prompt van a Main, y `whos()` lista las variables en Main. 

`Core` contiene todos los identificadores considerados predefinidos en el lenguaje, por ejemplo, parte del lenguaje y no librerías. Cada módulo especifica implícitamente `using Core`, ya que uno no puede hacer nada sin sus definiciones.

`Base` es la librería estándar (los contenidos de `base/`). Todos los módulos contiene implícitamente `using Base`, ya que este se necesita en la gran mayoría de los casos.

### Definiciones de nivel superior por defecto y módulos esenciales (*bare*)

Además de a `using Base`, los módulos también contiene automáticamente una definició de la función `eval`, que evalúa expresiones dentro del contexto de este módulo.

Si estas definiciones por defecto no son deseadas, los módulos pueden ser definidos usando la palabra clave `baremodule`(nota: `Core` sigue siendo importando, como antes). En términos de `baremodule` un módulo estándar tiene este aspecto.

```
baremodule Mod

using Base

eval(x) = Core.eval(Mod, x)
eval(m,x) = Core.eval(m, x)

...

end
```

### Caminos absolutos y relativos de módulos

Dada la instrucción `using Foo`, el sistema buscará `Foo` dentro de `Main`. Si el módulo no existe, el sistema interará un `require("Foo")` que típicamente da como resultado cargar código desde un paquete instalado. 

Sin embargo, algunos módulos contienen submódulos, lo que significa que tu algunas veces tienes que acceder a un módulo que no está directamente disponible en `Main`. Hay dos formas de hacer esto. La primera es usar un camino absoluto, por ejemplo `using Base.Sort`. El segundo es usar un camino relativo, que hace más fácil importar submódulos del módulo actual o alguno de sus módulos adjuntos:

```
module Parent

module Utils
...
end

using .Utils

...
end
```

Aquí, el módulo `Parent` contiene un submódulo `Utils` y el código en `Parent` quiere que el contenido de `Utils` esté visible. Este se consigue empezando la instrucción `using` con un punto. Ada punto adicional añadido se muevo hacia arriba niveles adicionales en la jerarquía de módulos. Por ejemplo, `using ..Utils` buscaría en el módulo que contiene a `Parent`, no en el propio `Padre`.

Notese que los cualificadores de importación relativos son sólo válidos para las instrucciones `uing` e `import`.

### Caminos de ficheros de módulo

La variable global [`LOAD_PATH`](@ref) contiene los directorios donde Julia busca módulos cada vez que se invoca `require`. Esto puede extenderse usandog [`push!`](@ref):

```julia
push!(LOAD_PATH, "/Path/To/My/Module/")
```

Poniendo esta instrucción en el ficehro `~/.juliarc.jl` extenderemos la variable [`LOAD_PATH`](@ref) on every Julia startup.
en cada arranque de Julia. Alternativamente, el camino de carga de módulos puede ser extendido definiendo la variable de entorno `JULIA_LOAD_PATH`.

### Miscelánea sobre espacios de nombres

Si un nombre está cualificado (por ejemplo `Base.sin`) entonces puede ser accedido incluso aunque no esté exportado. Esto es bastante útil de cara a depuración.

También puede haber métodos agregados al usar el nombre calificado como el nombre de la función. Sin embargo, debido a las ambigüedades sintácticas que surgen, si uno desea agregar métodos a una función en un módulo diferente cuyo nombre contiene solo símbolos, tal como un operador (por ejemplo, `Base.+`), debe usar la notación `Base.:+` para referirse a él. Si el operador tiene más de un carácter de longitud, debe rodearlo entre paréntesis, por ejemplo: `Base.:(==)`.

Los nombres de macros se escriben con `@` en las instrucciones de importación y exportación. Por ejemplo, `import Mod.@mac`. Las macros en otros módulos pueden ser invocadas como `Mod.@mac` or `@Mod.mac`.

La sintaxis `M.x=y` no funciona para asignar una variable global en otro módulo: la asignación global es siempre local a un módulo.

Una variable puede estar "reservada" por el módulo actual sin asignar a ella declarándola como `global x` en el nivel superior. Esto puede usarse para prevenir conflictos de nombreas para globales inicializadas después del tiempo de carga.

###  Inicialización y precompilación de módulos

Los módulos grandes puede necesitar varios segundos para cargar, debido a que ejecutar todas las instrucciones en un módulo implica compilar una gran candidad de código. Julia proporciona la capacidad de crear versiones precompiladas de los módulos para reducir este timepo.

Para crear un fichero de módulo precompilado incremental, añadimos `__precompile--()` al principio de nuestro fichero de módulo (antes de que empiece la palabra `module`). Esto causará que él sea compilado automáticamente la primera vez que se importe. Alternativamete, podemos llamar a `Base.compilecache(modulename)`.  Los ficheros caché resultantes se almacenarán en `Base.LOAD_CACHE_PATH[1]`. Posteriormente, el módulo es recompilado automáticamente con `import` cada vez que alguna de sus dependencia cambia; las dependencias son módulos en imports, la construcción de Julia, los ficheros que incluye, o dependencias explícitas declaradas con `include_dependency(path)` en el fichero o ficheros de módulo.

Para dependencias de fichero, un cambio se determina examinando si el momento de modificación (mtime) de cada fichero cargado por `include` o añadido explícitamente mediante `include_dependency` está sin cambios o igual al tiempo de modificaci´no truncado al segundo más cercano (para acomodar sistemas que no pueden copiar mtime con exactitud menor que el segundo). También se tiene en cuenta si el camino al fichero elegido por la lógica de búsqueda en `require` se corresponde con el camino que había creado el fichero precompilado.

También se tiene en cuenta el conjunto de dependencias ya cargadas en el proceso actual y no recompilará esos módulos, incluso si sus ficheros cambian o desaparecen, para evitar crear incompatibilidades entre el sistema en ejecución y la caché precompilada. I queremos tener cambios en la fuente reflejados en el sistema en ejecución, deberíamos llamar a `reload("Module")` sobre el módulo que hemos cambiado, y cualquier módulo que dependa de él en el cuál quieres ver reflejado el cambio.

Precompilar un módulo también recursivamente precompila todo los módulos que son importados allí. Si sabemos que no es seguro precompilar nuestor módulo (por las razones descritas debajo) deberíamos poner `__precompile__(false)` en el fichero del módulo para causar que `Base.compilecache` lance un error (y por tanto evite que el módulo sea importado por cualquier otro módulo precompilado.

`__precompile` **NO** debería ser usado en un módulo a menos que todas sus dependencias estén también usando `__precompile__()`. Un fallo en hacer esto puede dar como resultado un error en tiempo de ejecución cuando se carga el módulo.

Para hacer que nuestro módulo funcione con la precompilación, sin embargo, necesitamos cambiar nuestro módulo para separar explícitamente cualquier paso de inicialización que deba ocurrir en tiempo de ejecución de pasos que pueden ocurrir en tiempo de compilación. Para este propósito, Julia te permite definir una función `__init__()` en tu módulo que ejecuta cualquier paso de  inicialización que deba tener lugar en tiempo de ejecución. Esta función no será llaada durante la compilación (`--ouput-*` o `__precompile__()`). Podemos, por supuesto, llamarla manualmente si es necesario, pero el comportamiento por defecto es asumir que esta función trata con el estado de computación para la máquina local, que no necesita ser (o incluso no debe ser) capturada en la imagen compilada. Ella puede ser llamada después de que el módulo sea cargado en un proceso, incluyendo si el está siendo cargado en una compilación incremental (`--output-incremental=yes`), pero no si está siendo cargado en un proceso de compilación completo. 

En particular, si defines un `function __init__()` en un módulo, entonces Julia llamará a `__init__()` inmediatamente después de que el módulo se cargue (es decir, mediante `import`, `using` o `require`) en tiempo de ejecución la *primera* vez (es decir, `__init__` sólo es llamado una vez, y sólo después de que todas las instrucciones en el módulo se hayan ejecutado). Como el es llamado después de que el módulo sea importado por completo, los submódulos u otros módulos importandos tienen sus funciones `__init__`llamadas antes del `__init__`del módulo contenedor.

Dos usos típicos de `__init__` son llamar a funciones de inicialización en tiempo de ejecución de librerías externas en C e inicializar constantes globales que implican punteros devueltos por las librerías externas. Por ejemplo, supongamos que estamos llamado a una librería en C `libfoo` que requiere que llamemos a la función de inicialización `foo_init()` en tiempo de ejecución. Supongamos que también queremos definir una constante global `foo_data_ptr` que almacena el valor de retorno de una función `void *foo_data()` definida por `libfoo` (esta constante tiene que se inicializada en tiempo de ejecución - no de compilación- debido a que el puntero cambiará de una ejecución a otra). Podríamos conseguir esto definiendo la siguiente función `__init__`en nuestro módulo:

```julia
const foo_data_ptr = Ref{Ptr{Void}}(0)
function __init__()
    ccall((:foo_init, :libfoo), Void, ())
    foo_data_ptr[] = ccall((:foo_data, :libfoo), Ptr{Void}, ())
end
```

Notese que es perfectamente posible definir un global dentro de la función como `__init__`; esta es una de las ventajas de usar un lenguaje dinámico. Pero haciéndolo una constante en un ámbito global, podemos asegurar que el tipo es conocido al compilador y le permite generar código mejor optimizado. Obviamente, otros globales de nuestro módulo que dependan de `foo_data_ptr` también tendrían que ser inicializados en `__init__`.

Las constantes que implican a la mayoría de los objetos Julia que no son producidas por `ccall` no necesitan se colocadas en `__init__`: sus definiciones pueden ser precompiladas y cargadas desde la imagen cacheada del módulo. Esto incluye complicados objetos alojados en el montón (*heap*) como los arrays. Sin embargo, cualquier rutina que devuelva un calor de puntero crudo debe ser llamada en tiempo de ejecución para que la precompilación funcione (los objetos Ptr se convertirán en punteros nulos a menos que sean ocultados dentro de un objeto isbits). Esto incluye los valore de retorno de las funciones Julia `cfunction` y  `pointer`.

Los tipos diccionario y conjunto, o en general cualquiera que dependa de la salida de un método `hash(key)`, son un caso más difícil. En el caso común de que las claves sean números, cadenas, símbolos, rangos, `Expr` o composiciones de estos tipos (vía arraysm, tuplas, conjuntos, pares, etc.) son seguros de precompilar. Sin embargo, para otros tipos clave, tales como `Function` o `DataType` y tipos genéricos definidos por el usuario donde no se ha definido un método `hash` el método `hash` de apoyo depende de la dirección en memoria del objeto (vía su `object_id`) y por tanto puede cambiar de ejecución a ejecución. Si tienes uno de esos tipos como clave, o no estás seguro, puedes inicializar este diccionario desde dentro de tu función `__init__` para mayor seguridad. Alternativamente, puedes usar el tipo diccionario `ObjectIdDict`, que es especialmente manejado por precompilación por lo que es seguro de inicializar en tiempo de compilación.

Cuando se usa precompilación, es importante mantener un claro sentido de la distinción entre la fase de compilación y la de ejecución. En este modo, a menudo será mucho más evidente que Julia es un compilador que permite la ejecución de código arbitrario Julia, no un intérprete independiente que también genera código compilado.

Otros escenarios de fallo potencial conocidos incluyen:

1. Contadores globales (por ejemplo, para intentar identificar objetos únicamente). Considere el siguiente código:

   ```julia
   mutable struct UniquedById
       myid::Int
       let counter = 0
           UniquedById() = new(counter += 1)
       end
   end
   ```

   mientras que la intención de este código era dar a cada instancia un id único, el valor del contador es grabado al final de la compilación. Todos los usos posteriores de este módulo compilado incrementalmente empezarán desde el mismo valor de contador.
  
  Note que `object_if` (qeu trabaja haciendo *hash* en el puntero a memoria) tieen problemas similares (ver las notas sobre el uso de `Dict abajo).

   Una alternativa es usar una macro para capturar [`@__MODULE__`](@ref) y almacenarlo sólo con el valor actual de `counter`; sin embargo, puede ser mejor rediseñar el código para no depender de este estado global.
2. Las colecciones asociativas (tales como `Dict` y `Set`) necesitan ser re-hasheadas en `__init__` (en el futuro, 
   puede proporcionarse un mecanismo para registrar un inicializador de función).
3. Dependiendo de los efectos secundarios de tiempo de compilación que persisten a través del tiempo de carga. El 
   ejemplo incluye: modificar arrays u otras variables en otros módulos de Julia; mantener manejadores para abrir 
   archivos o dispositivos; almacenar punteros a otros recursos del sistema (incluyendo memoria);
4. Crear "copias" accidentales de estado global desde otro módulo, haciendo referencia directamente a él en vez de 
   a través de su ruta de búsqueda. Por ejemplo, (en el ámbito global).

   ```julia
   #mystdout = Base.STDOUT #= will not work correctly, since this will copy Base.STDOUT into this module =#
   # instead use accessor functions:
   getstdout() = Base.STDOUT #= best option =#
   # or move the assignment into the runtime:
   __init__() = global mystdout = Base.STDOUT #= also works =#
   ```

Varias restricciones adicionales se colocan sobre las operaciones que se pueden hacer mientras se precompila el código para ayudar al usuario a evitar otras situaciones de comportamiento incorrecto:

1. Llamar [`eval`](@ref) para provocar un efecto secundario en otro módulo. Esto también provocará que se emita una
   advertencia cuando se establece el indicador de precompilación incremental.
2. Las sentencias `global consts` del ámbito local después de `__init __()` se han iniciado (vea el problema #12010 
   para los planes de agregar un error para esto).
3. Reemplazar un módulo (o llamar [`workspace`](@ref)) es un error de tiempo de ejecución al realizar una 
   precompilación incremental.

Algunos otros puntos a tener en cuenta:

1. Ninguna recarga de código / invalidación de caché se realiza después de que se realizan cambios en los propios 
   archivos fuente (incluyendo [`Pkg.update`](@ref)) y no se realiza ninguna limpieza después de [`Pkg.rm`](@ref).
2. El comportamiento de compartir la memoria de una matriz reestructurada es ignorado por la precompilación (cada 
   vista obtiene su propia copia).
3. Esperar que el sistema de archivos no cambie entre tiempo de compilación y tiempo de ejecución, p.ej 
   [`@__ FILE __`](@ref) / `source_path()` para encontrar recursos en tiempo de ejecución, o la macro 
   BinDeps @checked_lib. A veces esto es inevitable. Sin embargo, cuando sea posible, puede ser una buena práctica 
   copiar recursos en el módulo en tiempo de compilación para que no sea necesario encontrarlos en tiempo de ejecución.
4. Los objetos `WeakRef` y los finalizadores no son manejados correctamente por el serializador (esto se arreglará 
   en una próxima versión).
5. Por lo general, es mejor evitar la captura de referencias a instancias de objetos de metadatos internos como 
   `Method`, `MethodInstance`, `MethodTable`, `TypeMapLevel`, `TypeMapEntry` y campos de esos objetos, ya que esto 
   puede confundir al serializador y puede no conducir al resultado deseado. No es necesariamente un error hacer esto, 
   pero sólo tiene que estar preparado para que el sistema intente copiar algunos de ellos y crear una única instancia 
   única de otros.
   
A veces es útil durante el desarrollo del módulo para desactivar la precompilación incremental. El indicador de línea de comandos `--compilecache = {yes | no}` le permite activar y desactivar la precompilación del módulo. Cuando se inicia Julia con `--compilecache = no` se ignoran los módulos serializados en el caché de compilación al cargar módulos y dependencias de módulo. `Base.compilecache()` todavía se puede llamar manualmente y respetará las directivas `__precompile__()` para el módulo. El estado de este indicador de línea de comandos se pasa a [`Pkg.build()`](@ref) para deshabilitar el desencadenamiento automático de precompilación al instalar, actualizar y crear explícitamente paquetes.
updating, and explicitly building packages.
