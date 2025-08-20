# Doc - Marketing Scripts

---

## Score PVP Price
Normaliza la comisión entre mínimos y máximos de referencia y aplica un bonus si el PVP supera un umbral. Útil para priorizar productos con mejor margen potencial y mayor valor por pedido.

**Casos de uso típicos**:
- Priorizar productos con buena comisión y ticket medio alto.
- Seleccionar artículos para campañas con mayor rentabilidad potencial.
- Aplicar reglas de puja o visibilidad basadas en margen/valor.

**Modelo de parámetros para el script de PVP**:
- percent_commission (Default: 6%) — % de comisión aplicado (0–100) a las ventas del feed.
- current_price — Precio actual del producto.
- pvp_greater_than (Default 50.0) — umbral para activar bonus (30%) score.
- min_commission (Default 1.0%) — comisión mínima de referencia.
- max_commission (Default 15.0%) — comisión máxima de referencia.

---

## Score Price Competitiveness
Valorar la competitividad de precio de un producto frente al precio de mercado. Devuelve un score normalizado entre 0.0 y 1.0 (mayor es mejor).

**Casos de uso típicos**:
- Priorizar productos con precio igual o mejor que mercado en campañas de performance.
- Detectar oportunidades de ajuste de precio en productos poco competitivos.
- Alimentar reglas de puja y visibilidad basadas en precio relativo.

**Modelo de parámetros para el script de competitividad de precios**:
- current_price: (float) - Precio actual del producto.
- market_price: (float) - Precio de mercado del producto.

**Ejemplos:**
- Score para precio 100 €, mercado 20 € = 0.10
- Score para precio 100 €, mercado 50 € = 0.12
- Score para precio 100 €, mercado 80 € = 0.52
- Score para precio 100 €, mercado 90 € = 0.68
- Score para precio 100 €, mercado 93 € = 0.77
- Score para precio 100 €, mercado 95 € = 0.84
- Score para precio 100 €, mercado 97 € = 0.91
- Score para precio 100 €, mercado 100 € = 1.00
- Score para precio 100 €, mercado 110 € = 1.00
- Score para precio 100 €, mercado 120 € = 1.00

---

## Score Top Selling Products
Valorar productos Top Selling combinando posición en el ranking y señales de demanda relativa (nivel y evolución). Devuelve un score entre `min_score` y `max_score`.

**Casos de uso típicos**:
- Priorizar productos líderes para campañas de performance.
- Detectar mejoras/empeoramientos de ranking y ajustar pujas.
- Combinar tendencia de demanda con ranking para decisiones de stock y visibilidad.

**Modelo de parámetros para el cálculo de score de productos top selling**:
- is_top_selling_product: (True/False) - Indica si el producto está en nuestro inventario.
- rank_top_selling_product: (entero) - Posición del producto en el ranking de ventas.
- previous_rank: (entero) - Posición anterior del producto en el ranking de ventas.
- relative_demand: (RelativeDemand | str) - Demanda relativa actual del producto.
- previous_relative_demand: (RelativeDemand | str) - Demanda relativa anterior del producto.
- relative_demand_change: (RelativeDemandChangeType | str) - Tipo de cambio en la demanda relativa.
- max_rank: (entero | Default: 100) - Posición máxima en el ranking.
- min_score: (float | Default: 0.2) - Puntuación mínima.
- mid_score: (float | Default: 0.5) - Puntuación media.
- max_score: (float | Default: 1.0) - Puntuación máxima.

**Tabla de resultados por producto**:

