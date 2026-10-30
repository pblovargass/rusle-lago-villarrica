# Registro del trabajo

Este documento reúne lo que no cabe en el informe. Explica cada decisión metodológica, las ideas que se probaron y se descartaron, el análisis complementario completo y las limitaciones con su consecuencia. Las tablas del informe y sus anexos son un resumen de lo que aparece aquí. Hay una versión resumida en el anexo del informe. 

Inicialmente fue un Claude Docs, donde le decía a la IA mis ideas, cambios en el código, decisiones, justificaciones, resultados, etc. para que fuese completando el documento. Al final de cada sesión le decía a claude "actualiza el registro" para así no perder información ni reproducibilidad. 

## 1. Decisiones metodológicas

| Tema | Decisión | Justificación |
| --- | --- | --- |
| Área de estudio | Unidad hidrográfica Lago Villarrica de CIREN (`r09_033`), 49.475,6 ha, con 27.843 ha de suelo | Es la unidad del inventario de CIREN. Abarca el lago y su entorno inmediato, no la cuenca completa del plan de descontaminación (2.805 km²) |
| Capa base | Pérdida potencial `r09_033_rep_1ha` (A = R·K·LS, con C = P = 1) | La Etapa 1 exige la capa potencial. REP es la potencial y REA la actual en la nomenclatura de CIREN |
| Datos de CIREN | Copia en Google Drive y asset en GEE | El servidor de CIREN falló varias veces durante el trabajo, y la capa es demasiado pesada para enviarla a GEE desde Python |
| Factor modificado | Solo C, con A = A_CIREN × C, manteniendo R, K y LS de CIREN y P = 1 | En GEE se permite un solo cambio, y usar R, K y LS propios mezclaría cuatro cambios a la vez |
| P | P = 1 | No hay prácticas de conservación documentadas en la cuenca, y las medidas del plan de descontaminación aún no se implementan |
| Sensor | Landsat 8 Collection 2 Nivel 2 en ambos periodos | Con el mismo satélite las imágenes se comparan directamente. Landsat 8 funciona desde abril de 2013 |
| Periodos | Abril a marzo de 2013-14 y de 2023-24 | Misma ventana estacional, diez años de diferencia y línea base anterior a la erupción del volcán Villarrica de marzo de 2015 |
| Comparabilidad climática | Precipitación CHIRPS de 1.600 mm (2013-14) y 1.750 mm (2023-24) | La diferencia es de cerca de 9%, así que los cambios de NDVI se atribuyen principalmente a la cobertura |
| Nubes | Filtro de escenas con menos de 30% de nubes, máscara por píxel con QA_PIXEL y compuesto de mediana | Las nubes y sombras bajan el NDVI y simularían pérdida de vegetación |
| Máscara de agua | Píxeles con agua al menos el 50% del tiempo (JRC Global Surface Water) | El lago distorsionaba los promedios. Se conservan las zonas urbanas para ver la expansión urbana en el NDVI |
| Máscara de suelo | Clases 1 a 7 de CIREN | RUSLE no aplica sobre agua, roca ni superficies selladas |
| Fórmula de C | C = 0,431 − 0,805·NDVI, acotada entre 0 y 1 (De Jong, 1994) | Fórmula sugerida para el curso. Da C = 0 con NDVI mayor que 0,535 |
| Valor mínimo de C | C final igual al mayor valor entre la fórmula de De Jong y el valor mínimo de la clase de MapBiomas | Corrige la saturación de la fórmula en vegetación densa. Con un valor único, las praderas recibían el mismo C que el bosque |
| Mapa de cobertura | MapBiomas 2023 para ambos periodos | Las reclasificaciones entre años incluyen errores evidentes y explicaban por sí solas la baja de la erosión (sección 3.3) |
| Resolución | 30 m en EPSG:32719, sin escala automática en los cálculos de área | Es la resolución de Landsat, la más fina de las capas combinadas |
| Límites de clase | El límite inferior pertenece a la clase superior | Es la convención de los rangos de CIREN |
| Franja ribereña | Suelo a 500 m o menos del agua permanente | Representa el suelo con el camino más corto al agua. El ancho es un supuesto |
| Umbrales | 2 y 8 t ha⁻¹ año⁻¹ (Ellies, 2000), 5 (Peña, 1985) y 12,36 (Núñez-Hidalgo et al., 2025) | Son umbrales chilenos de origen distinto, que responden preguntas distintas y no se homogeneizan |
| Superación de umbrales | Se usa "supera" (mayor que) y "alcanza o supera" como rango de incertidumbre | El umbral es el máximo aceptable, y los valores justo en el umbral son en gran parte efecto del redondeo de CIREN |

