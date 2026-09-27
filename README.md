# Detección de cambios en Brumadinho (Minas Gerais, Brasil) con Sentinel-2

Midterm Project: *Satellite-Based Change Detection Implementation*
Pontificia Universidad Javeriana

## Descripción

Este proyecto aplica técnicas de detección de cambios con imágenes satelitales al caso de Brumadinho (Minas Gerais, Brasil) en enero de 2019. En ese evento un flujo de relaves mineros cubrió el valle del Ribeirão Ferro-Carvão y alcanzó el río Paraopeba.

Se comparan imágenes Sentinel-2 L2A de antes (t1) y después (t2) mediante dos índices espectrales:

- **NDVI**, sensible a la vegetación: `(B08 − B04) / (B08 + B04)`
- **NDWI** (McFeeters, 1996), sensible al agua: `(B03 − B08) / (B03 + B08)`

Para cada índice se calcula su diferencia entre fechas:

```
ΔÍndice = Índice_t2 − Índice_t1
```

Todo el procesamiento se realizó en **QGIS**, y el reporte está escrito en **LaTeX** siguiendo la plantilla del curso.

## Datos

| Parámetro | Valor |
|---|---|
| Sensor | Sentinel-2 MSI, nivel 2A (reflectancia BOA) |
| Fuente | Copernicus Browser (descarga *Analytical*) |
| t1 (antes) | 17 de enero de 2019 |
| t2 (después) | 1 de febrero de 2019 |
| Bandas | B03 (verde), B04 (rojo), B08 (NIR), SCL (clasificación de escena) |
| Formato | GeoTIFF, UInt16 (reflectancia × 10 000) |
| SRC de descarga | WGS 84 (EPSG:4326), ≈ 9,4 m × 9,3 m por píxel |
| Tamaño | 1416 × 598 píxeles |
| Área de descarga | 74,61 km² — lon −44,224111 a −44,096943; lat −20,159193 a −20,108763 |
| SRC del proyecto / ROI | WGS 84 / UTM zona 23S (EPSG:32723) |