| Test | score (TS=True) | score (TS=False) | current_demand | previous_demand | change_type | current_rank | previous_rank |
|----------------:|------------------:|-----:|-----------------|-----------------|-------------|--------------:|---------------:|
| 01 | 0.78 | 0.18 | HIGH | MEDIUM | RISER | 10 | 20 |
| 02 | 0.46 | 0.10 | LOW | HIGH | SINKER | 50 | 30 |
| 03 | 1.00 | 0.20 | VERY_HIGH | VERY_HIGH | FLAT | 1 | 1 |
| 04 | 0.53 | 0.13 | MEDIUM | LOW | RISER | 100 | 150 |
| 05 | 0.38 | 0.10 | VERY_LOW | VERY_HIGH | SINKER | 200 | 50 |
| 06 | 0.54 | 0.16 | HIGH | HIGH | FLAT | None | None |
| 07 | 0.75 | 0.10 | RELATIVE_DEMAND_UNSPECIFIED | HIGH | FLAT | 5 | 5 |
| 08 | 0.70 | 0.10 | LOW | LOW | FLAT | 10 | 10 |
| 09 | 0.82 | 0.14 | HIGH | LOW | SINKER | 5 | 10 |
| 10 | 0.63 | 0.10 | LOW | HIGH | RISER | 10 | 5 |
| 11 | 0.97 | 0.16 | HIGH | HIGH | SINKER | 1 | 1 |
| 12 | 0.48 | 0.10 | LOW | LOW | RISER | 1000 | 1000 |
| 13 | 0.57 | 0.20 | VERY_HIGH | RELATIVE_DEMAND_UNSPECIFIED | FLAT | None | 1 |
| 14 | 0.47 | 0.10 | RELATIVE_DEMAND_UNSPECIFIED | RELATIVE_DEMAND_UNSPECIFIED | FLAT | None | None |
| 15 | 0.91 | 0.10 | LOW | LOW | RELATIVE_DEMAND_CHANGE_TYPE_UNSPECIFIED | 1 | 1 |

---

## Score Demand Seasonality
Calcular métricas de demanda y estacionalidad a partir de series mensuales de volúmenes de búsqueda (idealmente 12 meses por keyword) y combinar ambos indicadores en un score final entre 0 y 1.

**Casos de uso típicos**:
- Detectar productos con demanda muy alta en el presente (picos recientes).
- Identificar patrones estacionales marcados (ej. verano/invierno, campañas, etc.).
- Priorizar acciones de marketing o pricing en función del momento del año.

**El módulo expone tres métodos estáticos**:
- score_high_demand(volumes) 
    * Devuelve un score entre 0 y 1 basado en la demanda actual.
    * Indica si el volumen del mes actual está cerca del máximo histórico.
- score_seasonality(volumes)
    * Devuelve un score entre 0 y 1 basado en la estacionalidad de la demanda.
    * Indica si el histórico de volúmenes muestra patrones estacionales claros (0=producto sin estacionalidad, 1=producto con estacionalidad muy marcada).
- score_demand_seasonality(volumes, weight_seasonality=0.2, weight_high_demand=0.8)
    * Pesos internos:
        - `weight_seasonality: float = 0.2`
        - `weight_high_demand: float = 0.8`
    * Combina los scores de alta demanda y estacionalidad en un score final.
    * Permite ajustar la estrategia en función de la demanda actual y las tendencias estacionales.
    * Indica de manera cuantitativa si nos encontramos ante un producto con estacionalidad activa.

**Tabla de resultados por keyword (12 meses, más reciente al final)**:

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

---

## Score Demand Stability
Medir la estabilidad de la demanda a partir de series mensuales de volúmenes de búsqueda y penalizar caídas abruptas recientes. Devuelve un score final entre 0 y 1.

**Casos de uso típicos**:
- Detectar productos con demanda sostenida y predecible.
- Identificar señales de riesgo por caídas recientes del interés.
- Priorizar inversiones en productos con demanda estable.

**El módulo expone tres métodos estáticos**:
- score_stability(volumes)
    * Devuelve un score entre 0 y 1 basado en la estabilidad de la demanda.
    * Indica si el histórico de volúmenes muestra una tendencia estable o inestable.
    * Devuelve 1 si los volúmenes son muy constantes; 0 si son muy variables.
- score_sudden_falls(volumes, window=3)
    * Devuelve un score entre 0 y 1 basado en la estabilidad de la demanda.
    * Indica si el histórico de volúmenes ha tenido caídas abruptas recientes.
    * Penaliza caídas fuertes en los últimos `window` meses.
- score_final(params)
    * Parámetros:
        - `volumes`: lista de volúmenes mensuales.
        - `weight_stability`: peso para el score de estabilidad (default 0.5).
        - `weight_sudden_falls`: peso para el score de caídas recientes (default 0.5).
    * Devuelve un score entre 0 y 1 basado en la estabilidad de la demanda.
    * Combina estabilidad y caídas recientes con pesos configurables.
    * Indica de manera cuantitativa si nos encontramos ante un producto con demanda estable.

