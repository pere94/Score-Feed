# ScoreDemandSeasonality

Documento de referencia del componente `ScoreDemandSeasonality` ubicado en `scripts/scores/score_demand_seasonality/score_demand_seasonality.py`.

## Objetivo
Calcular métricas de demanda y estacionalidad a partir de series mensuales de volúmenes de búsqueda (idealmente 12 meses por keyword) y combinar ambos indicadores en un score final entre 0 y 1.

Casos de uso típicos:
- Detectar productos con demanda muy alta en el presente (picos recientes).
- Identificar patrones estacionales marcados (ej. verano/invierno, campañas, etc.).
- Priorizar acciones de marketing o pricing en función del momento del año.

## Cómo funciona (resumen)
El módulo expone tres métodos estáticos:
- `score_high_demand(volumes)`
- `score_seasonality(volumes)`
- `score_final(params)`

Descripción de los cálculos (formato Python, fácil de copiar/pegar):

### 1) Alta demanda reciente (último mes vs. máximo histórico)
```python
import numpy as np

# volumes: lista de volúmenes mensuales (más reciente al final)
if not volumes:
    high_demand_score = 0.0
else:
    current_month_volume = float(volumes[-1])
    historical_max_volume = float(np.max(volumes))
    if (not np.isfinite(current_month_volume)
            or not np.isfinite(historical_max_volume)
            or historical_max_volume <= 0):
        high_demand_score = 0.0
    else:
        current_vs_max_ratio = current_month_volume / historical_max_volume
        if current_vs_max_ratio >= 0.9:
            high_demand_score = 1.0  # muy alta demanda
        elif current_vs_max_ratio >= 0.8:
            high_demand_score = 0.8  # alta demanda
        elif current_vs_max_ratio >= 0.7:
            high_demand_score = 0.6  # demanda moderada
        else:
            high_demand_score = current_vs_max_ratio  # demanda baja (valor continuo)
```

### 2) Estacionalidad (variabilidad + intensidad de temporada)
```python
import numpy as np

# Recomendado: 12 meses. Mínimo: 6 meses. Si media = 0 => 0.0
if volumes is None or len(volumes) < 6:
    seasonality_score = 0.0
else:
    volumes_arr = np.array(volumes, dtype=float)
    mean_volume = float(np.mean(volumes_arr))
    if mean_volume <= 0:
        seasonality_score = 0.0
    else:
        std_volume = float(np.std(volumes_arr))
        # Coeficiente de variación
        coefficient_of_variation = std_volume / mean_volume

        # Tercio superior vs. tercio inferior
        num_months = len(volumes_arr)
        tertile_size = max(1, num_months // 3)
        sorted_volumes = np.sort(volumes_arr)
        high_season_volumes = sorted_volumes[-tertile_size:]
        low_season_volumes = sorted_volumes[:tertile_size]
        avg_high_season = float(np.mean(high_season_volumes))
        avg_low_season = float(np.mean(low_season_volumes))

        # Ratio estacional e intensidad normalizada
        seasonal_ratio = (avg_high_season / avg_low_season) if avg_low_season > 0 else 0.0
        seasonal_intensity = min(1.0, max(0.0, (seasonal_ratio - 1.0) / 2.0))

        # Score combinado: 30% variabilidad + 70% intensidad
        variability_score = min(1.0, coefficient_of_variation)
        seasonality_score = 0.3 * variability_score + 0.7 * seasonal_intensity

        # Bonus por amplitud marcada
        if seasonal_ratio > 3.0:
            seasonality_score *= 1.2
        seasonality_score = float(np.clip(seasonality_score, 0.0, 1.0))
```

### 3) Score final (combinación por keyword)
```python
import numpy as np

# all_volumes: lista de listas (una lista de 12 meses por keyword)
# pesos por defecto
weight_high_demand = 0.8
weight_seasonality = 0.2

high_demand_scores = []
seasonality_scores = []
for volumes_for_keyword in all_volumes:
    # Omitir keywords con menos de 12 meses
    if not volumes_for_keyword or len(volumes_for_keyword) < 12:
        continue
    high_demand_scores.append(score_high_demand(volumes_for_keyword))
    seasonality_scores.append(score_seasonality(volumes_for_keyword))

mean_high_demand_score = float(np.mean(high_demand_scores)) if high_demand_scores else 0.0
mean_seasonality_score = float(np.mean(seasonality_scores)) if seasonality_scores else 0.0
final_combined_score = (weight_high_demand * mean_high_demand_score
                        + weight_seasonality * mean_seasonality_score)
final_combined_score = max(0.0, min(1.0, final_combined_score))

# Retorna objeto ModelResponseScoreDemandSeasonality con:
# - score_final: final_combined_score
# - s_high_demand: mean_high_demand_score  
# - s_seasonality: mean_seasonality_score
```