### Valores mínimos de C por clase de cobertura

| Código MapBiomas | Clase | Equivalente en Panagos et al. (2015) | Rango de C | Valor usado |
| --- | --- | --- | --- | --- |
| 3, 59, 60 | Bosque | Bosques | 0,0001–0,003 | 0,0001 |
| 9 | Silvicultura | Bosques | 0,0001–0,003 | 0,0001 |
| 67 | Bosque achaparrado | Bosque-matorral de transición | 0,003–0,05 | 0,003 |
| 66 | Matorral | Brezales y matorral | 0,01–0,1 | 0,01 |
| 12, 63 | Pastizal, estepa | Pastizales naturales | 0,01–0,08 | 0,01 |
| 11 | Humedal | Sin valor en Panagos, asimilado a pastizal | — | 0,01 |
| 15 | Pastura | Praderas | 0,05–0,15 | 0,10 |
| 18 | Agricultura | Cereales | 0,20 | 0,20 |
| 25 | Otra área sin vegetación | Vegetación escasa | 0,1–0,45 | 0,1 |

El bosque usa el mínimo de su rango porque su dosel denso cubre el suelo. La pastura usa el centro del rango porque la inspección visual mostró pasto corto, con cubierta incompleta y zonas secas. Esa evidencia se limita a pocos potreros en imágenes de fecha desconocida, y por eso se acompaña de una prueba de sensibilidad.

## 2. Ideas que se probaron y se descartaron

| Idea inicial | Por qué se cambió | Qué la reemplazó |
| --- | --- | --- |
| Cambiar K en la Etapa 2 | Se optó por cambiar C con NDVI, después descubrí Mapbiomas y lo sumé al trabajo | C con NDVI. Stolpe (2005) y Bonilla y Johnson (2012) quedaron para discutir la incertidumbre de K |
| Calcular el escenario con R, K, L, S, C y P propios | Cambiaría cuatro o más factores respecto de CIREN | A = RUSLE de CIREN × C |
| Factor P con coberturas de MODIS | Sin practicas actuales, si planeadas, pero nada en ejecución | P = 1 |
| Asignar P menor que 1 según el plan de descontaminación | Son medidas futuras, el registro de fertilizantes no cambia la pérdida de suelo y la restauración ribereña actúa sobre la entrega del sedimento | P = 1, con la mención de que prácticas futuras reducirían P |
| Capa `r09_033_rea_1ha` como base | Era la capa actual, no la potencial | `r09_033_rep_1ha` |
| Leer la capa directamente del servidor de CIREN | El servidor falló varias veces | Copia en GeoPackage en Drive |
| Enviar la capa a GEE desde Python | Supera el límite de tamaño de una solicitud | Asset en GEE |
| Un valor mínimo único de C (0, 0,001, 0,002 y 0,003) | El 98% del suelo quedaba en ese valor y las praderas recibían el C del bosque | Valores mínimos por clase de MapBiomas |
| Valor mínimo de la pastura en 0,05 | La inspección visual mostró cubierta incompleta | 0,10, con 0,05 y 0,15 como sensibilidad |
| NDVI sin enmascarar nubes ni agua | Las nubes y el lago distorsionaban el NDVI. El C medio daba 0,072 | Máscara QA_PIXEL y máscara de agua JRC |
| Una sola máscara de suelo para NDVI y C | Eliminaba las ciudades del NDVI | Máscara de tierra para NDVI y de suelo para C |
| Clases de MapBiomas de cada año | Reclasificaciones poco creíbles (241 ha de pradera a bosque primario) explicaban toda la baja de la erosión | Mapa de 2023 para ambos periodos |
| Hipótesis de que el cambio se concentraba en torno a Pucón y Villarrica | La profesora indicó seguir con la hipótesis de sedimento y fósforo. Además, los aumentos estaban lejos de las ciudades | Hipótesis sobre el aporte de las coberturas silvoagropecuarias |
| Contrastar con estaciones de sedimento de la DGA | Las estaciones del sector no registran sedimento en suspensión, o no lo publican | Se declara como limitación |
| NDVI sobre el lago como indicador de sedimentos | La mediana anual borra las floraciones y el NDVI sobre agua es ruidoso | Se menciona como trabajo futuro |
| Umbrales de Verheijen et al. (2009) y del USDA (11,2 t ha⁻¹ año⁻¹) | Existen umbrales chilenos para suelos comparables | Ellies (2000), Peña (1985) y Núñez-Hidalgo et al. (2025) |

