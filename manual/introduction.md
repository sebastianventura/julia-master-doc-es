# [Introducción](@id man-introduction)

La computación científica ha requerido tradicionalmente el máximo rendimiento, aunque los expertos de los distintos  dominios se hayan movido en gran parte a los idiomas dinámicos más lentos para el trabajo diario. Creemos que hay muchas buenas razones para preferir lenguajes dinámicos para estas aplicaciones, y no esperamos que su uso disminuya. Afortunadamente, el diseño de lenguajes y las técnicas de compilación modernos permiten casi eliminar el compromiso del rendimiento y proporcionar un solo entorno suficientemente productivo para la creación de prototipos y suficientemente eficiente para implementar aplicaciones de alto rendimiento. El lenguaje de programación de Julia cumple este papel: es un lenguaje dinámico y flexible, apropiado para la computación científica y numérica, con un rendimiento comparable al de los lenguajes tradicionales de tipo estático.

Debido a que el compilador de Julia es diferente de los intérpretes utilizados para lenguajes como Python o R, podría parecer al principio que el rendimiento de Julia no es intuitivo. Si encuentra que algo es lento, le recomendamos que lea la sección [Consejos de Rendimiento](@ref man-performance-tips) antes de intentar otra cosa. Una vez que entienda cómo funciona Julia, será fácil escribir código que casi tan rápido como el código C.

Julia features optional typing, multiple dispatch, and good performance, achieved using type inference
and [just-in-time (JIT) compilation](https://en.wikipedia.org/wiki/Just-in-time_compilation),
implemented using [LLVM](https://en.wikipedia.org/wiki/Low_Level_Virtual_Machine). It is multi-paradigm,
combining features of imperative, functional, and object-oriented programming. Julia provides
ease and expressiveness for high-level numerical computing, in the same way as languages such
as R, MATLAB, and Python, but also supports general programming. To achieve this, Julia builds
upon the lineage of mathematical programming languages, but also borrows much from popular dynamic
languages, including [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)), [Perl](https://en.wikipedia.org/wiki/Perl_(programming_language)),
[Python](https://en.wikipedia.org/wiki/Python_(programming_language)), [Lua](https://en.wikipedia.org/wiki/Lua_(programming_language)),
and [Ruby](https://en.wikipedia.org/wiki/Ruby_(programming_language)).

The most significant departures of Julia from typical dynamic languages are:

  * El lenguaje básico impone muy poco; La biblioteca estándar se ha escritop en el propio Julia, incluyendo operaciones primitivas como la aritmética entera.
  * Un lenguaje enriquecido de tipos para construir y describir objetos, que también se puede utilizar opcionalmente para hacer declaraciones de tipo.
  * La capacidad de definir el comportamiento de la función a través de muchas combinaciones de tipos de argumentos mediante el [despacho múltiple](https://en.wikipedia.org/wiki/Multiple_dispatch)
  * Generación automática de código eficiente y especializado para diferentes tipos de argumentos.
  * Buen rendimiento, aproximándose al de los lenguajes compilados estáticamente como C.

Aunque a veces se habla de lenguajes dinámicos como "sin tipo", definitivamente no son: cada objeto, ya sea primitivo o definido por el usuario, tiene un tipo. La falta de declaraciones de tipos en la mayoría de los idiomas dinámicos, sin embargo, significa que uno no puede instruir al compilador acerca de los tipos de valores y, a menudo, no puede hablar explícitamente de tipos en absoluto. En lenguajes estáticos, por otro lado, aunque uno puede -y normalmente debe- anotar tipos para el compilador, los tipos sólo existen en tiempo de compilación y no pueden ser manipulados o expresados en tiempo de ejecución. En Julia, los tipos son objetos en tiempo de ejecución y también se pueden utilizar para transmitir información al compilador.

Aunque el programador casual no necesita usar explícitamente los tipos o el despacho múltiple, son las características centrales unificadoras de Julia: las funciones se definen en diferentes combinaciones de tipos de argumentos y se aplican despachando a la definición concordante más específica. Este modelo se ajusta bien a la programación matemática, donde no es natural que el primer argumento "posea" una operación como en la programación orientada a objetos tradicional. Los operadores son sólo funciones con notación especial - para ampliar la adición a nuevos tipos de datos definidos por el usuario, se definen nuevos métodos para la función +. El código existente se aplica sin problemas a los nuevos tipos de datos.

En parte debido a la inferencia de tipo en tiempo de ejecución (aumentada por anotaciones de tipos opcionales), y en parte debido a enfoque muy basado en el rendimiento desde el inicio del proyecto, la eficiencia computacional de Julia supera la de otros lenguajes dinámicos e incluso rivaliza con la de lenguajes de compilación estática. Para los problemas numéricos a gran escala, la velocidad siempre ha sido, continúa siendo, y probablemente siempre será crucial: la cantidad de datos procesados se ha mantenido fácilmente al ritmo de la Ley de Moore durante las últimas décadas.

Julia tiene como objetivo crear una combinación sin precedentes de facilidad de uso, potencia y eficiencia en un solo idioma. Además de lo anterior, algunas ventajas de Julia sobre sistemas comparables incluyen:

  * Libre y de código abierto ([con licencia MIT](https://github.com/JuliaLang/julia/blob/master/LICENSE.md))
  * Los tipos definidos por el usuario son tan rápidos y compactos como los predefinidos.
  * No hay necesidad de vectorizar código para el rendimiento; el código devectorizado es rápido
  * Diseñado para el paralelismo y la computación distribuida.
  * Hilos "verdes" de peso ligero ([coroutinas](https://en.wikipedia.org/wiki/Coroutine)).
  * Sistema de tipos discreto pero potente.
  * Conversiones y promociones elegantes y extensibles para números y otros tipos.
  * Soporte eficiente para [Unicode](https://en.wikipedia.org/wiki/Unicode), incluyendo pero no 
  limitado a [UTF-8](https://en.wikipedia.org/wiki/UTF-8)
  * Llamada a las funciones C directamente (no se necesitan envolturas o API especiales).
  * Poderosas capacidades tipo shell para administrar otros procesos.
  * Macros similares a Lisp y otras instalaciones de metaprogramación.
