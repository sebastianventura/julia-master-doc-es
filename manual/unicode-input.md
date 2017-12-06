# Unicode Input

La siguiente tabla enumera los caracteres Unicode que pueden ingresarse mediante la terminación de pestañas de las abreviaturas similares a LaTeX en Julia REPL (y en otros entornos de edición). También puede obtener información sobre cómo escribir un símbolo ingresándolo en la ayuda REPL, es decir, escribiendo `?` Y luego ingresando el símbolo en REPL (por ejemplo, mediante copiar y pegar desde algún lugar donde vio el símbolo).

!!! warning

    Puede parecer que esta tabla contiene caracteres faltantes en la segunda columna, 
    o incluso muestra caracteres que son inconsistentes con los caracteres tal como 
    se representan en Julia REPL. En estos casos, se recomienda encarecidamente a los 
    usuarios que comprueben la elección de las fuentes en su navegador y entorno REPL, 
    ya que existen problemas conocidos con los glifos en muchas fuentes.

```@eval
#
# Generate a table containing all LaTeX and Emoji tab completions available in the REPL.
#

const NBSP = '\u00A0'

function tab_completions(symbols...)
    completions = Dict{String, Vector{String}}()
    for each in symbols, (k, v) in each
        completions[v] = push!(get!(completions, v, String[]), k)
    end
    return completions
end

function unicode_data()
    file = normpath(JULIA_HOME, "..", "..", "doc", "UnicodeData.txt")
    names = Dict{UInt32, String}()
    open(file) do unidata
        for line in readlines(unidata)
            id, name, desc = split(line, ";")[[1, 2, 11]]
            codepoint = parse(UInt32, "0x$id")
            names[codepoint] = titlecase(lowercase(name == "" ? desc : desc == "" ? name : "$name / $desc"))
        end
    end
    return names
end

# Surround combining characters with no-break spaces (i.e '\u00A0'). Follows the same format
# for how unicode is displayed on the unicode.org website:
# http://unicode.org/cldr/utility/character.jsp?a=0300
function fix_combining_chars(char)
    cat = Base.UTF8proc.category_code(char)
    return cat == 6 || cat == 8 ? "$NBSP$char$NBSP" : "$char"
end


function table_entries(completions, unicode_dict)
    entries = [[
        "Code point(s)", "Character(s)",
        "Tab completion sequence(s)", "Unicode name(s)"
    ]]
    for (chars, inputs) in sort!(collect(completions), by = first)
        code_points, unicode_names, characters = String[], String[], String[]
        for char in chars
            push!(code_points, "U+$(uppercase(hex(char, 5)))")
            push!(unicode_names, get(unicode_dict, UInt32(char), "(No Unicode name)"))
            push!(characters, isempty(characters) ? fix_combining_chars(char) : "$char")
        end
        push!(entries, [
            join(code_points, " + "), join(characters),
            join(inputs, ", "), join(unicode_names, " + ")
        ])
    end
    return Markdown.Table(entries, [:l, :l, :l, :l])
end

table_entries(
    tab_completions(
        Base.REPLCompletions.latex_symbols,
        Base.REPLCompletions.emoji_symbols
    ),
    unicode_data()
)
```