## API pública

### Clase `ScoreDemandSeasonality`

- `score_high_demand(volumes: list[int|float]) -> float`
  - Usa el último valor como actual y el máximo como referencia.
  - Devuelve un score ∈ [0,1] siguiendo el mapeo descrito arriba.

- `score_seasonality(volumes: list[int|float]) -> float`
  - Requiere ≥6 valores; recomendado 12.
  - Combina CV con intensidad estacional basada en terciles alto/bajo.
  - Devuelve un score ∈ [0,1].

- `score_final(params: ModelParamsScoreDemandSeasonality) -> ModelResponseScoreDemandSeasonality`
  - Combina las métricas promediadas por keyword con pesos configurables.
  - Devuelve un objeto `ModelResponseScoreDemandSeasonality` con tres campos:
    - `score_final`: Score combinado final ∈ [0,1]
    - `s_high_demand`: Media del score de alta demanda ∈ [0,1]
    - `s_seasonality`: Media del score de estacionalidad ∈ [0,1]

### Modelo de entrada
`ModelParamsScoreDemandSeasonality` (Pydantic):
- `volumes: List[List[int|float]]` — lista de 12 meses por keyword (más reciente al final).
- `weight_high_demand: float = 0.8`
- `weight_seasonality: float = 0.2`

### Modelo de respuesta  
`ModelResponseScoreDemandSeasonality` (Pydantic):
- `score_final: float` — Score final combinado ∈ [0,1]
- `s_high_demand: float` — Media del score de alta demanda ∈ [0,1]
- `s_seasonality: float` — Media del score de estacionalidad ∈ [0,1]

## Ejemplos de uso

> Nota: Los ejemplos usan datos ficticios. El bloque `if __name__ == "__main__"` del script demuestra cómo obtener datos reales vía Google Ads Keyword Planner, lo que requiere credenciales/configuración previas.

### 1) Score de alta demanda
```python
from scripts.scores.score_demand_seasonality.score_demand_seasonality import ScoreDemandSeasonality

volumes = [500, 600, 800, 900, 1100, 1300, 1200, 1400, 1600, 1700, 1800, 1750]
score_hd = ScoreDemandSeasonality.score_high_demand(volumes)
print(score_hd)  # p.ej. 0.97… o 1.0 según el máximo
```

### 2) Score de estacionalidad
```python
volumes_summer = [500, 600, 800, 1200, 2000, 3000, 2500, 1800, 1000, 700, 600, 500]
score_se = ScoreDemandSeasonality.score_seasonality(volumes_summer)
print(score_se)  # ~0.8–0.9 para un patrón muy estacional
```

### 3) Score final multikeyword
```python
from scripts.scores.score_demand_seasonality.models import ModelParamsScoreDemandSeasonality
from scripts.scores.score_demand_seasonality.score_demand_seasonality import ScoreDemandSeasonality

params = ModelParamsScoreDemandSeasonality(
    volumes=[
        [900, 950, 1000, 1100, 1200, 1400, 1300, 1500, 1600, 1550, 1700, 1900],
        [300, 280, 260, 250, 240, 230, 220, 210, 220, 230, 240, 235],
    ],
    weight_high_demand=0.8,
    weight_seasonality=0.2,
)
response = ScoreDemandSeasonality.score_final(params)
print(f"Score final: {response.score_final}")
print(f"Score alta demanda: {response.s_high_demand}")
print(f"Score estacionalidad: {response.s_seasonality}")
```

## Resultados reales (ejecución con Google Ads Keyword Planner)

- Ideas generadas: 112
- Competencia reportada por Google Ads: HIGH en todos los casos listados

Tabla de resultados por keyword (12 meses, más reciente al final):

| keyword | competition | volumes (12m) | score_high_demand | score_seasonality |
|---|---|---|---:|---:|
| mantas refrescantes para perros | HIGH | [110, 40, 40, 30, 40, 40, 50, 210, 1300, 5400, 3600, 1600] | 0.2963 | 1.0 |
| camas refrescantes para perros | HIGH | [20, 20, 20, 20, 10, 30, 30, 70, 880, 2900, 1000, 480] | 0.1655 | 1.0 |
| manta refrescante | HIGH | [90, 30, 40, 20, 20, 20, 40, 110, 390, 1900, 2900, 1000] | 0.3448 | 1.0 |
| manta refrescante para gatos | HIGH | [20, 20, 30, 20, 10, 10, 10, 20, 210, 1300, 880, 480] | 0.3692 | 1.0 |
| alfombra refrescante para perros | HIGH | [40, 30, 20, 20, 10, 30, 30, 140, 720, 4400, 1900, 720] | 0.1636 | 1.0 |
| alfombra refrescante perro | HIGH | [10, 10, 10, 10, 10, 10, 10, 30, 140, 390, 210, 70] | 0.1795 | 1.0 |
| esterilla refrigerante para perros | HIGH | [10, 10, 10, 10, 10, 10, 10, 20, 140, 480, 170, 50] | 0.1042 | 1.0 |
| manta refrescante perro | HIGH | [20, 20, 10, 20, 10, 20, 20, 90, 590, 1300, 880, 320] | 0.2462 | 1.0 |
| alfombra refrigerante perros | HIGH | [10, 10, 10, 10, 10, 10, 10, 10, 30, 140, 170, 70] | 0.4118 | 1.0 |
| esterilla refrescante para perros | HIGH | [10, 10, 10, 10, 10, 10, 10, 30, 210, 880, 390, 170] | 0.1932 | 1.0 |