## 3. Análisis complementario

### 3.1 Etapa 1, superficie por clase en ambas plataformas

| Clase | Vectorial (ha) | GEE (ha) | Diferencia |
| --- | --- | --- | --- |
| Nula | 2.762,5 | 3.021,9 | +9,4% |
| Muy leve | 158,7 | 187,7 | +18,2% |
| Leve | 414,7 | 489,6 | +18,1% |
| Moderada-leve | 973,3 | 1.130,0 | +16,1% |
| Moderada-grave | 4.271,9 | 4.655,7 | +9,0% |
| Grave | 7.948,6 | 7.938,5 | −0,1% |
| Muy grave | 11.313,5 | 10.898,4 | −3,7% |
| No suelos | 21.632,2 | 21.108,3 | −2,4% |

Las diferencias son mayores en las clases formadas por celdas aisladas, que tienen mucho borde en proporción a su área. Al convertir los polígonos en píxeles, cada píxel de borde se asigna a una sola clase, y ese reparto suma o resta superficie. Por el mismo motivo, la superficie con suelo es de 27.843 ha en el cálculo vectorial y de cerca de 28.790 ha al rasterizar a 30 m.

### 3.2 Diagnóstico del cambio de cobertura

Los compuestos de NDVI se construyeron con 29 escenas en 2013-14 y 32 en 2023-24. El NDVI medio de la tierra fue de 0,764 en ambos periodos, pero el promedio ocultaba cambios que se compensaban. La diferencia entre periodos tuvo una media de 0,0, un percentil 5 de −0,117 y un percentil 95 de +0,117. Con un umbral de ±0,1, perdieron vegetación de forma marcada 1.921 ha, la ganaron 2.090 ha y 27.924 ha no tuvieron cambio marcado. La inspección visual de algunas zonas con ganancia mostró rodales uniformes con hileras de plantación, compatibles con rotación forestal.

Con el C final, el valor mínimo de la clase determina C en cerca del 99% del suelo. El NDVI solo determina C donde la cobertura es escasa, es decir, cuando baja de un umbral que depende de la clase.

| Clase | Valor mínimo de C | El NDVI determina C si es menor que |
| --- | --- | --- |
| Bosque y silvicultura | 0,0001 | 0,535 |
| Pastura | 0,10 | 0,41 |
| Agricultura | 0,20 | 0,29 |

Por eso el modelo solo capta pérdidas fuertes de cobertura, como cosechas o suelo desnudo, y el cambio entre periodos probablemente subestima el efecto real del cambio de cobertura.

### 3.3 Descomposición del cambio y clasificaciones de MapBiomas

Con la clasificación de MapBiomas de cada año, la pérdida media habría disminuido entre periodos. Un escenario cruzado, con el NDVI de 2013-14 y las clases de 2023, permite separar ambos efectos.

| Componente (t ha⁻¹ año⁻¹) | Pastura en 0,05 | Pastura en 0,10 |
| --- | --- | --- |
| Cambio total con clases de cada año | −0,259 | −0,820 |
| Por cambio de clases de MapBiomas | −0,277 | −0,839 |
| Por cambio de NDVI | +0,018 | +0,019 |