**Tabla de resultados por keyword (12 meses, más reciente al final)**:

| keyword | competition | volumes (12m) | score_stability | score_sudden_falls |
|---|---|---|---:|---:|
| bufanda para perros | MEDIUM | [90, 140, 260, 480, 210, 70, 90, 30, 50, 20, 20, 30] | 0.0 | 1 |
| bufanda perro | HIGH | [70, 110, 210, 390, 210, 110, 70, 30, 20, 10, 50, 50] | 0.049087664458368874 | 1 |
| bufanda de navidad para perros | UNSPECIFIED | [0, 0, 20, 10, 0, 0, 0, 0, 0, 0, 0, 0] | 0.0 | 1 |
| bufanda de perro | UNSPECIFIED | [10, 10, 10, 30, 20, 10, 10, 10, 10, 0, 0, 10] | 0.2991974169889001 | 1 |
| bufanda galgo | HIGH | [10, 10, 20, 10, 20, 10, 10, 10, 10, 10, 10, 10] | 0.68056171750003 | 1 |
| bufanda navidad perro | UNSPECIFIED | [0, 0, 20, 30, 10, 10, 0, 0, 10, 0, 0, 0] | 0.0 | 1 |
| bufanda navideña para gatos | UNSPECIFIED | [0, 0, 0, 10, 0, 0, 0, 0, 0, 0, 0, 0] | 0.0 | 1 |
| bufanda navideña para perro | UNSPECIFIED | [0, 0, 10, 10, 0, 0, 0, 0, 0, 0, 0, 0] | 0.0 | 1 |
| bufanda para galgo | LOW | [10, 10, 20, 30, 10, 10, 0, 10, 10, 10, 10, 10] | 0.41098491062604836 | 1 |
| bufanda para perro pequeño | UNSPECIFIED | [10, 10, 10, 30, 10, 10, 10, 10, 0, 0, 10, 0] | 0.1717787655323364 | 0.0 |

Score final combinado:

- score_final: 0.4566966285063411

---

## Score Demand Trend
Medir la tendencia de demanda combinando tres señales complementarias sobre series mensuales de volúmenes de búsqueda, devolviendo un score final entre 0 y 1.

**Casos de uso típicos**:
- Detectar términos/productos en clara aceleración de demanda para campañas.
- Distinguir crecimientos sostenidos de picos aislados.
- Priorizar categorías con momentum positivo y tendencia sólida.

**El módulo expone tres métodos estáticos**:
- score_momentum(volumes, window=3, penalize_negative=True, normalization_factor=0.5)
    * Parámetros:
        - `volumes`: Lista de volúmenes mensuales de búsqueda, ordenados del mes más reciente hacia atrás.
        - `window`: Número de meses recientes a considerar para calcular el momentum. (default 3).
        - `penalize_negative`: Si True, penaliza fuertemente tendencias negativas devolviendo 0 si la pendiente es negativa más allá de cierto umbral.
        - `normalization_factor`: Factor para ajustar la sensibilidad de la normalización (más bajo = más sensible a cambios). (default 0.5).
    * Devuelve un score entre 0 y 1 basado en el momentum de la demanda.
    * Momentum reciente (sensibilidad a cambios de muy corto plazo).
    * Calcula un score de momentum reciente basado en la pendiente de los últimos 'n' meses, con mayor sensibilidad y penalización explícita para tendencias negativas y picos aislados.
- score_long_trend(volumes, penalize_negative=True, normalization_factor=0.5)
    * Parámetros:
        - `volumes`: Lista de volúmenes mensuales de búsqueda, ordenados del mes más reciente hacia atrás.
        - `penalize_negative`: Si True, penaliza fuertemente tendencias negativas devolviendo 0 si la pendiente es negativa más allá de cierto umbral.
        - `normalization_factor`: Factor para ajustar la sensibilidad de la normalización (más bajo = más sensible a cambios). (default 0.5).
    * Calcula un score de tendencia a largo plazo con mayor sensibilidad y penalización explícita para caídas fuertes.
    * Tendencia a largo plazo (direccionalidad sostenida).
    * Indica si la tendencia a largo plazo es positiva, negativa o estable.
