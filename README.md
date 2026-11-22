# Proyecto de ciencia de datos: ¿qué gana una partida de League of Legends?

**Aprendiz:** Juan Camilo Montes Velásquez
**Programa:** Tecnólogo en Análisis y Desarrollo de Software (ADSO), SENA

## Descripción

El proyecto busca identificar qué pasa en los primeros minutos de una partida de League of Legends que más se relaciona con ganarla, para que jugadores y entrenadores sepan en qué enfocarse al inicio basándose en datos y no en opiniones.

**Pregunta analítica:** ¿Qué variables de los primeros 15 minutos de una partida (diferencia de oro, torres y objetivos neutrales) influyen más en la probabilidad de ganar en partidas profesionales de League of Legends, y se mantienen en partidas ranked de alto nivel?

| Evidencia | Contenido | Notebook | Documento |
|---|---|---|---|
| GA1-220501094-01-AA1-EV05 – Diagnóstico de calidad | Carga, inspección y diagnóstico de los datasets | [`notebooks/data_quality.ipynb`](notebooks/data_quality.ipynb) | [`reports/data_quality.md`](reports/data_quality.md) |
| GA1-220501094-01-AA1-EV07 – Preparación, limpieza y estructuración | Limpieza, análisis con indicadores y estructuración de los datos | [`notebooks/data_preparation.ipynb`](notebooks/data_preparation.ipynb) | Este README (sección *Transformaciones*) |

## Fuentes de datos

