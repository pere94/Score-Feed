# Guía Funcional de Scores para Google Shopping (Feed Decisions)

Esta guía está orientada 100% a la activación en Google Shopping / Performance Max a partir de señales calculadas sobre el feed. Objetivo: usar los scores para segmentar, etiquetar (custom_label), priorizar presupuesto, ajustar pujas / objetivos y excluir productos poco eficientes.

Todos los scores ∈ [0.0, 1.0]. Más alto = más atractivo para la palanca que mide.

---
## 1. Resumen Estratégico
| Score | Qué indica en Shopping | Palanca Ads principal | Riesgo si se ignora |
|-------|------------------------|-----------------------|---------------------|
| PVP Price | Potencial de margen + ticket | Asignación presupuesto / CPA objetivo | Invertir en productos de bajo valor marginal |
| Price Competitiveness | Precio vs benchmark mercado | Bidding (ROAS / prioridad) | Gastar en productos caros no competitivos |
| Top Selling Product | Fuerza comercial probada | Campañas premium / mantener visibilidad | Perder share en best sellers |
| Demand Seasonality | Momento estacional óptimo | Escalado temporal de presupuesto | Llegar tarde al pico de demanda |
| Demand Stability | Previsibilidad de demanda | Campañas always-on / baseline inversión | Sobre-reaccionar a volatilidad |
| Demand Trend | Momentum / aceleración | Tests de escalado / push emergentes | Perder primeras ventanas de crecimiento |
| Search Volumes | Tamaño potencial de mercado | Jerarquía de campañas / clustering | Dispersar presupuesto en nichos mínimos |

---
## 2. Mapa Score → Acción de Feed / Campaña
| Score (umbral) | Acción recomendada en Shopping |
|----------------|--------------------------------|
| PVP Price ≥ 0.75 | Enviar a Campaña High Margin (prioridad alta / tROAS más laxo para escalar volumen) |
| PVP Price < 0.30 | Etiquetar para exclusión o menor prioridad (custom_label: low_margin) |
| Price Competitiveness ≥ 0.90 | Subir puja (o reducir tROAS objetivo) / incluir en campañas agresivas |
| 0.70 ≤ Price Competitiveness < 0.90 | Mantener en estándar, monitorizar benchmarks |
| Price Competitiveness < 0.50 | Pausar / bajar prioridad / revisar pricing antes de invertir |
| Top Selling Product ≥ 0.80 | Aislar en campaña Best Sellers + presupuesto dedicado |
| Top Selling Product < 0.40 | Mantener sólo si otras señales fuertes (Trend alta, PVP alta) |
| Demand Seasonality ≥ 0.70 | Incrementar presupuesto 2–4 semanas pre-pico (preboost) |
| Demand Stability ≥ 0.70 | Base catalog (always-on) para construir histórico de conversión |
| Demand Trend ≥ 0.65 | Crear ad group / listing group específico “Rising” para test de escalado |
| Search Volumes ≥ 0.70 | Cluster “Core Market” (campañas separadas) |
| Search Volumes < 0.30 & Trend ≥ 0.60 | Nicho emergente: incluir en experimento PMax emerge |

---
## 3. Taxonomía de Custom Labels (Sugerida)
| custom_label_x | Posibles valores | Fuente (score / regla) | Uso principal |
|----------------|------------------|------------------------|---------------|
| custom_label_0 | margin_high / margin_mid / margin_low | PVP Price (≥0.75 / 0.45–0.74 / <0.45) | Estructura campañas por margen |
| custom_label_1 | price_comp_high / mid / low | Price Competitiveness | Ajuste de prioridad / puja |
| custom_label_2 | bestseller / core / tail | Top Selling Product | Campañas dedicadas best sellers |
| custom_label_3 | seasonal_peak / seasonal_warm / offseason | Demand Seasonality | Time-based budget shifts |
| custom_label_4 | rising / stable / declining | Demand Trend + Stability | Exploración vs protección |

(Adaptar a límites de negocio y disponibilidad de labels libres.)

---
## 4. Segmentaciones Tipo (Filtros)
Ejemplos de reglas (pseudo-SQL / tabla intermedia de feed):
```sql
-- Best Sellers competitivos para empuje agresivo
SELECT * FROM products
WHERE top_selling_score >= 0.80
  AND price_competitiveness_score >= 0.90
  AND demand_stability_score >= 0.60;

-- Productos a revisar (posible pausa)
SELECT * FROM products
WHERE price_competitiveness_score < 0.50
  AND pvp_price_score < 0.40
  AND demand_trend_score < 0.40;

-- Nichos emergentes (experimental)
SELECT * FROM products
WHERE demand_trend_score >= 0.65
  AND search_volumes_score BETWEEN 0.30 AND 0.60
  AND price_competitiveness_score >= 0.70;

-- Pre-pico estacional (activar push)
SELECT * FROM products
WHERE demand_seasonality_score BETWEEN 0.60 AND 0.75
  AND demand_trend_score >= 0.50;
```

---
## 5. Playbooks Operativos
| Situación | Detección (scores) | Acción táctica | Revisión |
|-----------|--------------------|---------------|----------|
| Empujar best sellers | Top Selling ≥0.8 & Price Comp ≥0.9 | Campaña dedicada / Lower tROAS | Semanal |
| Mejorar rentabilidad | PVP ≥0.75 & Price Comp 0.7–0.85 | Ajustar feeds: imágenes + títulos + test puja | Quincenal |
| Corregir pricing | Price Comp <0.5 & Trend ≥0.5 | Analizar margen real, reducir precio si viable | Ad-hoc |
| Captar pico estacional | Seasonality ≥0.7 & Trend ≥0.5 | Subir presupuesto + creatividades alineadas | Ventana pico |
| Apostar emergentes | Trend ≥0.65 & Stability ≥0.5 | Crear experimento PMax / Listing group “Rising” | Cada 2 semanas |
| Contener gasto ineficiente | PVP <0.4 & Trend <0.4 | Bajar prioridad o excluir | Semanal |

