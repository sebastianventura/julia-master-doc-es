# [Fecha y Tiempo](@id stdlib-dates)

## Tipos para Fechas y Tiempo

```@docs
Base.Dates.Period
Base.Dates.CompoundPeriod
Base.Dates.Instant
Base.Dates.UTInstant
Base.Dates.TimeType
Base.Dates.DateTime
Base.Dates.Date
Base.Dates.Time
```

## Funciones para Fechas

Todas las funciones para fechas están definidas en el módulo `Dates`; nótese que sólo se han exportado las funciones `Date`, `DateTime` y `now`; para usar todas las dems funciones de `Dates`, es necesario prefijar cada llamada a funcion con `Dates.`, por ejemplo,`Dates.dayofweek(dt)`. Alternativamente, uno puede escribir `using Base.Dates` para llevar todas las funciones exportadas en `Main` para ser usadas sin el prefijo `Dates.`

```@docs
Base.Dates.DateTime(::Int64, ::Int64, ::Int64, ::Int64, ::Int64, ::Int64, ::Int64)
Base.Dates.DateTime(::Base.Dates.Period...)
Base.Dates.DateTime(::Function, ::Any...)
Base.Dates.DateTime(::Base.Dates.TimeType)
Base.Dates.DateTime(::AbstractString, ::AbstractString)
Base.Dates.format
Base.Dates.DateFormat
Base.Dates.@dateformat_str
Base.Dates.DateTime(::AbstractString, ::Base.Dates.DateFormat)
Base.Dates.Date(::Int64, ::Int64, ::Int64)
Base.Dates.Date(::Base.Dates.Period...)
Base.Dates.Date(::Function, ::Any, ::Any, ::Any)
Base.Dates.Date(::Base.Dates.TimeType)
Base.Dates.Date(::AbstractString, ::AbstractString)
Base.Dates.Date(::AbstractString, ::Base.Dates.DateFormat)
Base.Dates.Time(::Int64::Int64, ::Int64, ::Int64, ::Int64, ::Int64)
Base.Dates.Time(::Base.Dates.TimePeriod...)
Base.Dates.Time(::Function, ::Any...)
Base.Dates.Time(::Base.Dates.DateTime)
Base.Dates.now()
Base.Dates.now(::Type{Base.Dates.UTC})
Base.eps
```

### Funciones Accesoras

```@docs
Base.Dates.year
Base.Dates.month
Base.Dates.week
Base.Dates.day
Base.Dates.hour
Base.Dates.minute
Base.Dates.second
Base.Dates.millisecond
Base.Dates.microsecond
Base.Dates.nanosecond
Base.Dates.Year(::Base.Dates.TimeType)
Base.Dates.Month(::Base.Dates.TimeType)
Base.Dates.Week(::Base.Dates.TimeType)
Base.Dates.Day(::Base.Dates.TimeType)
Base.Dates.Hour(::DateTime)
Base.Dates.Minute(::DateTime)
Base.Dates.Second(::DateTime)
Base.Dates.Millisecond(::DateTime)
Base.Dates.Microsecond(::Dates.Time)
Base.Dates.Nanosecond(::Dates.Time)
Base.Dates.yearmonth
Base.Dates.monthday
Base.Dates.yearmonthday
```

### Funciones de Consulta

```@docs
Base.Dates.dayname
Base.Dates.dayabbr
Base.Dates.dayofweek
Base.Dates.dayofmonth
Base.Dates.dayofweekofmonth
Base.Dates.daysofweekinmonth
Base.Dates.monthname
Base.Dates.monthabbr
Base.Dates.daysinmonth
Base.Dates.isleapyear
Base.Dates.dayofyear
Base.Dates.daysinyear
Base.Dates.quarterofyear
Base.Dates.dayofquarter
```

### Funciones de Ajuste

```@docs
Base.trunc(::Base.Dates.TimeType, ::Type{Base.Dates.Period})
Base.Dates.firstdayofweek
Base.Dates.lastdayofweek
Base.Dates.firstdayofmonth
Base.Dates.lastdayofmonth
Base.Dates.firstdayofyear
Base.Dates.lastdayofyear
Base.Dates.firstdayofquarter
Base.Dates.lastdayofquarter
Base.Dates.tonext(::Base.Dates.TimeType, ::Int)
Base.Dates.toprev(::Base.Dates.TimeType, ::Int)
Base.Dates.tofirst
Base.Dates.tolast
Base.Dates.tonext(::Function, ::Base.Dates.TimeType)
Base.Dates.toprev(::Function, ::Base.Dates.TimeType)
```

### Periodos

```@docs
Base.Dates.Period(::Any)
Base.Dates.CompoundPeriod(::Vector{<:Base.Dates.Period})
Base.Dates.default
```

### Funciones de Redondeo

`Date` and `DateTime` values can be rounded to a specified resolution (e.g., 1 month or 15 minutes)
with `floor`, `ceil`, or `round`.

```@docs
Base.floor(::Base.Dates.TimeType, ::Base.Dates.Period)
Base.ceil(::Base.Dates.TimeType, ::Base.Dates.Period)
Base.round(::Base.Dates.TimeType, ::Base.Dates.Period, ::RoundingMode{:NearestTiesUp})
```

Las siguientes funciones no son exportadas:

```@docs
Base.Dates.floorceil
Base.Dates.epochdays2date
Base.Dates.epochms2datetime
Base.Dates.date2epochdays
Base.Dates.datetime2epochms
```

### Funciones de Conversión

```@docs
Base.Dates.today
Base.Dates.unix2datetime
Base.Dates.datetime2unix
Base.Dates.julian2datetime
Base.Dates.datetime2julian
Base.Dates.rata2datetime
Base.Dates.datetime2rata
```

### Constantes

Días de la semana:

| Variable    | Abbr. | Value (Int) |
|:----------- |:----- |:----------- |
| `Monday`    | `Mon` | 1           |
| `Tuesday`   | `Tue` | 2           |
| `Wednesday` | `Wed` | 3           |
| `Thursday`  | `Thu` | 4           |
| `Friday`    | `Fri` | 5           |
| `Saturday`  | `Sat` | 6           |
| `Sunday`    | `Sun` | 7           |

Meses del año:

| Variable    | Abbr. | Value (Int) |
|:----------- |:----- |:----------- |
| `January`   | `Jan` | 1           |
| `February`  | `Feb` | 2           |
| `March`     | `Mar` | 3           |
| `April`     | `Apr` | 4           |
| `May`       | `May` | 5           |
| `June`      | `Jun` | 6           |
| `July`      | `Jul` | 7           |
| `August`    | `Aug` | 8           |
| `September` | `Sep` | 9           |
| `October`   | `Oct` | 10          |
| `November`  | `Nov` | 11          |
| `December`  | `Dec` | 12          |