| Dataset | Fuente | Archivo | Contenido | Uso |
|---|---|---|---|---|
| Partidas profesionales 2025 | [Oracle's Elixir](https://oracleselixir.com/tools/downloads) | `2025_LoL_esports_match_data_from_OraclesElixir.csv` | 120.528 filas × 165 columnas: 10.044 partidas de 45 ligas, con 12 filas por partida (10 jugadores + 2 equipos) | Dataset principal: variables a los 10 y 15 minutos y resultado |
| Partidas ranked Diamante | [Kaggle](https://www.kaggle.com/datasets/bobbyscience/league-of-legends-diamond-ranked-games-10-min) (licencia CC0) | `high_diamond_ranked_10min.csv` | 9.879 partidas × 40 columnas, una fila por partida con datos de los dos equipos a los 10 minutos | Comprobar si los patrones se mantienen en ranked |

Los CSV originales **no se suben al repositorio** por su tamaño. Descárgalos de los enlaces anteriores y guárdalos sin modificar en `data/raw/` con el nombre indicado. Los datos de partidas son propiedad de Riot Games y su uso debe seguir sus términos y políticas.

## Estructura

```
proyecto-lol-ciencia-datos/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/                              <- datasets originales, sin modificar (no se suben)
│   └── processed/                        <- datasets limpios y estructurados (EV07)
├── notebooks/
│   ├── data_quality.ipynb                <- diagnóstico de calidad (EV05)
│   └── data_preparation.ipynb            <- limpieza, análisis y estructuración (EV07)
└── reports/
    └── data_quality.md                   <- informe del diagnóstico (EV05)
```

## Transformaciones realizadas (EV07)

Todas se hacen en `notebooks/data_preparation.ipynb`, sobre copias en memoria. Los archivos de `data/raw/` no se modifican.

### 1. Limpieza y estandarización

| Problema (diagnóstico) | Tratamiento |
|---|---|
| 12 filas por partida en el dataset profesional | Solo las filas de equipo (`position == 'team'`): 20.088 filas, 2 por partida |
| 165 columnas, muchas de fin de partida | 28 columnas de inicio de partida, renombradas a inglés `snake_case`. Se excluyen los totales de fin de partida para evitar fuga de información |
| Nulos en las variables a 10 y 15 min (1.640 filas de partidas `partial`, casi todas de la LPL) | Se conservan y se marcan con `has_early_stats = False`; se excluyen del análisis a 15 minutos |
| `split` vacío en torneos sin splits (2.724) | Rellenado con `'No split'` |
| 1 partida sin first blood pero con 38 asesinatos | `first_blood` como nulo en esa partida (dato no registrado) |
| `year` = 2026 en partidas de 2025 | Renombrado a `season`; el periodo se toma de `date` |
| `patch` decimal, `date` texto, banderas y diferencias como `float64`, `gameId` numérico | `patch` a texto (`'15.10'`), `date` a fecha, enteros *nullable* (`Int8`/`Int16`/`Int32`), categorías y `gameId` a texto |
| Columnas redundantes en ranked (`redGoldDiff`, `EliteMonsters`, `CSPerMin`…) y wards atípicos | Se eliminan 8 columnas redundantes y 12 fuera del alcance de la pregunta (incluidos los wards) |

La limpieza está en funciones reutilizables encadenadas con `pipe()`. Cada paso queda registrado en `cleaning_log.csv` y el resultado se valida con `assert`: 2 equipos y un ganador por partida, diferencias de oro simétricas, formatos y fechas correctos.

### 2. Filtrado, agrupación e indicadores

Se usan filtros booleanos, `between`, `isin` y `query`, ordenamiento con `nsmallest`/`nlargest`, `groupby` + `agg`, categorización con `pd.cut` y `pivot_table`. Indicadores principales (partidas profesionales con datos a 15 minutos):

| Indicador | Resultado |
|---|---|
| Tasa de victoria con el primer objetivo | Primera torre 70,1 %, primer heraldo 66,5 %, first blood 57,8 %, primer dragón 57,6 % |
| Tasa de victoria por diferencia de oro a los 15 min | Del 6,1 % (más de 3.000 abajo) al 93,9 % (más de 3.000 arriba); 50 % en partidas parejas |
| Diferencias promedio de los ganadores a los 15 min | +1.345 de oro, +952 de experiencia, +17 de CS y +1,6 asesinatos netos |
| Correlación con ganar | Oro 0,523 · experiencia 0,492 · CS 0,449 · primera torre 0,402 · primer heraldo 0,329 · first blood 0,155 · primer dragón 0,152 |

### 3. Combinación y estructuración

- **`merge`** del equipo azul con el rojo de cada partida profesional (`validate='1:1'`, `indicator=True`), con comprobación de resultados complementarios y diferencias simétricas.
- **Reestructuración del ranked** de una fila por partida a una fila por equipo con `concat`, usando el mismo esquema que el profesional.
- **`concat` de las dos fuentes** en un dataset consolidado a los 10 minutos, con una columna `source`. Los dos datasets **no se unen con `merge`** porque no comparten ninguna clave: son partidas distintas, y unirlos sería una combinación artificial.
- Verificación de claves únicas, cantidad de registros e integridad antes de exportar, y relectura de los archivos exportados.

**Resultado:** con más de 1.000 de oro de ventaja a los 10 minutos se gana el 80,8 % de las partidas profesionales y el 79,9 % de las ranked. Los patrones se mantienen en los dos niveles.

### Datasets procesados (`data/processed/`)

| Archivo | Una fila es | Filas | Contenido |
|---|---|---|---|
| `pro_teams_clean.parquet` | Un equipo en una partida profesional | 20.088 | Dataset profesional limpio, incluidas las partidas `partial` (marcadas con `has_early_stats`) |
| `pro_teams_15min.csv` | Un equipo en una partida profesional | 18.448 | Solo partidas con datos a 15 minutos completos; sin nulos. **Dataset principal de análisis** |
| `pro_games.csv` | Una partida profesional | 10.044 | Equipo azul y rojo en columnas (resultado del `merge`) |
| `ranked_teams_10min.csv` | Un equipo en una partida ranked | 19.758 | Dataset ranked reestructurado |
| `early_game_10min.csv` / `.parquet` | Un equipo | 38.206 | Las dos fuentes con el mismo esquema a los 10 minutos (columna `source`); sin nulos |
| `win_rate_by_objective_pro.csv`, `win_rate_by_objective_ranked.csv` | Un objetivo | 4 | Indicador 1 |
| `win_rate_by_gold_diff_15.csv` | Un tramo de oro | 5 | Indicador 2 |
| `correlation_with_win_15min.csv` | Una variable | 8 | Indicador 4 |
| `cleaning_log.csv` | Un paso del pipeline | 11 | Registro de cambios de la limpieza |
| `quality_before_after.csv` | Un dataset | 7 | Calidad antes y después del procesamiento |

## Cómo ejecutarlo

Requisitos: Python 3.12 o superior (pandas 3 necesita 3.11+) y Git.

```bash
git clone https://github.com/juancamilomontes/proyecto-lol-ciencia-datos.git
cd proyecto-lol-ciencia-datos
git checkout develop
python -m venv .venv
.\.venv\Scripts\Activate.ps1          # Windows PowerShell
# source .venv/bin/activate          # macOS / Linux
python -m pip install -r requirements.txt
jupyter lab
```

1. Descarga los dos CSV (ver **Fuentes de datos**) y guárdalos en `data/raw/`.
2. Abre `notebooks/data_quality.ipynb` para ver el diagnóstico de calidad (EV05).
3. Abre `notebooks/data_preparation.ipynb` y ejecuta todas las celdas en orden (*Run → Run All Cells*). Cada sección usa las variables de la anterior. Tarda menos de un minuto y vuelve a generar los archivos de `data/processed/`.

`requirements.txt` se generó con `python -m pip freeze` (pandas 3.1.1, NumPy 2.5.4, PyArrow 26.0.1 para Parquet y JupyterLab 4.6.5). Si PowerShell bloquea la activación, ejecuta una vez `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

## Flujo de trabajo con Git

- `main` guarda las versiones estables.
- `develop` integra el trabajo de cada actividad.
- Cada actividad se desarrolla en una rama propia y se integra en `develop` con `git merge --no-ff`: `feat/entorno` (entorno) y `feature/project-data-preparation` (EV07).
- Las versiones del diagnóstico están marcadas con tags (`v0.1`, `v0.2`, `v1.0`).