> Los rásteres no se incluyen en el repositorio por su tamaño. Se pueden volver a descargar desde [Copernicus Browser](https://browser.dataspace.copernicus.eu/) con los parámetros de esta tabla.

## Estructura del repositorio

```
.
├── README.md
├── report/
│   ├── main.tex            # Reporte en LaTeX
│   ├── references.bib      # Bibliografía (biblatex / biber)
│   └── figures/            # Capturas de QGIS y mapas temáticos
├── data/
│   └── ROI_Brumadinho.gpkg # Polígono del área de estudio
└── qgis/
    └── proyecto.qgz        # Proyecto de QGIS (opcional)
```

## Metodología

El flujo sigue las cinco fases propuestas en el enunciado del proyecto. Esta entrega parcial cubre las fases 1 a 3.

### Fase 1 — Definición del problema y adquisición de datos
- Selección de t1 y t2: las escenas más cercanas al evento con el ROI libre de nubes, usando un filtro de nubosidad máxima del 30 % en Copernicus Browser.
- Descarga de B03, B04, B08 y SCL en formato *Raw*.
- Creación del GeoPackage `ROI_Brumadinho` en QGIS. El polígono cubre el sector de la mina, el corredor del flujo y el tramo del río Paraopeba.
- Simplificación del polígono con Douglas-Peucker, tolerancia de 15 m.

### Fase 2 — Preprocesamiento
- **Corrección atmosférica:** producto L2A (Sen2Cor), así que no se requirió corrección adicional.
- **Co-registro:** misma extensión, dimensiones, SRC y tipo de dato en t1 y t2. Se verificó visualmente con composiciones en falso color (R=B08, G=B04, B=B03).
- **Remuestreo:** no fue necesario, porque se usa un solo sensor y la misma malla.
- **Recorte:** *Ráster → Extracción → Cortar ráster por capa de máscara* usando el ROI.
- **Máscara de nubes:** a partir de la banda SCL, con esta expresión en la calculadora ráster:
  ```
  ("SCL@1" = 0) OR ("SCL@1" = 1) OR ("SCL@1" = 3) OR
  ("SCL@1" = 8) OR ("SCL@1" = 9) OR ("SCL@1" = 10)
  ```
  Un valor de 1 indica un píxel inválido: sin datos, saturado, sombra, nube o cirro.

### Fase 3 — Procesamiento y análisis
- Cálculo de NDVI y NDWI para t1 y t2 con la calculadora ráster.
- Diferencias ΔNDVI y ΔNDWI (t2 − t1).
- Interpretación y comparación de ambos índices por zona: corredor, río, mina y áreas estables.

### Fase 4 — Posprocesamiento y evaluación *(entrega final)*
- Umbralización de ΔNDVI (por ejemplo, ±2σ).
- Validación con Google Earth Pro: matriz de confusión, Kappa y F1.
- Vectorización, mapas temáticos y cálculo del área de cambio.

## Resultados principales

- **ΔNDVI** muestra una disminución fuerte y confinada al corredor del Ribeirão Ferro-Carvão y al cauce del Paraopeba. El rango observado va de −0,406 a 0,144.
- **NDWI** identifica consistentemente el agua superficial (NDWI t2 entre −0,819 y 0,439), pero es poco sensible al material depositado.
- **Conclusión:** el NDVI es el índice más discriminante para delimitar la zona de cambio. El NDWI complementa el análisis del río.

## Cómo compilar el reporte

1. Comprime la carpeta `report/`, o sube `main.tex`, `references.bib` y `figures/`.
2. En [Overleaf](https://www.overleaf.com/) ve a **New Project → Upload Project** y selecciona el ZIP.
3. Compila con **pdfLaTeX**. La bibliografía usa **biber**, que Overleaf ejecuta automáticamente.

Para compilar en local:

```bash
cd report
latexmk -pdf main.tex
```

## Cómo reproducir el análisis en QGIS

1. Descarga desde Copernicus Browser las bandas B03, B04, B08 y SCL (*Raw*) para ambas fechas, con los parámetros de la tabla de datos.
2. Carga los rásteres y `data/ROI_Brumadinho.gpkg` en QGIS.
3. Recorta cada banda con el ROI.
4. Genera las máscaras de nubes con la expresión SCL.
5. Calcula los índices y sus diferencias en la calculadora ráster:
   ```
   NDVI  = ("B08" - "B04") / ("B08" + "B04")
   NDWI  = ("B03" - "B08") / ("B03" + "B08")
   dNDVI = "NDVI_t2" - "NDVI_t1"
   dNDWI = "NDWI_t2" - "NDWI_t1"
   ```

## Pendientes

- [ ] Re-descargar la banda SCL en formato *Raw*, de una sola banda con valores de 0 a 11, y regenerar las máscaras de nubes.
- [ ] Corregir la simbología del mapa ΔNDWI y actualizar `figures/dndwi.png`.
- [ ] Completar las estadísticas de los índices (media y desviación estándar) dentro del ROI.
- [ ] Desarrollar la Fase 4.

## Referencias

- Drusch, M. et al. (2012). Sentinel-2: ESA's Optical High-Resolution Mission for GMES Operational Services. *Remote Sensing of Environment*, 120, 25–36.
- Lillesand, T., Kiefer, R. W., & Chipman, J. (2015). *Remote Sensing and Image Interpretation* (7th ed.). Wiley.
- Main-Knorn, M. et al. (2017). Sen2Cor for Sentinel-2. *Proc. SPIE 10427*.
- McFeeters, S. K. (1996). The use of the Normalized Difference Water Index (NDWI) in the delineation of open water features. *International Journal of Remote Sensing*, 17(7), 1425–1432.
- QGIS Development Team (2026). *QGIS Geographic Information System*. Open Source Geospatial Foundation.
- Rouse, J. W. et al. (1974). Monitoring Vegetation Systems in the Great Plains with ERTS. *NASA SP-351*.
- Singh, A. (1989). Digital change detection techniques using remotely-sensed data. *International Journal of Remote Sensing*, 10(6), 989–1003.
- Tucker, C. J. (1979). Red and photographic infrared linear combinations for monitoring vegetation. *Remote Sensing of Environment*, 8(2), 127–150.