- score_accelerated_growth(volumes, window=3)
    * Parámetros:
        - `volumes`: Lista de volúmenes mensuales de búsqueda, ordenados del mes más reciente hacia atrás.
        - `window`: Número de meses recientes a considerar para detectar aceleración. (default 3).
    * Crecimiento acelerado (aceleración y consistencia de subidas).
    * Evalúa si en los últimos 'window' meses hay un crecimiento sostenido y acelerado en el volumen de búsquedas, identificando oportunidades para campañas o acciones de marketing.
- score_final(params)
    * Parámetros:
        - `volumes`: Lista de volúmenes mensuales de búsqueda, ordenados del mes más reciente hacia atrás.
        - `window`: Número de meses recientes a considerar para calcular el momentum y crecimiento acelerado (default 3).
        - `weight_long_trend`: Peso para el score de tendencia a largo plazo (default 0.35).
        - `weight_momentum`: Peso para el score de momentum (default 0.4).
        - `weight_accelerated_growth`: Peso para el score de crecimiento acelerado (default 0.25).
    * Combina los scores de momentum, tendencia a largo plazo y crecimiento acelerado con pesos configurables.
    * Indica de manera cuantitativa si nos encontramos ante un producto con tendencia positiva.

**Tabla de resultados por keyword (12 meses, más reciente al final)**:

| keyword                      | competition | volumes (12m)                                         | score_long_trend | score_momentum | score_accelerated_growth |
|------------------------------|-------------|-------------------------------------------------------|------------------|---------------|-------------------------|
| cama perro                   | HIGH        | [18100, 18100, 22200, 18100, 12100, 8100, 8100, 6600, 6600, 5400, 9900, 12100] | 0.4              | 0.78          | 1.0                     |
| cama para perros             | HIGH        | [12100, 14800, 14800, 14800, 9900, 8100, 8100, 6600, 6600, 5400, 6600, 8100]   | 0.42             | 0.64          | 1.0                     |
| cama de perro                | HIGH        | [4400, 5400, 5400, 5400, 3600, 2400, 2900, 1900, 1900, 1600, 2900, 2900]       | 0.41             | 0.0           | 0.85                    |
| cama refrescante perro       | HIGH        | [40, 40, 30, 20, 10, 40, 90, 210, 1600, 4400, 2400, 880]                       | 0.8              | 0.0           | 0.0                     |
| camas para perros pequeños   | HIGH        | [1000, 1000, 1000, 1300, 880, 720, 880, 720, 880, 590, 720, 720]               | 0.46             | 0.0           | 0.82                    |
| cama para perro grande       | HIGH        | [590, 590, 590, 590, 390, 320, 390, 260, 260, 140, 260, 320]                   | 0.4              | 0.73          | 1.0                     |
| cama viscoelástica perro     | HIGH        | [880, 720, 880, 720, 720, 590, 590, 480, 590, 390, 590, 590]                   | 0.45             | 0.0           | 0.85                    |
| cama para mascotas           | HIGH        | [590, 720, 590, 720, 390, 320, 320, 260, 260, 260, 390, 390]                   | 0.42             | 0.0           | 0.85                    |
| cama mascota                 | HIGH        | [1300, 1600, 1600, 1300, 720, 390, 480, 390, 390, 320, 880, 1000]              | 0.4              | 0.89          | 1.0                     |
| cama perro verano            | HIGH        | [70, 40, 20, 30, 40, 50, 140, 390, 1000, 1600, 880, 390]                       | 0.75             | 0.0           | 0.0                     |

**Score final combinado:**

- score_final: 0.4777

---

## Score Search Volumes
Medir el potencial de oportunidad de búsqueda a partir de series mensuales (idealmente 12 meses por keyword), combinando:
- Volumen total anual (potencial absoluto, en escala logarítmica)
- Media mensual (consistencia/recurrente)

**Casos de uso típicos**:
- Priorizar categorías o clusters de keywords con alto potencial de demanda.
- Comparar términos con órdenes de magnitud distintos sin que los picos dominen.
- Dar señal base de mercado para combinar con tendencia, estacionalidad y pricing.

