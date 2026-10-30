## Objetivo e hipótesis
 
El objetivo fue estimar cómo cambia la pérdida de suelo de la cuenca al reemplazar el C = 1 de la capa potencial de CIREN por un C basado en la cobertura vegetal real, y evaluar dónde se concentra esa pérdida.
 
La hipótesis, derivada del plan de descontaminación del Lago Villarrica, fue que las coberturas silvoagropecuarias concentran la mayor parte de la pérdida de suelo estimada, en proporción mayor a la superficie que ocupan, por lo que constituyen las principales fuentes potenciales de sedimento y fósforo particulado hacia el lago.

## Resultados principales
 
- La cobertura vegetal reduce la pérdida media estimada de 191 a 2,9 t ha⁻¹ año⁻¹. Sin vegetación, cerca del 85% del suelo tendría pérdidas de 50 t ha⁻¹ año⁻¹ o más.
- Praderas y cultivos ocupan el 22,5% de la superficie de suelo, pero concentran el 95,5% de la pérdida total estimada (83.045 t año⁻¹). La hipótesis se apoya para estas coberturas, pero no para las plantaciones forestales ni para la franja de 500 m junto al agua.
- Según el umbral de tolerancia adoptado, entre el 6,7% del suelo (máximo tolerable nacional, 12,36 t ha⁻¹ año⁻¹) y el 21,4% (tasa de formación de suelo, 2 t ha⁻¹ año⁻¹) supera la pérdida tolerable.
- Entre 2013-14 y 2023-24 la pérdida media aumentó apenas 0,019 t ha⁻¹ año⁻¹. Ese cambio no depende de los supuestos probados, pero el nivel absoluto de pérdida sí depende del valor de C asignado a las praderas.
![Pérdida de suelo estimada con cobertura, 2023-24](resultados/figura2_perdida_2023_24_gee.png)
 
## Contenido del repositorio
 
| Archivo o carpeta | Contenido |
| --- | --- |
| [`informe.pdf`](informe.pdf) | Informe entregado, con anexos |
| [`Trabajo_RUSLE.ipynb`](Trabajo_RUSLE.ipynb) | Notebook de Google Earth Engine (Python) |
| [`datos/`](datos/) | Capas de pérdida potencial y actual de CIREN para la cuenca (GeoPackage) |
| [`resultados/`](resultados/) | Mapas (PNG), tablas (CSV) y capas exportadas desde GEE (GeoTIFF) |
| [`registro.md`](registro.md) | Decisiones metodológicas, ideas descartadas, análisis complementario y limitaciones |
 
## Cómo reproducir el análisis
 
1. Se necesita una cuenta de Google Earth Engine con un proyecto de Google Cloud registrado.
2. Copiar los archivos de `datos/` a Google Drive, en la carpeta `MyDrive/RUSLE`, porque el notebook los lee desde ahí. Otra opción es cambiar la ruta en la celda de lectura de la cuenca.
3. Abrir `Trabajo_RUSLE.ipynb` en Google Colab, escribir el identificador del propio proyecto de GEE en la celda de autenticación y ejecutar todas las celdas en orden.
4. La celda E1.4 lee la capa de CIREN desde un asset de GEE. Para usar uno propio, subir el shapefile que genera la celda E1.3 como asset y cambiar la ruta en E1.4.
Las capas de Landsat 8, CHIRPS, JRC Global Surface Water y MapBiomas se leen directamente desde el catálogo de GEE, por lo que no se incluyen en el repositorio.
 
## Datos utilizados
 
| Dato | Fuente | Uso |
| --- | --- | --- |
| Pérdida potencial y actual de suelo (`r09_033_rep_1ha`, `r09_033_rea_1ha`) | CIREN, Inventario de Erosión (descargadas el 14 y el 22 de octubre de 2026) | Base del modelo y contraste |
| Landsat 8 Collection 2 Nivel 2 (`LANDSAT/LC08/C02/T1_L2`) | USGS | NDVI de 2013-14 y 2023-24 |
| CHIRPS diario (`UCSB-CHG/CHIRPS/DAILY`) | UCSB Climate Hazards Center | Comparación de precipitación entre periodos |
| JRC Global Surface Water v1.4 (`JRC/GSW1_4/GlobalSurfaceWater`) | Pekel et al. (2016) | Máscara de agua y franja ribereña |
| MapBiomas Chile, Colección 2.0 | MapBiomas Chile | Clases de cobertura para el valor mínimo de C |
 
## Limitaciones principales
 
RUSLE no representa cárcavas, remoción en masa, lahares, erosión en caminos forestales y riberas, ni la entrega del sedimento al lago, por lo que la pérdida estimada no equivale al sedimento que llega al agua. Ni la capa de CIREN ni el escenario propio se pudieron validar con mediciones, porque ninguna estación del sector publica datos de sedimento en suspensión. El nivel absoluto de la pérdida depende de un valor de C poco restringido para las praderas. La lista completa, con la consecuencia de cada limitación, está en [`registro.md`](registro.md).
