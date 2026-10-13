# AgroBYTE — Reto AgroCebada FIRA 2026

**Compatibilidad recomendada:** Python 3.11 o 3.12. **Estado:** experimental.

## ¿Qué hace este proyecto?

Estimamos el rendimiento de cebada en **toneladas por hectárea (t/ha)** para 59 parcelas sin rendimiento conocido, utilizando 138 parcelas etiquetadas de un total de 197. Se integran datos satelitales de Sentinel-2, Landsat y Planet, condiciones climáticas y geografía. **Los modelos y CSV actuales son experimentales**, no la entrega final oficial.

Nuestro objetivo científico es comparar algoritmos, cuantificar errores mediante validación cruzada y comprender qué tanto generalizan a municipios no vistos. Después integraremos el mejor modelo en una aplicación web. Los datos oficiales se mantienen sin modificaciones.

## 1. Estructura obligatoria

```text
AGROBYTE PROJECT/
├── DATASET_RETO_AGRO_2026/       # Datos oficiales de FIRA: los copiamos aquí sin renombrar
│   ├── Conjunto_datos_BASICO_AgroCebada2026.csv
│   ├── Conjunto_datos_PRO_AgroCebada.csv
│   ├── ID_area_rendimiento_70_30_Reto_AgroCebada.csv
│   ├── Parcelas_Reto_AGC_CONJUNTO.shp
│   ├── Parcelas_Reto_AGC_CONJUNTO.shx
│   ├── Parcelas_Reto_AGC_CONJUNTO.dbf
│   ├── Parcelas_Reto_AGC_CONJUNTO.prj
│   ├── Parcelas_Reto_AGC_CONJUNTO.cpg
│   ├── Reto_AgroCebada_CHIRPS_Precipitacion_2022_2025.zip
│   ├── Reto_AgroCebada_Temperatura_2022_2025.zip
│   └── Reto_AgroCebada_Topografia_INEGI_CEM4.zip
├── codigo/                        # Tratamiento de datos y modelos comentados
├── documentacion/                 # Guías para el equipo y experimentos
├── historial/                     # Resultados históricos de referencia (NO oficiales)
├── salidas/                       # Archivos generados por los scripts
├── tests/                         # Pruebas automáticas
├── ejecutar.py                    # Punto de entrada del programa
└── requirements.txt               # Dependencias Python
```

Los nombres pueden incluir `(2)` antes de `.csv`; los scripts buscan patrones `Conjunto_datos_BASICO*.csv`, `Conjunto_datos_PRO*.csv` e `ID_area_rendimiento*.csv`. Debe haber exactamente **una** coincidencia por patrón. Las tres carpetas de rásteres **descomprimidas** que aparecían en la captura pueden permanecer; utilizamos los ZIP originales porque así estaba programado el pipeline de referencia.

**No hace falta escribir ninguna ruta `C:\Users\...` ni configurar un usuario de Windows.** Buscamos `DATASET_RETO_AGRO_2026` automáticamente junto a `ejecutar.py`. También funcionan rutas de Linux y macOS. Si excepcionalmente se desea otro origen, se puede pasar `--datos /ruta/alternativa`.

## 2. Instalar Python (se recomienda 3.11 o 3.12)

Abrimos esta carpeta principal en **VS Code → Archivo → Abrir carpeta**. En la terminal comprobamos:

