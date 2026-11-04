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

## Cómo reproducir el diagnóstico

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt
jupyter notebook notebooks/diagnostico_calidad.ipynb
```
