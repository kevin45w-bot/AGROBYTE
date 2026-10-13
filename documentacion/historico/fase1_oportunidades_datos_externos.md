# Datos externos candidatos — AgroCebada FIRA 2026

Este documento presenta **candidatos de investigación**, no variables ya integradas o validadas. Para incorporar una fuente: registrar URL, organismo, licencia, fechas, unidades, resolución, cobertura para las 197 parcelas, método de emparejamiento espacial y ganancia incremental en CV sin fuga de información.

## 1. Alta prioridad: SoilGrids (ISRIC)

**Fuente oficial:** https://isric.org/explore/soilgrids/ y documentación https://docs.isric.org/globaldata/soilgrids/

- Propiedades a investigar: pH, arcilla/arena/limo, carbono orgánico, nitrógeno, capacidad de intercambio catiónico; capas a 250 m y diversas profundidades.
- Hipótesis: el suelo explica fertilidad, almacenamiento de agua y parte de la variabilidad entre parcelas, añadiendo información que satélite y clima no captan por completo.
- **Riesgo:** estimaciones de propiedades del suelo son modelos, no muestreos de laboratorio de las parcelas; resolución espacial y correlación con ubicación. ¡No asumir que mejorarán rendimiento!
- **Acceso:** ISRIC indica que su API REST tiene el servicio temporalmente pausado/inestable. Priorizar capas ráster descargadas, WCS o Google Earth Engine si están disponibles; no basar el despliegue en la API REST.

## 2. Alta prioridad: producción agrícola municipal histórica — DGSIAP/SIAP

**Fuente oficial:** https://nube.agricultura.gob.mx/datosAbiertos/Agricola.php

- Buscar cebada grano y rendimientos municipales **2022, 2023 y 2024** (años previos al ciclo de rendimiento objetivo de 2025), preferiblemente diferenciando modalidad de temporal y riego si se dispone.
- Hipótesis: el contexto productivo histórico puede captar rasgos de manejo, infraestructura o aptitud agroclimática no medidos en las variables satelitales.
- **Riesgo de fuga:** NO agregar rendimiento municipal 2025 como predictor sin investigar si incorpora rendimientos de las mismas parcelas evaluadas. Preferir históricos anteriores a 2025. Normalizar entidad/municipio y cultivo; revisar que haya datos históricos en todos los municipios. Cualquier asociación con el rendimiento individual debe validarse fuera de muestra.

## 3. Prioridad secundaria: TerraClimate — balance hídrico

**Fuente:** https://developers.google.com/earth-engine/datasets/catalog/IDAHO_EPSCOR_TERRACLIMATE

- Variables candidatas: evapotranspiración potencial (`pet`), déficit climático (`def`), humedad de suelo modelada (`soil`), VPD (`vpd`) y radiación solar (`srad`).
- **Importante:** aproximadamente 4.6 km de resolución; puede ser redundante con CHIRPS/CHIRTS y no representa variabilidad de cada parcela. Confirmar disponibilidad de meses del 2025 antes de programar una integración; el catálogo consultado no lo confirmó inequívocamente.

## 4. Datos que NO buscaría prioritariamente

- Más precipitación mensual para 2022–2025 (ya tenemos CHIRPS).
- Más Tmax/Tmin mensual del mismo periodo y escala (ya tenemos CHIRTS-ERA5).
- NDVI genérico de los mismos satélites sin una razón específica (ya tenemos series del reto).
- Precios de cebada como *variable predictora de rendimiento físico* (pueden servir al módulo financiero, pero no sustituyen variables agronómicas).

## Esquema sugerido para fuentes externas

`ID_POLIGONO | variable | valor | unidad | periodo | fecha_observacion | fuente | URL | resolución_espacial | método_de_extracción | advertencia_fuga`

**Criterio de aceptación:** mostrar mejoría estable en RMSE/MAE por parcelas, evaluar por municipios y justificar el mecanismo agronómico. La disponibilidad comercial, la calidad de la fuente y la temporalidad importan tanto como la correlación.
