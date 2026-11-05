# Proyecto de ciencia de datos: ¿qué gana una partida de League of Legends?

**Aprendiz:** Juan Camilo Montes Velásquez
**Programa:** Tecnólogo en Análisis y Desarrollo de Software (ADSO), SENA
**Evidencia:** GA1-220501094-01-AA1-EV05 – Diagnosticar la calidad de datos: preparación, exploración y documentación

## Propósito

Diagnosticar la calidad inicial de los datasets seleccionados para el proyecto antes de procesarlos y analizarlos.

**Pregunta analítica:** ¿Qué variables de los primeros 15 minutos de una partida (diferencia de oro, torres y objetivos neutrales) influyen más en la probabilidad de ganar en partidas profesionales de League of Legends, y se mantienen en partidas ranked de alto nivel?

## Datasets

| Dataset | Fuente | Archivo | Uso en el proyecto |
|---|---|---|---|
| Partidas profesionales 2025 | [Oracle's Elixir](https://oracleselixir.com/tools/downloads) | `2025_LoL_esports_match_data_from_OraclesElixir.csv` | Dataset principal: variables a los 15 minutos y resultado |
| Partidas ranked Diamante (10 min) | [Kaggle](https://www.kaggle.com/datasets/bobbyscience/league-of-legends-diamond-ranked-games-10-min) | `high_diamond_ranked_10min.csv` | Comparación con partidas ranked |

Los CSV **no se suben al repositorio** por su tamaño. Descárgalos de los enlaces anteriores y guárdalos en `data/raw/` con el nombre indicado.

Los datos de partidas son propiedad de Riot Games y su uso debe seguir sus términos y políticas.

## Estructura

```
proyecto-lol-ciencia-datos/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/                      <- CSV descargados (no se suben)
├── notebooks/
│   └── diagnostico_calidad.ipynb
└── reports/
    └── informe_diagnostico.md
```

## Resultados

El diagnóstico completo está en [`reports/informe_diagnostico.md`](reports/informe_diagnostico.md). En resumen:

- **Partidas profesionales:** 120.528 filas, pero son 10.044 partidas (12 filas cada una). Las variables a los 15 minutos faltan en las 820 partidas con datos parciales, que incluyen toda la LPL. Hay tipos incorrectos (`patch` decimal, `date` texto) y `year` no siempre coincide con la fecha.
- **Partidas ranked:** 9.879 partidas sin nulos ni duplicados, pero solo con datos hasta el minuto 10, sin parche, con columnas redundantes y con valores atípicos en los wards.

## Cómo reproducir el diagnóstico

Requisitos: Python 3.12 o superior (pandas 3 necesita 3.11+) y Git.

```bash
git clone https://github.com/juancamilomontes/proyecto-lol-ciencia-datos.git
cd proyecto-lol-ciencia-datos
python -m venv .venv
.\.venv\Scripts\Activate.ps1          # Windows PowerShell
# source .venv/bin/activate          # macOS / Linux
python -m pip install -r requirements.txt
jupyter lab
```

1. Descarga los dos CSV (ver **Datasets**) y guárdalos en `data/raw/`.
2. En JupyterLab abre `notebooks/diagnostico_calidad.ipynb` y ejecuta todas las celdas (*Run → Run All Cells*).

`requirements.txt` se generó con `python -m pip freeze` (pandas 3.1.1, NumPy 2.5.4, JupyterLab 4.6.5). Si PowerShell bloquea la activación, ejecuta una vez `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

## Flujo de trabajo con Git

Siguiendo la guía *Proyecto práctica: entorno y cuaderno*:

- `main` guarda solo versiones estables.
- `develop` integra el trabajo; cada fase se hace en una rama `feat/*` y se integra con `git merge --no-ff`.
- Cada versión queda marcada con un tag (`v0.1`, `v0.2`, …, `v1.0`).
