# [Consejos de *Workflow*](@id man-workflow-tips)

He aquí algunos consejos para trabajar de forma eficiente con Julia.

## *Workflow* basado en el REPL

Como ya se explicó en [Interactuando con Julia](@ref), el REPL de Julia proporciona una funcionalidad completa que facilita un flujo de trabajo eficiente e interactivo. Aquí hay algunos consejos que pueden mejorar aún más su experiencia en la línea de mandatos.

### A *workflow* básico editor/REPL 

Los flujos de trabajo de Julia más básicos implican el uso de un editor de texto junto con la línea de comando `julia`. Un patrón común incluye los siguientes elementos:

  * **Ponga el código en desarrollo dentro de un módulo temporal.** Cree un fichero, digamos `Tmp.jl`, e incluya dentro de él

    ```
    module Tmp

    <your definitions here>

    end
    ```
  * **Ponga el código de test en otro fichero.** Cree otro fichero, digamos `tst.jl`, que comience con

    ```julia
    import Tmp
    ```

    e incluya pruebas para el contenido de `Tmp`. El valor de usar `import` contra` using` es que puede llamar 
    a `reload("Tmp")` en lugar de tener que reiniciar el REPL cuando cambian sus definiciones. Por supuesto, 
    el costo es la necesidad de anteponer `Tmp` a los usos de los nombres definidos en su módulo. (Puede reducir
    ese costo manteniendo el nombre de su módulo corto).
    
    ```
    module Tst
        using Tmp

        <scratch work>

    end
    ```

    La ventaja es que ahora puede hacer `using Tmp` en su código de prueba y, por lo tanto, puede 
    evitar anteponer` Tmp` a todas partes. La desventaja es que el código ya no se puede copiar 
    selectivamente al REPL sin algunos ajustes.
    
  * **Enjabonar. Enjuagar. Repetir.** Explore ideas en el prompt de  `julia`. Guarde buenas ideas en
    `tst.jl`. Ocasionalmente, reinicie el REPL, haciendo

    ```julia
    reload("Tmp")
    include("tst.jl")
    ```

### Simplifique la inicialización

Para simplificar el reinicio de REPL, coloque el código de inicialización específico del proyecto en 
un archivo, diga `_init.jl`, que puedes ejecutar en el inicio al emitir el mandato:

```
julia -L _init.jl
```

Si además agrega lo siguiente a su archivo `.juliarc.jl`

```julia
isfile("_init.jl") && include(joinpath(pwd(), "_init.jl"))
```

entonces llamar a `julia` desde este directorio ejecutará el código de inicialización sin el argumento 
de línea de mandatos adicional.

## Browser-based workflow

Es también posible interactuar con un REPL Julia en el navegador a través de [IJulia](https://github.com/JuliaLang/IJulia.jl).
Ver la documentación del paquete para más detalles.
