# Paquetes

Julia tiene un administrador de paquetes incorporado para instalar la funcionalidad añadida y escrita en Julia. También puede instalar bibliotecas externas utilizando el sistema estándar de su sistema operativo para hacerlo, o compilando desde los fuentes. La lista de paquetes Julia registrados se puede encontrar en [http://pkg.julialang.org](http://pkg.julialang.org). Todos los mandatos del gestor de paquetes se encuentran en el módulo `Pkg`, incluido en la instalación `Base` de Julia.

Primero revisaremos los mecanismos de la familia de mandatos de `Pkg` y luego brindaremos algunas pautas sobre cómo registrar sus paquetes. Asegúrese de leer la siguiente sección sobre las convenciones de nombres de paquetes, las versiones de etiquetado y la importancia de un archivo `REQUIRE` para cuando esté listo para agregar su código al repositorio de METADATA seleccionado.

## Estado de un paquete

La función [`Pkg.status()`](@ref) imprime un resumen del estado de los paquetes que uno ha instalado. Inicialmente uno no tendrá paquetes instalados:

```julia-repl
julia> Pkg.status()
INFO: Initializing package repository /Users/stefan/.julia/v0.6
INFO: Cloning METADATA from git://github.com/JuliaLang/METADATA.jl
No packages installed.
```

El directorio de paquetes es inicializado automáticamente la primera vez que una ejecute un mandato `Pkg` que asume su existencia – que incluya [`Pkg.status()`](@ref). He aquí un conjunto de ejemplo no trivial de paquetes requeridos y adicionales:

```julia-repl
julia> Pkg.status()
Required packages:
 - Distributions                 0.2.8
 - SHA                           0.3.2
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.6
```

Estos paquetes están todos en versiones registradas, administradas por `Pkg`. Los paquetes pueden estar en estados más complicados, indicados por anotaciones a la derecha de la versión del paquete instalado; estos estados y anotaciones serán explicados a medida que los encontremos. Para el uso programático, [`Pkg.installed ()`](@ref) devuelve un diccionario, donde se hacen corresponder los nombres de paquetes instalados con la versión instalada de ese paquete:

```julia-repl
julia> Pkg.installed()
Dict{String,VersionNumber} with 4 entries:
"Distributions"     => v"0.2.8"
"Stats"             => v"0.2.6"
"SHA"               => v"0.3.2"
"NumericExtensions" => v"0.2.17"
```

## Añadir y eliminar paquetes

El administrador de paquetes de Julia es un poco inusual ya que es declarativo en lugar de imperativo. Esto significa que uno le dice lo que quiere y el gestor descubre qué versiones instalar (o eliminar) para satisfacer esos requisitos de manera óptima, - y mínimamente. Por tanto, en lugar de instalar un paquete, simplemente lo agrega a la lista de requisitos y luego "resuelve" lo que necesita instalar. En particular, esto significa que si algún paquete se ha instalado porque lo necesitaba una versión anterior de algo que usted quería, y una versión más nueva ya no tiene ese requisito, la actualización realmente eliminará ese paquete.

Los requisitos de paquetes están en el archivo `~ /.julia/v0.6/REQUIRE`. Este archivo puede ser editado a mano y luego llamarse a [`Pkg.resolve()`](@ref) para instalar, actualizar o eliminar paquetes para satisfacer de manera óptima los requisitos, o puede hacer [`Pkg.edit()`](@ref), que abrirá 'REQUIRE' en su editor (configurado a través de las variables de entorno `EDITOR` o` VISUAL`), y luego llamará automáticamente a [`Pkg.resolve()`](@ref) después si es necesario. Si solo desea agregar o eliminar el requisito para un solo paquete, también puede usar los mandatos no interactivos [`Pkg.add()`](@ref) y [`Pkg.rm()`](@ref), que agregan o eliminan un solo requisito a `REQUIRE` y luego llaman a [`Pkg.resolve ()`](@ref).

Puede agregarse un paquete a la lista de requisitos con la función [`Pkg.add()`](@ref), y se instalará el paquete y todos los paquetes de los que depende.

```julia-repl
julia> Pkg.status()
No packages installed.

julia> Pkg.add("Distributions")
INFO: Cloning cache of Distributions from git://github.com/JuliaStats/Distributions.jl.git
INFO: Cloning cache of NumericExtensions from git://github.com/lindahua/NumericExtensions.jl.git
INFO: Cloning cache of Stats from git://github.com/JuliaStats/Stats.jl.git
INFO: Installing Distributions v0.2.7
INFO: Installing NumericExtensions v0.2.17
INFO: Installing Stats v0.2.6
INFO: REQUIRE updated.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.7
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.6
```

Lo que está haciendo es primero agregar `Distribuciones` a su archivo `~/.julia/v0.6/REQUIRE`:

```
$ cat ~/.julia/v0.6/REQUIRE
Distributions
```

A continuación, ejecuta [`Pkg.resolve()`](@ref) utilizando estos nuevos requisitos, lo que lleva a la conclusión de que el paquete `Distributions` debe instalarse ya que es obligatorio pero no está instalado. Como se dijo anteriormente, puede lograr lo mismo editando su archivo `~ /.julia/v0.6/REQUIRE` a mano y luego ejecutando [`Pkg.resolve()`](@ref) usted mismo:

```julia-repl
$ echo SHA >> ~/.julia/v0.6/REQUIRE

julia> Pkg.resolve()
INFO: Cloning cache of SHA from git://github.com/staticfloat/SHA.jl.git
INFO: Installing SHA v0.3.2

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.7
 - SHA                           0.3.2
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.6
```

Esto es funcionalmente equivalente a llamar a [`Pkg.add("SHA")`](@ref), excepto que [`Pkg.add()`](@ ref) no cambia `REQUIRE` hasta *después* de que la instalación haya finalizado, por lo que si hay problemas, `REQUIRE` quedará como estaba antes de llamar a [`Pkg.add()`](@ref). El formato del archivo `REQUIRE` se describe en [Especificación de requisitos](@ref); permite, entre otras cosas, requerir rangos específicos de versiones de paquetes.

Cuando decida que no quiere tener un paquete más, puede usar [`Pkg.rm()`](@ref) para eliminar el requisito del archivo `REQUIRE`:

```julia-repl
julia> Pkg.rm("Distributions")
INFO: Removing Distributions v0.2.7
INFO: Removing Stats v0.2.6
INFO: Removing NumericExtensions v0.2.17
INFO: REQUIRE updated.

julia> Pkg.status()
Required packages:
 - SHA                           0.3.2

julia> Pkg.rm("SHA")
INFO: Removing SHA v0.3.2
INFO: REQUIRE updated.

julia> Pkg.status()
No packages installed.
```
Una vez más, esto es equivalente a editar el archivo `REQUIRE` para eliminar la línea con cada nombre de paquete y ejecutar [`Pkg.resolve()`](@ref) para actualizar el conjunto de paquetes instalados para que coincidan. Mientras que [`Pkg.add()`](@ref) y [`Pkg.rm()`](@ref) son convenientes para agregar y eliminar requisitos para un solo paquete, cuando desea agregar o eliminar paquetes múltiples, puede llamar a [`Pkg.edit()`](@ref) para cambiar manualmente el contenido de `REQUIRE` y luego actualizar sus paquetes en consecuencia. [`Pkg.edit()`](@ref) no retrotrae el contenido de `REQUIRE` si [`Pkg.resolve()`](@ref) falla, sino que debe ejecutar [`Pkg.edit()`](@ ref) otra vez para corregir el contenido de los archivos uno mismo.

Debido a que el administrador de paquetes usa libgit2 internamente para administrar los repositorios git del paquete, los usuarios pueden encontrarse con problemas de protocolo (por ejemplo, si están detrás de un *firewall*) al ejecutar [`Pkg.add()`](@ref). Por defecto, se accederá a todos los paquetes alojados por GitHub a través de 'https'; este valor predeterminado se puede modificar llamando a [`Pkg.setprotocol!()`](@ref). El siguiente comando se puede ejecutar desde la línea de comando para decirle a git que use 'https' en lugar del protocolo 'git' cuando clona todos los repositorios, dondequiera que estén alojados:

```
git config --global url."https://".insteadOf git://
```

Sin embargo, este cambio será en todo el sistema y, por lo tanto, es preferible utilizar [`Pkg.setprotocol!()`](@ref).

!!! note
    
    Las funciones del administrador de paquetes también aceptan el sufijo `.jl` sobre los nombres de
    paquetes, aunque el sufijo sea eliminado internamente. Por ejemplo:

    ```julia
    Pkg.add("Distributions.jl")
    Pkg.rm("Distributions.jl")
    ```

## Instalación de paquetes fuera de línea

Para las máquinas sin conexión a Internet, los paquetes se pueden instalar copiando el directorio raíz del paquete (proporcionado por [`Pkg.dir()`](@ref)) desde una máquina con el mismo sistema operativo y entorno.

[`Pkg.add()`](@ref) hace lo siguiente dentro del directorio raíz del paquete:

1. Agrega el nombre del paquete a `REQUIRE`.
2. Descarga el paquete en `.cache`, luego copia el paquete en el directorio raíz del paquete.
3. Realiza recursivamente el paso 2 contra todos los paquetes enumerados en el archivo `REQUIRE` del paquete.
4. Ejecuta [`Pkg.build()`](@ref)

!!! warning

    Copiar paquetes instalados desde una máquina diferente es frágil para paquetes que requieren dependencias 
    externas binarias. Dichos paquetes pueden romperse debido a diferencias en las versiones del sistema 
    operativo, entornos de compilación y / o dependencias absolutas de rutas.

## Instalar paquetes no registrados

Los paquetes de Julia son simplemente repositorios git, clonables a través de cualquiera de los [protocolos] (https://www.kernel.org/pub/software/scm/git/docs/git-clone.html#URLS) que admite git, y que contienen código Julia que sigue ciertas convenciones de diseño. Los paquetes oficiales de Julia están registrados en el repositorio [METADATA.jl](https://github.com/JuliaLang/METADATA.jl), disponible en una ubicación conocida [^1]. Los mandatos [`Pkg.add()`](@ref) y [`Pkg.rm()`](@ref) de la sección anterior interactúan con los paquetes registrados, pero el administrador de paquetes también puede instalar y trabajar con paquetes no registrados. Para instalar un paquete no registrado, usaremos [`Pkg.clone(url)`](@ref), donde `url` es una URL de git desde la cual se puede clonar el paquete:

```julia-repl
julia> Pkg.clone("git://example.com/path/to/Package.jl.git")
INFO: Cloning Package from git://example.com/path/to/Package.jl.git
Cloning into 'Package'...
remote: Counting objects: 22, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 22 (delta 8), reused 22 (delta 8)
Receiving objects: 100% (22/22), 2.64 KiB, done.
Resolving deltas: 100% (8/8), done.
```

Por convención, los nombres de los repositorios de Julia terminan con `.jl` (el `.git` adicional indica un repositorio "vacío" de git), lo que evita que colisionen con los repositorios de otros lenguajes, y también hace que los paquetes de Julia sean fáciles de encontrar en los motores de búsqueda. Sin embargo, cuando los paquetes están instalados en su directorio `.julia/v0.6`, la extensión es redundante, por lo que la dejamos fuera.

Si los paquetes no registrados contienen un archivo `REQUIRE` en la parte superior de su árbol fuente, ese archivo se usará para determinar de qué paquetes registrados depende el paquete no registrado, y se instalarán automáticamente. Los paquetes no registrados participan en la misma lógica de resolución de versiones que los paquetes registrados, por lo que las versiones de paquetes instalados se ajustarán según sea necesario para satisfacer los requisitos de los paquetes registrados y no registrados.

[^1]:

    El conjunto oficial de paquetes está en  [https://github.com/JuliaLang/METADATA.jl(https://github.com/JuliaLang/METADATA.jl), 
    pero los individuos y las organizaciones pueden usar fácilmente un repositorio de 
    metadatos diferente. Esto permite controlar qué paquetes están disponibles para la i
    nstalación automática. Solo se pueden permitir versiones de paquete auditadas y aprobadas, 
    y hacer paquetes privados u horquillas disponibles. Ver [Repositorio METADATA personalizado](@ref) 
    para más detalles.
    
## Actualizando Paquetes

Cuando los desarrolladores de paquetes publican nuevas versiones registradas de los paquetes que está utilizando, por supuesto, querrá las nuevas versiones brillantes. Para obtener las últimas y mejores versiones de todos sus paquetes, simplemente haga [`Pkg.update ()`] (@ ref):

When package developers publish new registered versions of packages that you're using, you will,
of course, want the new shiny versions. To get the latest and greatest versions of all your packages,
just do [`Pkg.update()`](@ref):

```julia-repl
julia> Pkg.update()
INFO: Updating METADATA...
INFO: Computing changes...
INFO: Upgrading Distributions: v0.2.8 => v0.2.10
INFO: Upgrading Stats: v0.2.7 => v0.2.8
```

El primer paso para actualizar paquetes es generar nuevos cambios en `~/.julia/v0.6/ METADATA` y ver
si se ha publicado alguna nueva versión del paquete registrado. Después de esto, [`Pkg.update()`](@ref)
intenta actualizar paquetes que están desprotegidos en una rama y no están sucios (es decir, no se han realizado cambios a los archivos rastreados por git) al extraer los cambios del repositorio en sentido ascendente del paquete.
Los cambios en sentido ascendente solo se aplicarán si no es necesaria la fusión o rebase, es decir, si la rama
puede ser ["fast-forwarded"](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging).
Si la rama no se puede reenviar rápidamente, se supone que está trabajando en ella y actualizará
el repositorio usted mismo.

Finalmente, el proceso de actualización vuelve a calcular un conjunto óptimo de versiones de paquetes a tener instalado para satisfacer sus requisitos de nivel superior y los requisitos de paquetes "fijos". Un paquete es considerado
corregido si es uno de los siguientes:

1. **No registrado:** el paquete no está en `METADATA` - uno lo instaló con [`Pkg.clone()`](@ref).
2. **Retirado:** el repositorio del paquete está en una rama de desarrollo.
3. **Sucio:** se han realizado cambios a los archivos en el repositorio.

Si cualquiera de estos es el caso, el administrador del paquete no puede cambiar libremente la versión instalada de
el paquete, por lo que sus requisitos deben ser satisfechos por cualquier otra versión del paquete que elija.
La combinación de requisitos de nivel superior en `~/.julia/v0.6/REQUIRE` y el requisito de requisitos fijos
los paquetes se usan para determinar qué se debe instalar.

También puede actualizar solo un subconjunto de los paquetes instalados, proporcionando argumentos a la función [`Pkg.update`](@ref). En ese caso, solo los paquetes proporcionados como argumentos y sus dependencias serán
actualizados:

```julia-repl
julia> Pkg.update("Example")
INFO: Updating METADATA...
INFO: Computing changes...
INFO: Upgrading Example: v0.4.0 => 0.4.1
```

Este proceso de actualización parcial todavía calcula el nuevo conjunto de versiones de paquetes de acuerdo con los requisitos de nivel superior y los paquetes "fijos", pero considera además todos los demás paquetes, excepto los explícitamente proporcionados, y sus dependencias, como corregidas.

## Pago, Pin y Gratis

Puede querer usar la versión `maestra` de un paquete en lugar de una de sus versiones registradas. Es posible que haya correcciones o funcionalidades que necesite y que aún no se hayan publicado en ninguna versión registrada, o puede que sea un desarrollador del paquetes y necesite realizar cambios en `master` o en alguna otra rama de desarrollo. En tales casos, puede hacer [`Pkg.checkout(pkg)`](@ ref) para verificar la rama `master` de` pkg` o [`Pkg.checkout(pkg, branch)`](@ref) para verificar alguna otra rama:

```julia-repl
julia> Pkg.add("Distributions")
INFO: Installing Distributions v0.2.9
INFO: Installing NumericExtensions v0.2.17
INFO: Installing Stats v0.2.7
INFO: REQUIRE updated.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7

julia> Pkg.checkout("Distributions")
INFO: Checking out Distributions master...
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9+             master
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7
```

Inmediatamente después de instalar `Distributions` con [`Pkg.add()`](@ref) está en la versión registrada más reciente actual - `0.2.9` en el momento de escribir esto. Luego, después de ejecutar [`Pkg.checkout("Distributions")`](@ref), puede ver en la salida de [`Pkg.status()`](@ref) que `Distributions` está en una versión no registrada mayor que `0.2.9`, indicado por el número de "pseudo-versión" `0.2.9+`.

Cuando compra una versión no registrada de un paquete, la copia del archivo `REQUIRE` en el repositorio del paquete tiene prioridad sobre cualquier requisito registrado en `METADATA`, por lo que es importante que los desarrolladores mantengan este archivo exacto y actualizado, lo que refleja los requisitos reales de la versión actual del paquete. Si el archivo `REQUIRE` en el repositorio del paquete es incorrecto o falta, las dependencias se pueden eliminar cuando se desprotege el paquete. Este archivo también se usa para completar las versiones del paquete publicadas recientemente si utiliza la API que `Pkg` proporciona para esto (que se describe a continuación).

Cuando decide que ya no desea tener un paquete desprotegido en una rama, puede "liberarlo" de nuevo al control del administrador de paquetes con [`Pkg.free(pkg)`](@ref):

```julia-repl
julia> Pkg.free("Distributions")
INFO: Freeing Distributions...
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7
```

Después de esto, dado que el paquete está en una versión registrada y no en una sucursal, su versión se actualizará a medida que se publiquen nuevas versiones registradas del paquete.

Si desea fijar un paquete en una versión específica para que llamar [`Pkg.update()`](@ref) no cambie la versión en la que está el paquete, puede usar [`Pkg.pin()`](@ref) función:


```julia-repl
julia> Pkg.pin("Stats")
INFO: Creating Stats branch pinned.47c198b1.tmp

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7              pinned.47c198b1.tmp
```

Después de esto, el paquete `Stats` permanecerá anclado en la versión` 0.2.7` - o más específicamente, en el `commit  47c198b1`, pero como las versiones están asociadas permanentemente a un hash git dado, esto es lo mismo. [`Pkg.pin()`](@ref) funciona creando una rama descartable para la confirmación a la que desea fijar el paquete y luego verificando esa ramificación. De forma predeterminada, fija un paquete en la confirmación actual, pero puede elegir una versión diferente pasando un segundo argumento:

```julia-repl
julia> Pkg.pin("Stats",v"0.2.5")
INFO: Creating Stats branch pinned.1fd0983b.tmp
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.5              pinned.1fd0983b.tmp
```

Ahora el paquete `Stats` está anclado en `commit 1fd0983b`, que corresponde a la versión `0.2.5`. Cuando decides "desanclar" un paquete y dejar que el administrador de paquetes lo actualice nuevamente, puedes usar [`Pkg.free()`](@ref) como lo harías para deshacerte de cualquier rama:

```julia-repl
julia> Pkg.free("Stats")
INFO: Freeing Stats...
INFO: No packages to install, update or remove.

julia> Pkg.status()
Required packages:
 - Distributions                 0.2.9
Additional packages:
 - NumericExtensions             0.2.17
 - Stats                         0.2.7
```

Después de esto, el paquete `Stats` es gestionado nuevamente por el administrador del paquete, y las futuras llamadas a [`Pkg.update()`](@ref) lo actualizarán a versiones más nuevas cuando se publiquen. La rama descartable `pinned.1fd0983b.tmp` permanece en el repositorio local de` Stats`, pero como las ramas de git son extremadamente livianas, esto realmente no importa; si desea limpiarlos, puede acceder al repositorio y eliminar esas ramas [^2].

[^2]:
    Los paquetes que no están en las ramas también se marcarán como sucios si realiza cambios en el repositorio, 
    pero eso es menos común.

## Repositorio METADATA Personalizado

Por defecto, Julia supone que utilizará el [repositorio oficial METADATA.jl](https://github.com/JuliaLang/METADATA.jl) para descargar e instalar paquetes. También podemos proporcionar una ubicación de repositorio de metadatos diferente. Un enfoque común es mantener nuestra rama `metadata-v2` actualizada con la rama oficial de Julia y agregar otra rama con sus paquetes personalizados. Puede inicializar su repositorio de metadatos local utilizando esa ubicación y rama personalizadas y luego volver a establecer la base de nuestra rama personalizada con la rama oficial `metadata-v2`. Para utilizar un repositorio y una sucursal personalizados, utilice el siguiente mandato:

```julia-repl
julia> Pkg.init("https://me.example.com/METADATA.jl.git", "branch")
```

El argumento de la rama es opcional y se predetermina a `metadata-v2`. Una vez inicializado, un archivo llamado `META_BRANCH` en su ruta `~/.julia/vX.Y/` hará un seguimiento de la rama con la que se inicializó su repositorio METADATA. Si desea cambiar de rama, necesitará modificar el archivo `META_BRANCH` directamente (¡tenga cuidado!) O elimine el directorio` vX.Y` y reinicie su repositorio METADATA utilizando el mandato `Pkg.init`.

# Desarrollo de Paquetes

El administrador de paquetes de Julia está diseñado para que cuando tenga un paquete instalado, ya pueda ver su código fuente y el historial completo de desarrollo. También puede realizar cambios en los paquetes, enviarlos por git y contribuir fácilmente a las correcciones y mejoras en el flujo ascendente. Del mismo modo, el sistema está diseñado para que, si desea crear un nuevo paquete, la forma más sencilla de hacerlo sea dentro de la infraestructura proporcionada por el administrador de paquetes.

## [Initial Setup](@id man-initial-setup)

Dado que los paquetes son repositorios de git, antes de realizar cualquier desarrollo de paquete, tenemos que fijar los siguientes ajustes de configuración de git global estándar:

```
$ git config --global user.name "FULL NAME"
$ git config --global user.email "EMAIL"
```

donde `FULL NAME` es su nombre completo actual (se permiten espacios entre las comillas dobles) y` EMAIL` es su dirección de correo electrónico real. Aunque no es necesario usar [GitHub](https://github.com/) para crear o publicar paquetes de Julia, la mayoría de los paquetes de Julia al momento de escribir esto están alojados en GitHub y el administrador de paquetes sabe cómo formatear correctamente las URL de origen. y de lo contrario trabajar con el servicio sin problemas. Le recomendamos que cree una [cuenta gratuita](https://github.com/join) en GitHub y luego haga lo siguiente:

```
$ git config --global github.user "USERNAME"
```

donde `USERNAME` es nuestro nombre real de usuario en GitHub. Una vez que hace esto, el administrador de paquetes reconoce el nombre de usuario GitHub y puede configurar las cosas en consecuencia. También debemos [cargar](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fssh) nuestra clave pública SSH a GitHub y configurar un [agente SSH](https: //linux.die.net/man/1/ssh-agent) en nuestra máquina de desarrollo para que pueda realizar cambios con una molestia mínima. En el futuro, haremos que este sistema sea extensible y admitiremos otras opciones comunes de alojamiento git como [BitBucket] (https://bitbucket.org) y permitiremos a los desarrolladores elegir su favorito. Como las funciones de desarrollo del paquete se han movido al paquete [PkgDev](https://github.com/JuliaLang/PkgDev.jl), debe ejecutar `Pkg.add ("PkgDev"); import PkgDev` para acceder a las funciones que comienzan con `PkgDev` en el documento siguiente.

## Hacer cambios a un paquete existente

### Cambios en la Documentación

Si desea mejorar la documentación en línea de un paquete, el enfoque más fácil (al menos para pequeños cambios) es utilizar la funcionalidad de edición en línea de GitHub. Primero, vaya a la "página de inicio" de GitHub del repositorio, busque el archivo (por ejemplo, `README.md`) dentro de la estructura de carpetas del repositorio y haga clic en él. Verá el contenido que se muestra, junto con un pequeño icono de "lápiz" en la esquina superior derecha. Al hacer clic en ese icono, se abre el archivo en modo de edición. Realice los cambios, escriba un breve resumen que describa los cambios que desea realizar (este es su *mensaje de confirmación*), y luego presione "Proponer cambio de archivo". Sus cambios serán enviados para su consideración por el propietario (s) del paquete y sus colaboradores.

Para cambios de documentación más grandes, y especialmente aquellos que espera tener que actualizar en respuesta a los comentarios, puede que le resulte más fácil utilizar el procedimiento para los cambios de código que se describe a continuación.

### Caambios en el Código

#### Resumen Ejecutivo

Aquí suponemos que ya ha configurado git en su máquina local y tiene una cuenta de GitHub (consulte más arriba). Imaginemos que está solucionando un error en el paquete `Images`:

```
Pkg.checkout("Images")           # check out the master branch
<here, make sure your bug is still a bug and hasn't been fixed already>
cd(Pkg.dir("Images"))
;git checkout -b myfixes         # create a branch for your changes
<edit code>                      # be sure to add a test for your bug
Pkg.test("Images")               # make sure everything works now
;git commit -a -m "Fix foo by calling bar"   # write a descriptive message
using PkgDev
PkgDev.submit("Images")
```

La última línea le presentará un enlace para enviar una solicitud de extracción para incorporar sus cambios.

#### Descripción Detallada

Si desea corregir un error o agregar una nueva funcionalidad, desea poder probar los cambios antes de enviarlos para su consideración. También debe tener una manera fácil de actualizar su propuesta en respuesta a los comentarios del propietario del paquete. En consecuencia, en este caso, la estrategia es trabajar localmente en su propia máquina; una vez que esté satisfecho con sus cambios, los envía para su consideración. Este proceso se llama *solicitud de extracción* porque usted está solicitando "extraer" sus cambios en el repositorio principal del proyecto. Debido a que el repositorio en línea no puede ver el código en su máquina privada, primero * envía * sus cambios a una ubicación visible públicamente, su propio * fork * en línea del paquete (alojado en su propia cuenta personal de GitHub).

Supongamos que ya tiene instalado el paquete `Foo`. En la siguiente descripción, todo lo que comience con `Pkg` o` PkgDev` debe escribirse en el prompt de Julia; cualquier cosa que comience con `git` debe escribirse en [modo de shell de julia](@ref man-shell-mode) (o usando el shell que viene con su sistema operativo). Dentro de Julia, puedes combinar estos dos modos:


```julia-repl
julia> cd(Pkg.dir("Foo"))          # go to Foo's folder

shell> git command arguments...    # command will apply to Foo
```

Ahora supongamos que está listo para hacer algunos cambios en `Foo`. Si bien hay varios enfoques posibles, aquí hay uno que se utiliza ampliamente:

  * Desde el prompt de Julia, escriba [`Pkg.checkout (" Foo ")`] (@ ref). Esto garantiza que está ejecutando 
    el último código (la rama `master`), en lugar de cualquier copia de la "versión oficial" que haya instalado. 
    (Si planea corregir un error, en este punto es una buena idea verificar nuevamente si el error ya ha sido 
    corregido por otra persona. Si lo ha hecho, puede solicitar que se etiquete un nuevo lanzamiento oficial 
    para que la corrección se distribuya al resto de la comunidad). Si recibe un error `Foo is dirty, bailing`, 
    consulte [Paquetes sucios](@ref) a continuación.
  * Crea una rama para tus cambios: navega a la carpeta del paquete (la que Julia informa desde 
    [`Pkg.dir("Foo")`](@ref)) y (en modo shell) crea una nueva rama usando `git checkout -b <newbranch>`, 
    donde `<newbranch>` podría ser un nombre descriptivo (por ejemplo,`fixbar`). Al crear una rama, se 
    asegura de que pueda moverse fácilmente entre su nuevo trabajo y la rama actual 'principal' (consulte 
    [https://git-scm.com/book/es/v2/Git-Branching-Branches-in-a-Nutshell](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)).

Si olvida hacer este paso hasta que haya realizado algunos cambios, no se preocupe: consulte [más detalles sobre la bifurcación] (@ ref man-branch-post-hoc) a continuación.

   * Haz tus cambios. Ya sea para corregir un error o agregar una nueva funcionalidad, en la mayoría de los casos tu 
     cambio debería incluir actualizaciones para las carpetas `src/` y `test/`. Si estás solucionando un error, agrega 
     un ejemplo mínimo que demuestre el error (en el código actual) al conjunto de pruebas; al contribuir con una 
     prueba para el error, te aseguras de que el error no vuelva a aparecer accidentalmente en algún momento posterior 
     debido a otros cambios. Si agregas una nueva funcionalidad, la creación de pruebas le demuestra al propietario 
     del paquete que te has asegurado de que el código funcione según lo previsto.
   * Ejecuta las pruebas del paquete y asegúrate de que pasen. Hay varias formas de ejecutar las pruebas:

      * Desde Julia, ejecuta [`Pkg.test (" Foo ")`] (@ ref): esto ejecutará tus pruebas en un proceso 
        separado (nuevo) `julia`.
      * Desde Julia, `include("runtests.jl")` desde la carpeta `test/` del paquete (es posible que el 
        archivo tenga un nombre diferente, busque uno que ejecute todas las pruebas): esto te permite 
        ejecutar las pruebas repetidamente en la misma sesión sin volver a cargar todo el código del 
        paquete; para paquetes que tardan un poco en cargarse, esto puede ser mucho más rápido. Con 
        este enfoque, debes hacer un trabajo adicional para realizar 
        [cambios en el código del paquete](@ref man-workflow-tips).
      * Desde el shell, ejecute `julia ../test / runtests.jl` desde dentro de la carpeta` src / `del 
        paquete.

   * Confirma tus cambios: consulte [https://git-scm.com/book/es/v2/Git-Basics-Recording-Changes-to-the-Repository](https://git-scm.com/book/es/v2/Git-Basics-Recording-Changes-to-the-Repository).
   * Envía tus cambios: desde el prompt de Julia, escribe `PkgDev.submit("Foo")`. Esto impulsará los 
     cambios a la bifurcacion de GitHub y la creará si aún no existe. (Si encuentras un error, 
     [asegúrate de haber configurado tus claves SSH](@ref man-initial-setup).) Julia le dará un 
     hipervínculo; abra ese enlace, edite el mensaje y luego haga clic en "enviar". En ese momento, 
     se notificará al propietario del paquete de sus cambios y podrá iniciar el debate. 
     (Si te sientes cómodo con git, también puedes hacer estos pasos manualmente desde el shell).
   * El propietario del paquete puede sugerir mejoras adicionales. Para responder a esas sugerencias, 
     puede actualizar fácilmente la solicitud de extracción (esto solo funciona para cambios que aún 
     no se han fusionado, para solicitudes de extracción fusionadas, realice nuevos cambios iniciando 
     una nueva rama):
     
      * Si ha cambiado ramas mientras tanto, asegúrese de volver a la misma rama con `git checkout fixbar` 
      (del modo shell) o [`Pkg.checkout("Foo", "fixbar")`](@ref) (del prompt de Julia).
      * Como arriba, haga sus cambios, ejecute las pruebas y comprometa sus cambios.
      * Desde el shell, escribe `git push`. Esto agregará sus nuevas confirmaciones a la misma solicitud 
        de extracción; debería verlos aparecer automáticamente en la página que contiene la discusión de 
        su solicitud de extracción.

Un posible tipo de cambio que el propietario puede solicitar es que elimine sus compromisos. Ver [Squashing](@ref man-squashing-and-rebasing) a continuación.
    
### Paquetes Sucios

Si no se pueden cambiar las ramas porque el administrador del paquete se queja de que su paquete está sucio, significa que tiene algunos cambios que no se han confirmado (enviado). Desde el shell, use `git diff` para ver cuáles son estos cambios; puede descartarlos (`git checkout changedfile.jl`) o confirmarlos antes de cambiar de rama. Si no puede resolver los problemas manualmente, como último recurso, puede eliminar toda la carpeta `"Foo"` y reinstalar una nueva copia con [`Pkg.add("Foo")`](@ref). Naturalmente, esto borra cualquier cambio que haya realizado.

### [Haciendo una Rama *post hoc*](@id man-branch-post-hoc)

Especialmente para los recién llegados a git, uno a menudo se olvida de crear una nueva rama hasta después de que ya se hayan hecho algunos cambios. Si todavía no has organizado ni comprometido tus cambios, puedes crear una nueva rama con `git checkout -b <newbranch>` como siempre - git te mostrará amablemente que algunos archivos han sido modificados y creará la nueva rama para ti. *Sus cambios aún no se han comprometido a esta nueva rama*, por lo que las reglas de trabajo normales aún se aplican.

Sin embargo, si ya has hecho un commit a `master` pero deseas volver al` master` oficial (llamado `origin/master`), utiliza el siguiente procedimiento:

  * Crea una nueva rama. Esta rama mantendrá tus cambios.
  * Asegúrate de que todo esté comprometido con esta rama.
  * `git checkout master`. Si esto no funciona, *no* continúes hasta que hayas resuelto los problemas, o puedes perder los cambios.
  * *Restablece* `master` (tu rama actual) a un estado anterior con `git reset --hard origin/master`
    (ver [https://git-scm.com/blog/2011/07/11/reset.html](https://git-scm.com/blog/2011/07/11/reset.html)).

Esto requiere un poco más de familiaridad con git, por lo que es mucho mejor tener el hábito de crear una rama desde el principio.

### [Squashing and rebasing](@id man-squashing-and-rebasing)

Dependiendo de los gustos del propietario(s) del paquete, él podrá pedirte que "squash" tus compromisos. Esto es especialmente probable si el cambio es bastante simple, pero su historial de compromiso se ve así:

```
WIP: add new 1-line whizbang function (currently breaks package)
Finish whizbang function
Fix typo in variable name
Oops, don't forget to supply default argument
Split into two 1-line functions
Rats, forgot to export the second function
...
```

Esto entra en el territorio del uso de git más avanzado, y se te anima a leer un poco ([https://git-scm.com/book/en/v2/Git-Branching-Rebasing]]https://git -scm.com/book/en/v2/Git-Branching-Rebasing)). Sin embargo, un breve resumen del procedimiento es el siguiente:

   * Para protegerse del error, comienza desde tu rama `fixbar` y crea una nueva rama con` git checkout -b fixbar_backup`. Como has comenzado desde `fixbar`, esta será una copia. Ahora vuelve a la que intentas modificar con `git checkout fixbar`.
   * Desde el shell, escribe `git rebase -i origin/master`.
   * Para combinar confirmaciones, cambia `pick` por `squash` (para opciones adicionales, consulta otras fuentes). Guarda el archivo y cierre la ventana del editor.
   * Edita el mensaje de confirmación combinado.

Si la operación de rebase funciona mal, puedes volver al principio para intentarlo de nuevo de esta manera:

```
git checkout fixbar
git reset --hard fixbar_backup
```

Ahora supongamos que has realizado la operación de rebase con éxito. Como el repositorio `fixbar` ahora se ha separado del que está en tu fork GitHub, vas a tener que hacer un *force push*:

   * Para que sea fácil referirse a tu rama GitHub, cree un "manejador" para ella con `git remote add myfork https://github.com/myaccount/Foo.jl.git`, donde la URL proviene de la "URL de clonación" en la página de tu bifurcación de GitHub.
   * Para que sea fácil referirse a tu rama GitHub, cree un "manejador" para ella con `git remote add myfork https://github.com/myaccount/Foo.jl.git`,
   * Fuerza el push a tu bifurcación con `git push myfork+fixbar`. El `+` indica que esto debería reemplazar la rama `fixbar` encontrada en` myfork`.

## Creando un nuevo Paquete

### REQUIRE habla por sí mismo

Deberías tener un archivo `REQUIRE` en tu repositorio de paquetes, con una directiva mínima de la versión de Julia que esperas que los usuarios ejecuten para que el paquete funcione. Poniendo un piso en qué versión de Julia apoya tu paquete se hace simplemente agregando `julia 0.x` en este archivo. Si bien esta línea es parcialmente informativa, también tiene la consecuencia de si `Pkg.update ()` actualizará el código que se encuentra en los directorios de versiones `.julia`. No actualizará el código encontrado en los directorios de versiones debajo del piso de lo que se especifica en su 'REQUIRE'.

A medida que la versión de desarrollo '0.y' madura, es posible que la utilices con más frecuencia y desees que su paquete la admita. Ten cuidado, la rama de desarrollo de Julia es "tierra de la rotura", y puede esperar que haya cosas que se rompan. Cuando vayas a arreglar lo que rompió tu paquete en la rama de desarrollo `0.y`, probablemente encontrarás que acaba de romper tu paquete en la versión estable.

Hay un mecanismo que se encuentra en el paquete [Compat](https://github.com/JuliaLang/Compat.jl) que le permitirá admitir tanto la versión estable como los cambios de última hora que se encuentran en la versión de desarrollo. Si decide utilizar esta solución, deberá agregar `Compat` a su archivo` REQUIRE`. En este caso, todavía tendrá `julia 0.x` en su` REQUIRE`. La `x` es la versión de piso de lo que su paquete admite.

Es posible que tampoco tengas interés en apoyar la versión de desarrollo de Julia. Del mismo modo que puede agregar un piso a la versión que espera que tengan los usuarios, puede establecer un límite superior. En este caso, pondría `julia 0.x 0.y-` en su archivo `REQUIRE`. El `-` al final del número de versión se refiere a las versiones preliminares de esa versión específica desde el primer compromiso. Al establecerlo como techo, quiere decir que el código es compatible con todo pero no incluye la versión de techo.

Otra situación es que está escribiendo la mayor parte del código para su paquete con Julia `0.y` y no desea admitir la versión estable actual de Julia. Si elige hacer esto, simplemente agregue `julia 0.y-` a su `REQUIRE`. Solo recuerda cambiar `julia 0.y-` a `julia 0.y` en tu archivo` REQUIRE` una vez que `0.y` sea lanzado oficialmente. Si no edita el dash cruft, estás sugiriendo que admita tanto el desarrollo como las versiones estables del mismo número de versión. Eso sería una locura. Consulte la [Especificación de requisitos](@ref) para obtener el formato completo de `REQUIRE`.

Por último, en muchos casos puede necesitar paquetes adicionales para las pruebas. Paquetes adicionales que solo son necesarios para las pruebas deben especificarse en el archivo `test/REQUIRE`. Este archivo `REQUIRE` tiene la misma especificación que el archivo `REQUIRE` estándar.

### Líneas Guía para Nombrar un Paquete

Los nombres de los paquetes deben ser sensatos para la mayoría de los usuarios de Julia, *incluso para aquellos que no son expertos en el dominio*. Cuando envíes tu paquete a METADATA, puedes esperar un poco de ida y vuelta sobre el nombre del paquete con tus colaboradores, especialmente si es ambiguo o puede confundirse con algo diferente de lo que es. Durante este intervalo de tiempo, no es raro obtener una variedad de *diferentes* sugerencias de nombres. Sin embargo, estas son solo sugerencias, con la intención de mantener un espacio de nombres ordenado en el repositorio de METADATA seleccionado. Como este repositorio pertenece a toda la comunidad, es probable que haya algunos colaboradores a los que les importe el nombre de su paquete. Aquí hay algunas pautas a seguir para nombrar su paquete:

1. Evita la jerga. En particular, evita los acrónimos a menos que haya una mínima posibilidad de confusión.

     * Está bien decir `USA` si estás hablando de USA.
     * No está bien decir `PMA`, incluso si estás hablando de una actitud mental positiva.
2. Evite usar `Julia` en el nombre de su paquete.

     * Por lo general, es claro, por contexto y para sus usuarios, que el paquete es un paquete Julia.
     * Tener a Julia en el nombre puede implicar que el paquete está conectado a, o avalado por, colaboradores del propio lenguaje Julia.
3. Los paquetes que proporcionan la mayor parte de su funcionalidad en asociación con un nuevo tipo deberían tener nombres pluralizados.

     * `DataFrames` proporciona el tipo` DataFrame`.
     * `BloomFilters` proporciona el tipo` BloomFilter`.
     * Por el contrario, `JuliaParser` no proporciona ningún tipo nuevo, sino una nueva funcionalidad en la función `JuliaParser.parse()`.
4. Err del lado de la claridad, incluso si la claridad te parece larga.

     * `RandomMatrices` es un nombre menos ambiguo que`RndMat` o `RMT`, aunque estos últimos sean más cortos.
5. Un nombre menos sistemático puede adaptarse a un paquete que implemente uno de varios enfoques posibles para su dominio.

     * Julia no tiene un solo paquete completo de graficación. En cambio, `Gadfly`,` PyPlot`, `Winston`
       y otros paquetes implementan cada uno un enfoque único basado en una filosofía de diseño particular.
     * Por el contrario, `SortingAlgorithms` proporciona una interfaz coherente para usar muchos sistemas bien establecidos
       de algoritmos de clasificación.
6. Los paquetes que envuelven bibliotecas externas o programas deben tener el nombre de esas bibliotecas o programas.

     * `CPLEX.jl` envuelve la biblioteca` CPLEX`, -que se puede identificar fácilmente en una búsqueda web.
     * `MATLAB.jl` proporciona una interfaz para llamar al motor de MATLAB desde dentro de Julia.
     
### Generando el paquete

Supongamos que quiere crear un nuevo paquete de Julia llamado `FooBar`. Para comenzar, haga `PkgDev.generate(pkg,license)` donde `pkg` es el nuevo nombre del paquete y` license` es el nombre de una licencia que el generador de paquetes conoce:

```julia-repl
julia> PkgDev.generate("FooBar","MIT")
INFO: Initializing FooBar repo: /Users/stefan/.julia/v0.6/FooBar
INFO: Origin: git://github.com/StefanKarpinski/FooBar.jl.git
INFO: Generating LICENSE.md
INFO: Generating README.md
INFO: Generating src/FooBar.jl
INFO: Generating test/runtests.jl
INFO: Generating REQUIRE
INFO: Generating .travis.yml
INFO: Generating appveyor.yml
INFO: Generating .gitignore
INFO: Committing FooBar generated files
```

Esto crea el directorio `~/.julia/v0.6/FooBar`, lo inicializa como un repositorio de git, genera un grupo de archivos que todos los paquetes deben tener y los envía al repositorio:

```
$ cd ~/.julia/v0.6/FooBar && git show --stat

commit 84b8e266dae6de30ab9703150b3bf771ec7b6285
Author: Stefan Karpinski <stefan@karpinski.org>
Date:   Wed Oct 16 17:57:58 2013 -0400

    FooBar.jl generated files.

        license: MIT
        authors: Stefan Karpinski
        years:   2013
        user:    StefanKarpinski

    Julia Version 0.3.0-prerelease+3217 [5fcfb13*]

 .gitignore       |  2 ++
 .travis.yml      | 13 +++++++++++++
 LICENSE.md       | 22 +++++++++++++++++++++++
 README.md        |  3 +++
 REQUIRE          |  1 +
 appveyor.yml     | 34 ++++++++++++++++++++++++++++++++++
 src/FooBar.jl    |  5 +++++
 test/runtests.jl |  5 +++++
 8 files changed, 85 insertions(+)
```

Por el momento, el administrador del paquete conoce la licencia "Expat" del MIT, indicada por `"MIT"`, la licencia simplificada BSD, indicada por `"BSD"`, y la versión 2.0 de la licencia del software Apache, indicada por `"ASL"`. Si desea utilizar una licencia diferente, puede solicitarnos que la agreguemos al generador de paquetes, o simplemente seleccione una de estas tres y luego modifique el archivo `~/.julia/v0.6/PACKAGE/LICENSE.md` después de haberlo generado.

Si creó una cuenta de GitHub y configuró git para saber sobre ello, `PkgDev.generate()` establecerá una URL de origen adecuada para usted. También generará automáticamente un archivo `.travis.yml` para usar el servicio de prueba automatizado [Travis] (https://travis-ci.org), y un archivo `appveyor.yml` para usar [AppVeyor](https://www.appveyor.com). Deberá habilitar las pruebas en los sitios web de Travis y AppVeyor para su repositorio de paquetes, pero una vez que lo haya hecho, ya tendrá pruebas de funcionamiento. Por supuesto, todas las pruebas predeterminadas hacen es verificar que `usando FooBar` en Julia funciona.

### Cargando ficheros estáticos No-Julia

Si su código de paquete necesita cargar archivos estáticos que no son código Julia, p. una biblioteca externa o archivos de datos, y se encuentran dentro del directorio del paquete, use la macro `@__DIR__` para determinar el directorio del archivo fuente actual. Por ejemplo, si `FooBar/src/FooBar.jl` necesita cargar  `FooBar/data/foo.csv`, use el siguiente código:

```julia
datapath = joinpath(@__DIR__, "..", "data")
foo = readcsv(joinpath(datapath, "foo.csv"))
```

### Making Your Package Available

Once you've made some commits and you're happy with how `FooBar` is working, you may want to get
some other people to try it out. First you'll need to create the remote repository and push your
code to it; we don't yet automatically do this for you, but we will in the future and it's not
too hard to figure out [^3]. Once you've done this, letting people try out your code is as simple
as sending them the URL of the published repo – in this case:

```
git://github.com/StefanKarpinski/FooBar.jl.git
```

For your package, it will be your GitHub user name and the name of your package, but you get the
idea. People you send this URL to can use [`Pkg.clone()`](@ref) to install the package and try
it out:

```julia-repl
julia> Pkg.clone("git://github.com/StefanKarpinski/FooBar.jl.git")
INFO: Cloning FooBar from git@github.com:StefanKarpinski/FooBar.jl.git
```

[^3]:
    Installing and using GitHub's ["hub" tool](https://github.com/github/hub) is highly recommended.
    It allows you to do things like run `hub create` in the package repo and have it automatically
    created via GitHub's API.

### Tagging and Publishing Your Package

!!! tip
    If you are hosting your package on GitHub, you can use the [attobot integration](https://github.com/attobot/attobot)
    to handle package registration, tagging and publishing.

Once you've decided that `FooBar` is ready to be registered as an official package, you can add
it to your local copy of `METADATA` using `PkgDev.register()`:

```julia-repl
julia> PkgDev.register("FooBar")
INFO: Registering FooBar at git://github.com/StefanKarpinski/FooBar.jl.git
INFO: Committing METADATA for FooBar
```

This creates a commit in the `~/.julia/v0.6/METADATA` repo:

```
$ cd ~/.julia/v0.6/METADATA && git show

commit 9f71f4becb05cadacb983c54a72eed744e5c019d
Author: Stefan Karpinski <stefan@karpinski.org>
Date:   Wed Oct 16 18:46:02 2013 -0400

    Register FooBar

diff --git a/FooBar/url b/FooBar/url
new file mode 100644
index 0000000..30e525e
--- /dev/null
+++ b/FooBar/url
@@ -0,0 +1 @@
+git://github.com/StefanKarpinski/FooBar.jl.git
```

This commit is only locally visible, however. To make it visible to the Julia community, you
need to merge your local `METADATA` upstream into the official repo. The `PkgDev.publish()` command
will fork the `METADATA` repository on GitHub, push your changes to your fork, and open a pull
request:

```julia-repl
julia> PkgDev.publish()
INFO: Validating METADATA
INFO: No new package versions to publish
INFO: Submitting METADATA changes
INFO: Forking JuliaLang/METADATA.jl to StefanKarpinski
INFO: Pushing changes as branch pull-request/ef45f54b
INFO: To create a pull-request open:

  https://github.com/StefanKarpinski/METADATA.jl/compare/pull-request/ef45f54b
```

!!! tip
    If `PkgDev.publish()` fails with error:

    ```
    ERROR: key not found: "token"
    ```

    then you may have encountered an issue from using the GitHub API on multiple systems. The solution
    is to delete the "Julia Package Manager" personal access token [from your Github account](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Ftokens)
    and try again.

    Other failures may require you to circumvent `PkgDev.publish()` by [creating a pull request on GitHub](https://help.github.com/articles/creating-a-pull-request/).
    See: [Publishing METADATA manually](@ref) below.

Once the package URL for `FooBar` is registered in the official `METADATA` repo, people know where
to clone the package from, but there still aren't any registered versions available. You can tag
and register it with the `PkgDev.tag()` command:

```julia-repl
julia> PkgDev.tag("FooBar")
INFO: Tagging FooBar v0.0.1
INFO: Committing METADATA for FooBar
```

This tags `v0.0.1` in the `FooBar` repo:

```
$ cd ~/.julia/v0.6/FooBar && git tag
v0.0.1
```

It also creates a new version entry in your local `METADATA` repo for `FooBar`:

```
$ cd ~/.julia/v0.6/FooBar && git show
commit de77ee4dc0689b12c5e8b574aef7f70e8b311b0e
Author: Stefan Karpinski <stefan@karpinski.org>
Date:   Wed Oct 16 23:06:18 2013 -0400

    Tag FooBar v0.0.1

diff --git a/FooBar/versions/0.0.1/sha1 b/FooBar/versions/0.0.1/sha1
new file mode 100644
index 0000000..c1cb1c1
--- /dev/null
+++ b/FooBar/versions/0.0.1/sha1
@@ -0,0 +1 @@
+84b8e266dae6de30ab9703150b3bf771ec7b6285
```

The `PkgDev.tag()` command takes an optional second argument that is either an explicit version
number object like `v"0.0.1"` or one of the symbols `:patch`, `:minor` or `:major`. These increment
the patch, minor or major version number of your package intelligently.

Adding a tagged version of your package will expedite the official registration into METADATA.jl
by collaborators. It is strongly recommended that you complete this process, regardless if your
package is completely ready for an official release.

As a general rule, packages should be tagged `0.0.1` first. Since Julia itself hasn't achieved
`1.0` status, it's best to be conservative in your package's tagged versions.

As with `PkgDev.register()`, these changes to `METADATA` aren't available to anyone else until
they've been included upstream. Again, use the `PkgDev.publish()` command, which first makes sure
that individual package repos have been tagged, pushes them if they haven't already been, and
then opens a pull request to `METADATA`:

```julia-repl
julia> PkgDev.publish()
INFO: Validating METADATA
INFO: Pushing FooBar permanent tags: v0.0.1
INFO: Submitting METADATA changes
INFO: Forking JuliaLang/METADATA.jl to StefanKarpinski
INFO: Pushing changes as branch pull-request/3ef4f5c4
INFO: To create a pull-request open:

  https://github.com/StefanKarpinski/METADATA.jl/compare/pull-request/3ef4f5c4
```

#### Publishing METADATA manually

If `PkgDev.publish()` fails you can follow these instructions to manually publish your package.

By "forking" the main METADATA repository, you can create a personal copy (of METADATA.jl) under
your GitHub account. Once that copy exists, you can push your local changes to your copy (just
like any other GitHub project).

1. go to [https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2FJuliaLang%2FMETADATA.jl%2Ffork](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2FJuliaLang%2FMETADATA.jl%2Ffork)
and create your own fork.

2. add your fork as a remote repository for the METADATA repository on your local computer (in
the terminal where USERNAME is your github username):

```
cd ~/.julia/v0.6/METADATA
git remote add USERNAME https://github.com/USERNAME/METADATA.jl.git
```

1. push your changes to your fork:

   ```
   git push USERNAME metadata-v2
   ```

4. If all of that works, then go back to the GitHub page for your fork, and click the "pull request"
link.

## Fixing Package Requirements

Si necesita corregir los requisitos registrados de una versión de paquete ya publicada, puede hacerlo simplemente editando los metadatos de esa versión, que seguirá teniendo el mismo hash de confirmación: el hash asociado a una versión es permanente:
```
$ cd ~/.julia/v0.6/METADATA/FooBar/versions/0.0.1 && cat requires
julia 0.3-
$ vi requires
```

Como el hash de confirmación (*commit*) permanece igual, el contenido del archivo `REQUIRE` que se retirará en el repositorio **no** coincidirá con los requisitos en `METADATA` después de dicho cambio; esto es inevitable. Sin embargo, cuando se fijan los requisitos en `METADATA` para una versión anterior de un paquete, también se debe corregir el archivo` REQUIRE` en la versión actual del paquete.

## Especificación de Requerimientos

El archivo `~/.julia/v0.6/REQUIRE`, el archivo` REQUIRE` dentro de los paquetes y los archivos `require` del paquete `METADATA` utilizan un formato simple basado en línea para expresar los rangos de las versiones del paquete que necesitan estar instalados. Los archivos `REQUIRE` y `METADATA requires` también deben incluir el rango de versiones de `julia` con las que se espera que funcione el paquete. Además, los paquetes pueden incluir un archivo `test/REQUIRE` para especificar paquetes adicionales que solo son necesarios para la prueba.

Así es cómo se analizan e interpretan estos archivos.

   * Todo lo que haya después de que una marca `#` se elimina de cada línea como un comentario.
   * Si solo queda espacio en blanco, la línea se ignorará.
   * Si quedan caracteres que no sean espacios en blanco, la línea es un requisito y el espacio en blanco se 
     divide en palabras.

El requisito más simple posible es simplemente el nombre del nombre de un paquete en una línea por sí mismo:

```julia
Distributions
```

Este requisito se satisface con cualquier versión del paquete `Distributions`. El nombre del paquete puede ir seguido de cero o más números de versión en orden ascendente, lo que indica intervalos aceptables de versiones de ese paquete. Una versión abre un intervalo, la siguiente la cierra y la siguiente abre un nuevo intervalo, y así sucesivamente; si se da un número impar de números de versión, las versiones arbitrariamente grandes satisfarán; si se proporciona un número par de números de versión, el último es un límite superior para los números de versión aceptables. Por ejemplo, la línea:

```
Distributions 0.1
```

se satisface con cualquier versión de `Distribuciones` superior o igual a` 0.1.0`. El sufijo de una versión con `-` también permite versiones preliminares. Por ejemplo:

```
Distributions 0.1-
```

se satisface con las versiones preliminares tales como `0.1-dev` o` 0.1-rc1`, o con cualquier versión mayor o igual a `0.1.0`.

Esta entrada de requisito:

```
Distributions 0.1 0.2.5
```

se satisface con versiones desde '0.1.0' hasta, pero sin incluir, '0.2.5'. Si quiere indicar que cualquier versión de `0.1.x` va a funcionar, querrá escribir:

```
Distributions 0.1 0.2-
```

If you want to start accepting versions after `0.2.7`, you can write:

```
Distributions 0.1 0.2- 0.2.7
```
Si una línea de requisitos tiene palabras iniciales que comienzan con `@`, es un requisito dependiente del sistema. Si su sistema coincide con estos condicionales del sistema, se incluye el requisito, de lo contrario, se ignora el requisito. Por ejemplo:

```
@osx Homebrew
```

requerirá el paquete `Homebrew` solo en los sistemas donde el sistema operativo es OS X. Las condiciones del sistema que actualmente son compatibles son (jerárquicamente):

  * `@unix`

      * `@linux`
      * `@bsd`

          * `@osx`
  * `@windows`

La condición `@unix` se cumple en todos los sistemas UNIX, incluidos Linux y BSD. Los condicionales de sistema negados también son compatibles al agregar un `!` Después del `@` inicial. Ejemplos:

```
@!windows
@unix @!osx
```

La primera condición se aplica a cualquier sistema excepto Windows y la segunda condición se aplica a cualquier sistema UNIX además de OS X.

Los controles de tiempo de ejecución para la versión actual de Julia se pueden realizar utilizando la variable incorporada `VERSION`, que es de tipo `VersionNumber`. Dicho código ocasionalmente es necesario para realizar un seguimiento de la funcionalidad nueva o obsoleta entre varias versiones de Julia. Ejemplos de controles de tiempo de ejecución:

```julia
VERSION < v"0.3-" #exclude all pre-release versions of 0.3

v"0.2-" <= VERSION < v"0.3-" #get all 0.2 versions, including pre-releases, up to the above

v"0.2" <= VERSION < v"0.3-" #To get only stable 0.2 versions (Note v"0.2" == v"0.2.0")

VERSION >= v"0.2.1" #get at least version 0.2.1
```

See the section on [version number literals](@ref man-version-number-literals) for a more complete description.
