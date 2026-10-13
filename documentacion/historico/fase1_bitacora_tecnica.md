# Bitácora científica y registro de experimentos — AgroCebada 2026

## 0. Origen

- Bases: **Reto AgroCebada FIRA 2026**, 8 páginas, adjuntas por el equipo.
- Descripción de variables: **Descripción general del conjunto de datos**, 3 páginas; define 138 rendimientos conocidos, 59 ocultos, ciclo abril-octubre 2025, sensores y fuentes.
- Diccionario de índices satelitales: documento de FIRA, 3 páginas. Señala explícitamente que índices de sensores distintos no deben considerarse idénticos.
- Fecha de esta batería: 2026-10-11/12 UTC, aproximadamente (véase `configuracion_experimento.json`).
- Desarrollo: scripts Python reproducibles, revisados con los datos del reto.

## 1. Objetivo y unidad estadística

Variable objetivo: `RENDIMIENTO_T_HA` (toneladas por hectárea). Unidad observacional e independiente a efectos del concurso: parcela, identificada por `ID_POLIGONO`.

- Total: 197 parcelas únicas.
- Entrenamiento: 138 parcelas con rendimiento observado.
- Predicción: 59 parcelas sin rendimiento mostrado.
- Restricción: no usar el rendimiento oculto para nada; NO dividir observaciones satelitales de la misma parcela entre folds independientes.

## 2. Extracción de características

1. Comprobar unicidad de IDs y correspondencia con polígonos.
2. Reproyectar geometrías EPSG:4326 a EPSG:6372 para centroides métricos, y volver a EPSG:4326 para extraer series raster.
3. Comprobar área oficial CSV vs área Shapefile con diferencia inferior a 0.011 hectáreas (ambas originalmente redondeadas).
4. Para los CSV satelitales, separar **Sentinel-2**, **Landsat** y **Planet**, restringir al ciclo abril-octubre 2025; probar nubosidad máxima de 0, 10, 20 y 40%.
5. Por cada parcela/sensor/mes, mediana de índices seleccionados, sin fusionar observaciones de sensores distintos. También incluir estadísticas del ciclo y número de observaciones válidas.
6. Extraer precipitación mensual CHIRPS, temperaturas mín. y máx. CHIRTS-ERA5 de los 7 meses del ciclo; generar agregados de 2025 y promedios históricos de 2022–2024 por parcela. Los datos climáticos de 2025 se utilizan en el experimento **retrospectivo**, no para predicción ex ante.
7. Extraer elevación y pendiente INEGI CEM de archivos `120m` del paquete suministrado.
8. Preservar estado/municipio/código de parcela para informes, pero **no incluir identificadores de parcela** en características del estimador.
9. Imputación mediana y codificación de categorías ajustadas **únicamente sobre entrenamiento** en cada pliegue mediante `Pipeline`.

### Limitaciones técnicas

- Las medianas por mes reducen ruido, pero pierden variación intra-mensual; el estudio final debe comparar otra extracción fenológica.
- El preprocesamiento de series satelitales no está supervisado: no aprende parámetros de rendimiento de las 138 parcelas. Las reglas de selección basadas en error predictivo se comparan en CV y requieren evaluación anidada para evitar optimismo de selección.
- Temperatura/precipitación en 0.05 grados (~5km): no representa microclima individual de cada parcela.
- Las geometrías pueden intersectar varios píxeles; de momento muestrear centroide, no promedio zonal.
- Los polígonos que están fuera del ráster o valores `NoData` se representan como nulos.

## 3. Estrategias comparadas

Se evaluaron árboles ExtraTrees con 65 árboles, mínimo 2 observaciones por hoja, 85% máximo de variables disponibles por división, semilla 42, con los conjuntos de características:

- Solo geografía: área, latitud, longitud, estado y municipio.
- Satélite + área.
- Geografía + clima y topografía.
- Completo = geografía, clima, topografía, satélites.

Tres arquitecturas con la configuración completa y nubosidad ≤20%:

1. `global`: un regresor ExtraTrees para todas las regiones.
2. `estado`: tres regresores especializados, con retorno al modelo global si un estado tiene menos de 12 observaciones conocidas en ese fold.
3. `ajuste`: modelo global con corrección de residual por estado, contraída por n/(n+25), y ajustada solo con las observaciones del fold de entrenamiento.

Aclaración: son **arquitecturas de referencia**, no modelos finales optimizados. La corrección residual emplea residuos **en entrenamiento** y puede contener sesgo; se usa solo como diagnóstico.

## 4. Validación

- Protocolo A `parcelas`: `StratifiedKFold(n_splits=5,shuffle=True,random_state=42)` estratificado por estado, parcela única por fold.
- Protocolo B `municipios`: `GroupKFold(n_splits=5)` con clave estado + municipio, sin compartir municipios entre entrenamiento y validación.
- El rendimiento `y` nunca es variable explicativa.
- Para cada alternativa, las predicciones OOF se confrontan con los rendimientos reales, y se calculan RMSE, MAE y R² sobre las 138 observaciones OOF.
- Comparaciones de nubosidad únicamente en protocolo A, no en protocolo B (limitación deliberada de esta fase).
- Selección provisional: menor RMSE del protocolo A entre modelos ExtraTrees; **el RMSE así seleccionado no es una estimación independiente de desempeño final**, por pruebas múltiples y elección de configuración.
- FIRA no ha proporcionado rendimientos de las 59 parcelas; las predicciones para ellas no tienen error observable.

## 5. Hallazgos principales

Consultar `comparacion_modelos_cv.csv` para cifras completas. En la ejecución inicial:

- Solo geografía global, CV por parcelas: RMSE ≈0.496 t/ha.
- Global completo, nubosidad máxima 10%, CV por parcelas: RMSE ≈0.509 t/ha.
- Global completo, nubosidad máxima 20%, CV por parcelas: RMSE ≈0.516 t/ha.
- Por estado, nubosidad máxima 20%, CV por parcelas: RMSE ≈0.543 t/ha.
- Global completo, CV por municipios: RMSE ≈0.852 t/ha, R² ≈ −0.009.

Estos resultados reflejan la dificultad de generalizar a municipios distintos y NO demuestran que los índices sean inútiles. Pueden existir efectos fenológicos, colinealidad, ruido, diferencias de escala e hiperparámetros inadecuados.

## 6. Próximos experimentos, en orden

1. Auditar rendimientos y la dependencia espacial, incertidumbre entre folds y densidad por municipio.
2. Comparar validación repetida por parcelas y validación leave-one-municipio-out, informando el sesgo de distribución de grupos.
3. Aprender/seleccionar características fenológicas (subperiodos, tendencias, pico de vigor y agua) con experimentos controlados, evitando usar el dataset oculto para decidir.
4. Comparar CatBoost, ElasticNet/Ridge, ExtraTrees, Gradient Boosting, regularización y modelos jerárquicos parciales; repetir con más semillas.
5. Evaluar incorporación de fuentes externas únicamente si mejoran error de validación y tienen cobertura homogénea de las 197 parcelas.
6. Congelar modelo, generar 59 predicciones finales con 1 fila por parcela y generar `predicciones_FIRA.csv` y documentación de inferencia.
7. Aplicación dashboard geográfico/financiero, CSV descargable, despliegue y prueba por enlace desde otra máquina.
8. Reporte técnico ajustado a extensiones oficiales, demo ≤5 minutos, bibliografía y anexos ≤2 cuartillas; auditoría de requisitos.