**El módulo expone dos métodos de scoring y un combinador final**:
- score_total_weighting(volumes, min_volume=100, max_volume=10000)
    * Parametros:
        - `volumes`: lista de volúmenes mensuales de búsqueda, ordenados del mes más reciente hacia atrás.
        - `min_volume`: Umbral mínimo del volumen total anual. Si el total es menor a este valor, el score será 0. (default 100).
        - `max_volume`: Umbral máximo del volumen total anual. Si el total es igual o superior, el score será 1. (default 10000).
    * Volumen total anual (potencial absoluto, en escala logarítmica)
    * La función evalúa cuán significativo es el volumen total de búsquedas en un periodo de 12 meses, utilizando una transformación logarítmica para mejorar la sensibilidad en rangos bajos y comprimir rangos altos.
- score_monthly_average(volumes, min_media=100, max_media=10000)
    * Parametros:
        - `volumes`: lista de volúmenes mensuales de búsqueda, ordenados del mes más reciente hacia atrás.
        - `min_media`: Umbral mínimo de la media mensual. Si la media es menor a este valor, el score será 0. (default 100).
        - `max_media`: Umbral máximo de la media mensual. Si la media es igual o superior, el score será 1. (default 10000).
    * Media mensual (consistencia/recurrente).
    * Esta métrica evalúa la consistencia del interés a lo largo del tiempo, evitando que un único pico distorsione el resultado. Es útil para identificar términos con volumen estable o recurrente, independientemente de su estacionalidad.
- score_final(params)
    * Parametros:
        - `volumes`: lista de volúmenes mensuales de búsqueda, ordenados del mes más reciente hacia atrás.
        - `weight_total`: Peso relativo del score de volumen total anual (default=0.6)
        - `weight_monthly`: Peso relativo del score de media mensual (default=0.4)
        - `min_volume`: Umbral mínimo del volumen total anual (default=100)
        - `max_volume`: Umbral máximo del volumen total anual (default=10000)
        - `min_media`: Umbral mínimo de la media mensual (default=100)
        - `max_media`: Umbral máximo de la media mensual (default=10000).

**Tabla de resultados por keyword (12 meses, más reciente al final)**:

| keyword | competition | volumes (12m) | score_total_weighting | score_monthly_average |
|---|---|---|---:|---:|
| cama perro | HIGH | [18100, 18100, 22200, 18100, 12100, 8100, 8100, 6600, 6600, 5400, 9900, 12100] | 1.0 | 1.0 |
| cama para perros | HIGH | [12100, 14800, 14800, 14800, 9900, 8100, 8100, 6600, 6600, 5400, 6600, 8100] | 1.0 | 0.9654882154882155 |
| cama de perro | HIGH | [4400, 5400, 5400, 5400, 3600, 2400, 2900, 1900, 1900, 1600, 2900, 2900] | 1.0 | 0.33249158249158245 |
| cama refrescante perro | HIGH | [40, 40, 30, 20, 10, 40, 90, 210, 1600, 4400, 2400, 880] | 0.9947249088333459 | 0.07205387205387206 |
| camas para perros pequeños | HIGH | [1000, 1000, 1000, 1300, 880, 720, 880, 720, 880, 590, 720, 720] | 1.0 | 0.07752525252525252 |
| cama para perro grande | HIGH | [590, 590, 590, 590, 390, 320, 390, 260, 260, 140, 260, 320] | 0.8360489289678584 | 0.029461279461279462 |
| cama viscoelástica perro | HIGH | [880, 720, 880, 720, 720, 590, 590, 480, 590, 390, 590, 590] | 0.944370480341446 | 0.05505050505050505 |
| cama para mascotas | HIGH | [590, 720, 590, 720, 390, 320, 320, 260, 260, 260, 390, 390] | 0.8584188616497621 | 0.03375420875420876 |
| kiwoko online | HIGH | [2900, 3600, 3600, 3600, 3600, 2900, 3600, 2900, 3600, 3600, 3600, 2900] | 1.0 | 0.32996632996632996 |
| cama mascota | HIGH | [1300, 1600, 1600, 1300, 720, 390, 480, 390, 390, 320, 880, 1000] | 1.0 | 0.07718855218855218 |

Score final combinado:

- score_final: 0.718919191919192