```powershell
python --version
```

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe ejecutar.py verificar
```

Usamos directamente `.venv\Scripts\python.exe` para evitar conflictos con políticas de activación de PowerShell. Si `python` no está disponible, probamos `py -3.11` para crear el entorno.

### Linux o macOS

```bash
python3 -m venv .venv
.venv/bin/python -m pip install --upgrade pip
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python ejecutar.py verificar
```

Si alguna dependencia geoespacial no puede instalarse, copiamos el mensaje completo del error y comprobamos versión de Python y arquitectura del sistema. No es correcto afirmar que todas las combinaciones de sistemas y versiones ya fueron probadas.

## 3. Comandos disponibles (desde la raíz del proyecto)

```powershell
python ejecutar.py verificar    # Comprobamos que encontramos los insumos; NO entrenamos.
python ejecutar.py fase1       # Generamos variables y comparamos modelos preliminares.
python ejecutar.py fase2       # Probamos los modelos avanzados y validación por municipios.
python ejecutar.py diagnostico # Verificamos los 59 resultados experimentales y los errores regionales (requiere fase2).
python ejecutar.py todo        # Ejecutamos fase1 y fase2, en ese orden.
```

En Windows con entorno virtual podemos sustituir `python` por `.\.venv\Scripts\python.exe`. La fase 2 puede tardar varios minutos o más según equipo; conviene ejecutar primero `verificar` y `fase1`.

### ¿Qué debemos ver en `verificar`?

```text
OK: Rendimientos y áreas
OK: Satélite Básico
OK: Satélite PRO
OK: Polígonos SHP
OK: Índice SHX
OK: Atributos DBF
OK: Sistema PRJ
OK: Precipitación
OK: Temperatura
OK: Topografía
Todos los insumos esenciales están presentes.
```

### ¿Qué cambia al ejecutar cada fase?

| Comando | Qué hace | Archivos nuevos |
|---|---|---|
| `verificar` | Comprueba que podemos ubicar los archivos | Ninguno relevante; no entrena |
| `fase1` | Resume satélites y clima por parcela y compara configuraciones | `salidas/fase1/` |
| `fase2` | Compara siete candidatos y guarda modelo experimental | `salidas/fase2/` |
| `diagnostico` | Verifica la integridad de resultados y estudia errores | `salidas/fase2/resultados/` |

Los CSV con `NO_FINALES` o `NO_ENTREGAR` no se presentan como predicciones definitivas. Para revisar el historial de modelos sin entrenar, se pueden consultar los CSV dentro de `historial/`.

## 4. ¿Cómo comprender el proyecto si no conocemos ML?

**Lectura recomendada, en orden:**

1. [`documentacion/METRICAS_Y_VALIDACION_EXPLICADAS.md`](documentacion/METRICAS_Y_VALIDACION_EXPLICADAS.md): un ejemplo con cuatro parcelas para entender MAE, RMSE y R².
2. [`documentacion/MODELOS_EXPLICADOS_Y_USO_EN_AGROBYTE.md`](documentacion/MODELOS_EXPLICADOS_Y_USO_EN_AGROBYTE.md): modelos en general y, por separado, cómo los empleamos en Fase 1 y 2.
3. [`documentacion/TRABAJO_EN_EQUIPO_Y_ARQUITECTURA.md`](documentacion/TRABAJO_EN_EQUIPO_Y_ARQUITECTURA.md): módulos propuestos para interfaz, mapa y pruebas.
4. [`documentacion/bitacora_cientifica.md`](documentacion/bitacora_cientifica.md): resultados y decisiones científicas disponibles.
5. [`documentacion/entregables_FIRA_2026.md`](documentacion/entregables_FIRA_2026.md): matriz de pendientes de la convocatoria.

## 5. ¿Qué sabemos hoy?

El conjunto contiene 197 parcelas, de las cuales 138 tienen rendimiento conocido y 59 no. El rendimiento objetivo es del ciclo abril–octubre de 2025. En pruebas exploratorias, un modelo Extra Trees con geografía produjo RMSE ≈ 0.496 t/ha y MAE ≈ 0.356 t/ha en validación por parcelas. Una selección anidada posterior obtuvo RMSE ≈ 0.503 t/ha y MAE ≈ 0.351 t/ha. **Todavía no hemos medido el error real sobre las 59 parcelas ocultas.**

Un resultado esencial: algunos modelos empeoran considerablemente cuando deben predecir municipios que nunca aparecieron en el entrenamiento. Por eso contrastamos validación por parcelas y por municipios.

## 6. Colaboración y seguridad de archivos

La carpeta `DATASET_RETO_AGRO_2026/` se mantiene intacta. Los scripts leen, pero nunca exportan ahí. **Por defecto, `.gitignore` evita que el dataset oficial, modelos grandes y salidas se suban accidentalmente a GitHub.** Cada integrante debe recibir y colocar el dataset por una vía autorizada antes de ejecutar los experimentos.

La inclusión de originales en un repositorio, incluso privado, debe depender de que FIRA autorice su redistribución. Si el dataset ya fue versionado antes de añadir `.gitignore`, deberá retirarse del índice de Git sin borrar el directorio local; el procedimiento se explica en `documentacion/TRABAJO_EN_EQUIPO_Y_ARQUITECTURA.md`.

## 7. Limitaciones y entregables pendientes

Faltan optimización final, revisión de nuevas fuentes de información, selección del modelo final, 59 predicciones definitivas, dashboard, pruebas de despliegue, video y reporte técnico de FIRA. **Fecha límite: 19 de octubre de 2026, 23:59 h.**

## 8. Pruebas automáticas del equipo

```powershell
python -m unittest discover -s tests -v
```

Estas pruebas comprueban que las rutas sean relativas al repositorio y que nunca se escriba dentro del directorio de datos oficiales. Para comprobar la ejecución científica real también realizamos `python ejecutar.py verificar`, `fase1`, `fase2` y `diagnostico`.