Score final combinado:

- score_final: 0.39794099784972

> Nota: En estos ejemplos, score_seasonality=1.0 indica patrones estacionales muy marcados (picos fuertes de verano). El score_high_demand varía según el último mes vs. máximo histórico.

## Dependencias
- `numpy` (cálculos numéricos)
- `pydantic` (modelo de entrada tipado)
- Opcional: `google-ads` si se usa el ejemplo real de Keyword Planner en `__main__`.

Revisar `requirements.txt` del proyecto para versiones concretas.

## Buenas prácticas y notas
- Preferir 12 meses de datos ordenados cronológicamente (más reciente al final).
- El score de alta demanda depende del último valor; asegúrate de que corresponde al mes más reciente.
- Maneja `NaN`/valores no numéricos antes de invocar el cálculo si tus datos son ruidosos.
- Si una keyword tiene <12 meses, `score_final` la omitirá silenciosamente (log INFO) y continuará con el resto.
- Los logs se emiten bajo el logger `ScoreDemandSeasonality` con nivel INFO.

## Limitaciones
- Series muy ruidosas sin patrón repetitivo pueden inflar el CV y aparentar estacionalidad.
- Con <6 meses, no se calcula estacionalidad (retorna 0.0).
- El bonus por `seasonal_ratio > 3` es heurístico; ajusta si tu dominio lo requiere.

## Ubicación del código
- Implementación principal: `scripts/scores/score_demand_seasonality/score_demand_seasonality.py`
- Modelo Pydantic: `scripts/scores/score_demand_seasonality/models.py`

## Changelog (breve)
- v1: Implementación de `high_demand`, `seasonality` y `score_final` + README inicial.

## Tabla de referencia

| Versiones | Script de puntuacion                             | QUE MIDE?                                                                                                                                 | ENRIQUECIMIENTO                                                                                                                                                                                                                   | DESCRIPCION |
|----------:|--------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| v1.0      | Estacionalidad de Demanda (score_seasonality)    | Estacionalidad del interés de mercado (0–1) a partir de búsquedas mensuales.<br>Bandas sugeridas:<br>- 0.80–1.00: Muy estacional<br>- 0.50–0.79: Estacional moderado<br>- 0.20–0.49: Estacional bajo<br>- 0.00–0.19: Demanda plana | Fuente: Google Ads Keyword Planner (series mensuales por URL/keyword).<br>Requisitos: ≥6 meses (ideal 12).<br>Frecuencia sugerida: mensual (sincronizada con actualización de datos).<br>Cobertura: categorías con volumen suficiente.<br>Limitaciones: picos atípicos, productos nuevos, cambios de naming o catalogación. | Cómo lo calcula (resumen):<br>- Detecta meses “altos” vs. “bajos” con terciles y mide la diferencia entre ellos (intensidad).<br>- Considera la variabilidad general de la serie (estabilidad vs. volatilidad).<br>- Combina ambas señales en un único score; añade un pequeño bonus si la diferencia alto/bajo es muy marcada.<br><br>Cómo interpretarlo (acciones):<br>- Muy estacional: subir inversión (SEM/PLA), stock/fulfillment, landings y creatividades de temporada; bundles; email/calendar marketing.<br>- Moderado: contenidos y pujas en picos; preparar logística; SEO estacional; optimizar fichas.<br>- Bajo/plano: campañas always-on eficientes; liquidación/cross-sell en bajas; enfoque long-tail; control estricto de CPA/ROAS.<br><br>Ejemplos típicos:<br>- Ropa de baño: ~0.85–0.95 (verano). Árbol de Navidad: ~0.90 (nov–dic). Abrigos: ~0.70–0.80 (otoño–invierno). Pilas: ~0.10–0.20 (plano).<br><br>Señales complementarias para decisión:<br>- Volumen de búsqueda total, precio competitivo, top selling, disponibilidad de stock, margen, ROAS/CPA, estacionalidad calendario retail (rebajas, BF).<br> |