La disminución provenía completamente del cambio de clases, mientras el NDVI indicaba un leve aumento. Entre 2013 y 2023, MapBiomas reclasifica 1.882 ha de pradera como bosque, y 241 de ellas como bosque primario, algo incompatible con diez años. Como no se puede separar el cambio real del error de clasificación, se usó el mapa de 2023 para ambos periodos. Con las clases de cada año subían de clase 785 ha y bajaban 1.586. Con el mapa único suben 125 ha y bajan 99.

| Código | Clase | 2013 (ha) | 2023 (ha) |
| --- | --- | --- | --- |
| 60 | Bosque secundario | 10.356 | 10.664 |
| 15 | Pastura | 8.301 | 5.842 |
| 59 | Bosque primario | 5.213 | 7.630 |
| 9 | Silvicultura | 2.866 | 2.487 |
| 66 | Matorral | 1.092 | 895 |
| 11 | Humedal | 434 | 423 |
| 18 | Agricultura | 217 | 611 |
| 67 | Bosque achaparrado | 147 | 85 |
| 25 | Otra área sin vegetación | 83 | 23 |
| 33 | Río, lago u océano | 63 | 57 |
| 12 | Pastizal | 11 | 54 |
| 24 | Infraestructura | 5 | 13 |
| 23 | Arena, playa y duna | 3 | 4 |
| 34 | Hielo y nieve | 2 | 2 |
| 29 | Afloramiento rocoso | 1 | 2 |

### 3.4 Sensibilidad a los valores mínimos de C

| Valor mínimo del bosque | 2013-14 (t ha⁻¹ año⁻¹) | 2023-24 (t ha⁻¹ año⁻¹) | Diferencia |
| --- | --- | --- | --- |
| 0,0001 | 2,876 | 2,895 | +0,019 |
| 0,0012 | 3,053 | 3,072 | +0,019 |
| 0,003 | 3,344 | 3,362 | +0,019 |

| Valor mínimo de la pastura | 2013-14 (t ha⁻¹ año⁻¹) | 2023-24 (t ha⁻¹ año⁻¹) | Diferencia |
| --- | --- | --- | --- |
| 0,05 | 1,698 | 1,716 | +0,018 |
| 0,10 | 2,876 | 2,895 | +0,019 |
| 0,15 | 4,056 | 4,077 | +0,022 |

El valor de la pastura es el supuesto con mayor efecto sobre el nivel de la pérdida, porque las praderas concentran casi toda la pérdida estimada. La diferencia entre periodos casi no cambia, porque cada píxel tiene el mismo valor mínimo en ambos periodos y su efecto se cancela al restar.

| Fuente de incertidumbre | Rango probado | Efecto sobre la pérdida media | ¿Cambia la diferencia entre periodos? |
| --- | --- | --- | --- |
| Valor mínimo de la pastura | 0,05 a 0,15 | De 1,7 a 4,1 t ha⁻¹ año⁻¹ | No |
| Valor mínimo del bosque | 0,0001 a 0,003 | +16% | No |
| Límite de 500 t ha⁻¹ año⁻¹ de la capa potencial | ×1 a ×2 | +5% | No |
| Clasificación de MapBiomas por año | Clases de cada año contra mapa único | Explicaba toda la baja | Sí |

### 3.5 Aporte de cada cobertura a la pérdida total (2023-24)

| Cobertura | ha | % superficie | t año⁻¹ | % pérdida total | Pérdida media (t ha⁻¹ año⁻¹) |
| --- | --- | --- | --- | --- | --- |
| Pastura | 5.841,1 | 20,4 | 68.015,5 | 81,9 | 11,6 |
| Agricultura | 610,1 | 2,1 | 11.309,8 | 13,6 | 18,5 |
| Silvicultura | 2.487,1 | 8,7 | 1.124,8 | 1,4 | 0,5 |
| Bosque secundario | 10.656,9 | 37,2 | 843,8 | 1,0 | 0,1 |
| Matorral | 894,2 | 3,1 | 739,5 | 0,9 | 0,8 |
| Bosque primario | 7.610,4 | 26,5 | 413,3 | 0,5 | 0,1 |
| Humedal | 421,6 | 1,5 | 344,9 | 0,4 | 0,8 |
| Pastizal | 54,3 | 0,2 | 142,7 | 0,2 | 2,6 |
| Otra área sin vegetación | 23,1 | 0,1 | 58,4 | 0,1 | 2,5 |
| Bosque achaparrado | 85,3 | 0,3 | 51,8 | 0,1 | 0,6 |

