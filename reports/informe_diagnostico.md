# Informe de diagnóstico de calidad de datos

**Aprendiz:** Juan Camilo Montes Velásquez
**Evidencia:** GA1-220501094-01-AA1-EV05 – Diagnosticar la calidad de datos: preparación, exploración y documentación
**Fecha:** 4 de noviembre de 2026

Todos los resultados de este informe salen del notebook [`notebooks/diagnostico_calidad.ipynb`](../notebooks/diagnostico_calidad.ipynb).

## 1. Nombre del proyecto y pregunta analítica

**Proyecto:** ¿Qué gana una partida de League of Legends?

**Pregunta analítica:** ¿Qué variables de los primeros 15 minutos de una partida (diferencia de oro, torres y objetivos neutrales) influyen más en la probabilidad de ganar en partidas profesionales de League of Legends, y se mantienen en partidas ranked de alto nivel?

La pregunta viene de la EV02. La ajusté porque en la EV03 encontré que el dataset ranked de Kaggle solo llega al minuto 10. Agregué un dataset de partidas profesionales (Oracle's Elixir, fuente ya identificada en la EV02) que sí trae datos a los 15 minutos.

## 2. Descripción de los datasets

| Dataset | Fuente | Contenido | Uso |
|---|---|---|---|
| `2025_LoL_esports_match_data_from_OraclesElixir.csv` | Oracle's Elixir | Partidas profesionales de 2025 de 45 ligas, con estadísticas de cada jugador y de cada equipo a los 10, 15, 20 y 25 minutos, y el resultado | Dataset principal |
| `high_diamond_ranked_10min.csv` | Kaggle (bobbyscience), licencia CC0 | Partidas ranked de nivel Diamante a Maestro, con estadísticas de ambos equipos a los 10 minutos y si ganó el equipo azul | Comparación con ranked |

## 3. Resultados de la inspección estructural

Para cada dataset ejecuté `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include='object')`.

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| Partidas profesionales | 120.528 | 165 | 122 float64, 23 str, 20 int64 | 12 filas por partida (10 jugadores + 2 equipos): son 10.044 partidas. Muchas columnas con nulos. |
| Partidas ranked | 9.879 | 40 | 34 int64, 6 float64 | 1 fila por partida. Sin columnas de texto, así que `describe(include='object')` no aplica. Columnas en pares azul/rojo. |

El hallazgo estructural más importante es que **en el dataset profesional una fila no es una partida**. Igual que en Instacart, donde un registro no es un cliente, aquí hay que distinguir filas, partidas, equipos y jugadores. Para mi pregunta sirven las filas con `position == "team"`.

## 4. Valores faltantes, tipos de datos e inconsistencias

### Valores faltantes
- **Dataset profesional:** 129 de 165 columnas tienen al menos un nulo. La mayoría son **datos que no aplican**: las filas de equipo no tienen `champion` ni `playername`, y las de jugador no tienen `towers` ni `firstdragon`.
- **Los nulos que sí importan** están en las variables a los 15 minutos (`golddiffat15`, `xpdiffat15`, `csdiffat15`, `firstherald`). Faltan en 1.640 filas de equipo, que son exactamente las 820 partidas marcadas como `datacompleteness = "partial"`. De ellas, 805 son de la **LPL**, es decir, todas las partidas de esa liga.
- Otros nulos: `split` (16.344, torneos sin splits como Worlds o MSI), `teamid` (3.636, ligas menores), `ban1` (414) y `url` (solo existe en las partidas parciales).
- **Dataset ranked:** no tiene ningún nulo.

### Tipos de datos
- `patch` es `float64`: el parche 15.10 se ve como `15.1`. Debería ser texto.
- `date` es texto (`str`), no fecha.
- `firstdragon`, `firstherald`, `firsttower` y `firstblood` son `float64` aunque solo valen 0 o 1 (por los nulos de las filas de jugador).
- `gameId` (ranked) es `int64`, pero es un identificador; `describe()` le calcula un promedio sin sentido.

### Posibles inconsistencias
- `year` dice **2026** en 2.976 filas cuyas fechas son de **2025** (julio a diciembre). Parece ser el año de la temporada, no el de la partida.
- 1 partida sin `firstblood`, 7 sin `firstdragon` y 19 sin `firstherald` (ningún equipo los tiene).
- Ranked: `blueWardsPlaced` tiene una mediana de 16 y un máximo de 250 en 10 minutos. 108 partidas superan los 100 wards, lo cual no es realista.
- Ranked: hay columnas redundantes (`blueGoldDiff = -redGoldDiff`, `blueEliteMonsters = blueDragons + blueHeralds`, `blueCSPerMin = blueTotalMinionsKilled / 10`).

Lo que sí está bien: en todas las partidas profesionales hay **exactamente un ganador**, `firsttower` siempre lo tiene un solo equipo, y ningún dataset tiene filas duplicadas ni valores negativos en conteos.

## 5. Principales hallazgos de calidad por dataset

| Dataset | Problema identificado | Evidencia | Posible impacto | Acción propuesta |
|---|---|---|---|---|
| Profesional | Filas que no son partidas | 12 filas por `gameid` | Contar filas como partidas multiplica todo por 12 | Filtrar `position == "team"` |
| Profesional | Nulos en variables a 15 min | 1.640 filas de equipo sin `golddiffat15`, todas `partial` | La LPL queda fuera del análisis | Excluir partidas `partial` y documentarlo |
| Profesional | Nulos que no aplican | `champion` vacío en equipos, `towers` vacío en jugadores | Confunde si no se separan las filas | No imputar; separar equipo y jugador |
| Profesional | Tipo incorrecto en `patch` | `float64`, 15.10 → 15.1 | Se mezclan parches al agrupar | Convertir a texto |
| Profesional | Tipo incorrecto en `date` | `str` | No se puede filtrar por fecha | `pd.to_datetime()` |
| Profesional | `year` no coincide con `date` | 2.976 filas con 2026 y fecha 2025 | Filtrar por `year` da resultados equivocados | Usar `date` para el periodo |
| Profesional | Partidas sin primer objetivo | 1 sin firstblood, 7 sin firstdragon, 19 sin firstherald | Pequeño sesgo | Revisarlas y decidir si se excluyen |
| Ranked | Valores atípicos en wards | Mediana 16, máximo 250 | Distorsionan promedios | Revisar con RIQ; recortar o excluir |
| Ranked | Columnas redundantes | `blueGoldDiff = -redGoldDiff`, etc. | Información repetida | Dejar una columna por variable |
| Ranked | Solo 10 minutos y sin parche | No hay `at15`, `patch` ni `date` | No responde a 15 min; versión desconocida | Comparar con las columnas `at10` del profesional |
| Ranked | Identificador numérico | `gameId` es `int64` | Estadísticas sin sentido | Convertir a texto |

## 6. Acciones de limpieza y preparación propuestas

1. Trabajar con las filas de equipo (`position == "team"`) del dataset profesional.
2. Excluir las 820 partidas con `datacompleteness == "partial"` en el análisis a 15 minutos, y aclarar que la LPL no queda representada.
3. Convertir `patch` y `gameId` a texto, y `date` a fecha.
4. Usar `date` en lugar de `year` para delimitar el periodo.
5. Revisar las partidas sin primer objetivo y los valores atípicos de wards, y decidir si se excluyen.
6. Eliminar las columnas redundantes del dataset ranked.
7. Llevar los dos datasets a la misma forma (una fila por equipo) y compararlos con las variables a los 10 minutos, que son las que tienen en común.

En esta etapa no se aplicó ninguna limpieza; solo se diagnosticó, como pide la actividad.

## 7. Conclusiones

Los datos están en condiciones de continuar con el proyecto, siempre que se apliquen las acciones anteriores.

- El **dataset profesional** responde la pregunta: tiene las variables a los 15 minutos y el resultado, y su estructura es consistente (un ganador por partida). Su principal problema es que el 8 % de las partidas, toda la LPL, no tiene datos a los 15 minutos.
- El **dataset ranked** está limpio, pero es limitado: solo llega al minuto 10, no indica versión del juego, repite información y tiene valores atípicos.
- La lección principal es la misma que en la EV04: **antes de calcular hay que saber qué representa cada fila y por qué faltan los datos**. Muchos nulos no son errores, y un mismo dataset puede tener filas de distinto tipo.