---
## 6. Umbrales Recomendados (Iniciales)
| Score | Alto | Medio | Bajo | Nota calibración |
|-------|------|-------|------|------------------|
| PVP Price | ≥0.75 | 0.45–0.74 | <0.45 | Percentiles 70/40 comisión + ticket |
| Price Competitiveness | ≥0.90 | 0.70–0.89 | <0.70 | Revisar vs benchmark categoría |
| Top Selling | ≥0.80 | 0.50–0.79 | <0.50 | Depende de profundidad ranking disponible |
| Seasonality | ≥0.70 | 0.40–0.69 | <0.40 | Ajustar por categoría (moda vs evergreen) |
| Stability | ≥0.70 | 0.50–0.69 | <0.50 | CV histórico (ajustar tras 2 ciclos) |
| Trend | ≥0.65 | 0.45–0.64 | <0.45 | Subir si demasiados marcados “rising” |
| Search Volumes | ≥0.70 | 0.40–0.69 | <0.40 | Basado en percentiles globales |

---
## 7. Flujo Sugerido Semanal
1. Actualizar dataset de scores (batch).  
2. Refrescar dashboards: distribución y % productos por banda.  
3. Lista acciones:
   - Añadir/retirar productos de campañas best sellers.
   - Reasignar presupuesto (Seasonality & Trend).  
   - Revisar pricing de cola cara (Price Comp bajo + PVP bajo valor).  
4. Generar reporte corto: cambios de estado (subidas/bajadas de banda).  
5. Ajustar umbrales si >50% del catálogo cae en “alto” (perderíamos discriminación).

---
## 8. Ejemplos de Decisiones Concretas
| Objetivo | Regla simple | Resultado esperado |
|----------|--------------|--------------------|
| Incrementar ROAS | Incluir sólo Price Competitiveness ≥0.9 en Campaña Prime | Mejor ratio conversión / coste |
| Escalar volumen rentable | Añadir PVP ≥0.75 incluso con Competitiveness 0.75–0.89 | Mayor ticket medio compensa ligero sobreprecio |
| Preparar temporada | Si Seasonality 0.60–0.69 esta semana → duplicar presupuesto en 14 días | Captar early seekers |
| Defender cuota | Top Selling ≥0.8 & Trend ≥0.5 → mantener impres share > X% | Evitar pérdida de ranking pagado |
| Limitar desgaste | Excluir Price Comp <0.5 & Trend <0.4 | Liberar presupuesto ineficiente |

---
## 9. Integración con Performance Max
- Agrupar productos con labels homogéneos para señales claras al sistema.
- Evitar mezclar extremos (baja competitividad + alto margen) en el mismo asset group al inicio (ruido para el modelo). 
- Después de 4–6 semanas, evaluar si PMax aprende a compensar y relajar segmentación.

---
## 10. Checklist de Calidad del Feed (Relacionada a Scores)
| Área | Pregunta | Acción |
|------|----------|-------|
| Datos de precio | ¿Hay % con market_price faltante? | Completar fuentes / fallback categoría |
| Margen proxy | ¿Distribución PVP Price aplastada (muchos =1)? | Ajustar min/max comisión |
| Seasonality | ¿Categorías evergreen con Seasonality alto? | Revisar outliers / picos únicos |
| Trend | ¿Demasiados rising >30% de catálogo? | Subir umbral Trend alta |
| Competitividad | ¿Productos caros en campañas premium? | Revaluar segmentación |

---
## 11. FAQ Operativa
**¿Por qué un producto competitivo y con margen no vende?** Posible baja demanda actual: revisar Trend / Seasonality.  
**¿Cuándo pausar por precio?** Price Competitiveness <0.5 y sin soporte de Trend alto ni PVP alto.  
**¿Cómo priorizar entre Seasonality y Trend?** Si Seasonality alta pero Trend baja, es pico tardío; si Trend alta y Seasonality baja, es categoría emergente aún fuera de temporada.

---
## 12. Roadmap de Mejora (Marketing)
| Mejora | Beneficio |
|--------|-----------|
| Añadir margen real (coste) | Decisiones de puja más precisas |
| Integrar benchmark click share | Mejor priorización de defensa |
| Score de calidad de listing (título/imágenes) | Identificar quick wins CRO feed |
| Cohorte de nuevos productos | Estrategia específica de incubación |

---
## 13. Notas de Gobernanza
- Versionar cambios de umbrales (registro mensual).  
- Mantener consistencia de custom_label para evitar “saltos” en histórico.  
- Documentar exclusiones masivas (motivo, fecha, % catálogo afectado).  

---
## 14. Contacto / Ownership
Owner: Growth & Data  |  Canal: #data-marketing  |  Última actualización: 2025-08-13

---
## 15. Apéndice (Interpretación Rápida)
| Situación | Mirar primero | Acción relámpago |
|-----------|---------------|------------------|
| Bajas conversiones en campaña core | Price Competitiveness | Filtrar y excluir <0.5 |
| Margen deteriorado | PVP Price | Reenfocar presupuesto a ≥0.75 |
| Necesidad de volumen | Search Volumes + Trend | Activar rising en cluster volumen medio |
| Temporada próxima | Seasonality (0.60–0.70) | Precalentar campañas |
| Alta volatilidad | Stability | Trasladar inversión a segmentos ≥0.7 |