La pérdida total es de 83.045 t año⁻¹ sobre 28.684 ha de suelo. Es suelo que se desprende en las laderas según el modelo, no sedimento que llega al lago. La franja de 500 m junto al agua ocupa el 10,1% del suelo y aporta el 8,6% de la pérdida, con una pérdida media de 2,46 t ha⁻¹ año⁻¹, menor que la del resto (2,94).

### 3.6 Umbrales de tolerancia por clase (2023-24, porcentaje de cada clase sobre el umbral)

| Clase | > 2 | > 5 | > 8 | > 12,36 |
| --- | --- | --- | --- | --- |
| Pastura | 91,5 | 80,0 | 59,8 | 25,7 |
| Agricultura | 90,2 | 86,4 | 76,2 | 62,8 |
| Pastizal | 27,2 | 9,8 | 6,8 | 3,8 |
| Otra área sin vegetación | 23,4 | 17,2 | 9,7 | 5,1 |
| Matorral | 5,0 | 0,4 | 0,3 | 0,2 |
| Silvicultura | 3,6 | 2,8 | 2,1 | 1,2 |
| Humedal | 2,6 | 1,2 | 0,8 | 0,1 |
| Bosque secundario | 0,7 | 0,4 | 0,2 | 0,1 |
| Bosque achaparrado | 0,2 | 0,2 | 0,2 | 0,1 |
| Bosque primario | 0,1 | 0,1 | 0,1 | 0,1 |

| Valor mínimo de la pastura | > 2 (ha) | > 5 (ha) | > 8 (ha) | > 12,36 (ha) |
| --- | --- | --- | --- | --- |
| 0,05 | 5.767 | 2.182 | 2.068 | 903 |
| 0,10 | 6.138 | 5.333 | 4.053 | 1.934 |
| 0,15 | 6.271 | 5.854 | 5.232 | 3.925 |

Si se cuentan como superados los valores justo en el umbral, la superficie sobre 2, 5 y 8 t ha⁻¹ año⁻¹ aumenta entre 4% y 11%. Esto ocurre porque la capa de CIREN está redondeada (por ejemplo, 50 × 0,10 = 5,0). El umbral de 12,36 no cambia porque no es un número redondo.

### 3.7 Contraste con la capa actual de CIREN

La ficha de `r09_033_rea_1ha` dice que se calculó con C = P = 1, pero su pérdida media (1,16 t ha⁻¹ año⁻¹) es unas 170 veces menor que la de la capa potencial (198,89). La ficha contiene un error y la capa sí incorpora un C, cuyo método no está documentado públicamente.

| Clase | CIREN actual (%) | Escenario 2023-24 (%) |
| --- | --- | --- |
| Nula | 93,8 | 80,3 |
| Muy leve | 3,9 | 7,1 |
| Leve | 2,0 | 10,5 |
| Moderada-leve | 0,3 | 1,7 |
| Moderada-grave | 0,1 | 0,3 |
| Grave | 0,04 | 0,0 |
| Muy grave | 0,02 | 0,0 |

Ambas estimaciones tienen el mismo orden de magnitud. El escenario propio estima más pérdida, sobre todo por el C de las praderas. El C efectivo medio es de cerca de 0,006 en CIREN y de 0,015 en el escenario propio.

### 3.8 Calidad de la capa potencial de CIREN

La capa potencial tiene 822 valores distintos, pero se concentra en cifras redondas (500, 200, 100, 300 y 400, y múltiplos de 10 bajo 100). Cerca del 19% de las celdas está exactamente en el máximo de 500 t ha⁻¹ año⁻¹, lo que indica un límite impuesto. Si esas celdas tuvieran el doble de pérdida potencial, la pérdida media de 2023-24 subiría de 2,895 a 3,029 t ha⁻¹ año⁻¹, menos de 5%, porque corresponden sobre todo a laderas boscosas con C muy bajo.

## 4. Limitaciones

**Datos y fuentes**

- Ni la capa de CIREN ni el escenario propio se validaron con mediciones. CIREN solo validó los estados erosivos (550 puntos, 80,9% de exactitud), y las estaciones de la DGA del sector no publican sedimento en suspensión.
- El monitoreo del lago se hace solo en primavera y verano, así que no capta los pulsos de sedimento del invierno lluvioso.
- La capa potencial de CIREN está redondeada y limitada a 500 t ha⁻¹ año⁻¹, lo que subestima las zonas más susceptibles, aunque cambia la pérdida media en menos de 5%.
- La ficha de la capa actual de CIREN es errónea y el método de su C no está documentado.
- La unidad analizada no incluye la cuenca del río Trancura, que aporta cerca del 90% del agua del lago, así que gran parte del sedimento que llega al lago probablemente se origina fuera del área.
- La inspección visual de coberturas se hizo con imágenes de fecha desconocida.

**Método**

- La fórmula de De Jong se ajustó en otro contexto y se satura en vegetación densa, por lo que solo capta pérdidas fuertes de cobertura y no distingue plantación de bosque nativo.
- Los rangos de C de Panagos et al. (2015) son europeos, y el valor de la pastura se basa en una inspección visual limitada.
- MapBiomas tiene errores de clasificación entre años, por lo que los cambios reales de clase solo se captan a través del NDVI.
- El C anual no considera la estacionalidad de la lluvia, así que subestima praderas o cultivos desnudos en invierno.
- P = 1 sobreestima la pérdida donde existan prácticas de conservación.
- Con solo dos fechas se capta la fase de cada rodal forestal, no su ciclo de rotación.
- La franja de 500 m y el umbral de cambio de NDVI de ±0,1 son supuestos.

**Procesos que RUSLE no modela**

Cárcavas, remoción en masa, lahares del volcán Villarrica, erosión en caminos forestales y riberas, escorrentía urbana y la entrega del sedimento al lago. Por eso la pérdida estimada no equivale al sedimento que llega al agua, y la erosión real en las laderas intervenidas probablemente es mayor.

**Interpretación**

- La pérdida total no es sedimento ni fósforo. Estimar la carga de fósforo requeriría el contenido de fósforo del suelo, su enriquecimiento al erosionarse y la fracción que llega al agua.
- La erosión no es la única fuente de fósforo del lago, porque también aportan las pisciculturas y las aguas servidas.
- No hay un umbral de tolerancia oficial en Chile, y el de trumaos se extrapola a pendientes mayores de 18%.

## 5. Fuentes consultadas

Las referencias completas están en el informe. Esta lista resume qué aportó cada fuente al trabajo.

| Fuente | Qué aportó |
| --- | --- |
| CIREN, Inventario de Erosión y resumen ejecutivo de la macrozona sur | Capas base, propósito del inventario y validación de los estados erosivos |
| Ministerio del Medio Ambiente (2018, 2026) | Declaración de zona saturada y plan de descontaminación del lago |
| SMA (2025) | Incumplimiento de la norma de calidad del lago por fósforo total y transparencia |
| Jofré (2024) | Datos del lago, antecedentes de eutrofización y medidas del plan |
| Núñez-Hidalgo et al. (2025) | Evaluación nacional de erosión y umbral de 12,36 t ha⁻¹ año⁻¹ |
| Ellies (2000) | Tasa de formación de suelo en Andisoles del sur de Chile |
| Peña (1985) | Tolerancia de pérdida en trumaos según la pendiente |
| Stolpe (2005) | Comparación de RUSLE, EPIC y WEPP en un suelo volcánico chileno |
| Bonilla y Johnson (2012) | Estimación de K en Chile central y límites de las ecuaciones de K |
| Panagos et al. (2015) | Rangos de C por cobertura |
| De Jong (1994) | Fórmula de C a partir de NDVI |
| Merritt et al. (2003), Alewell et al. (2019), Borrelli et al. (2021), Benavidez et al. (2018) | Tipos de modelos de erosión y fortalezas y limitaciones de RUSLE |
| Schindler (1977), Sharpley (1985) | Fósforo como limitante de las algas y su transporte con el sedimento |
