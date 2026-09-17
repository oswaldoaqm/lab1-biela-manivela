# Notas sobre los datos experimentales

## Nomenclatura

La guía (sección 3.4) pide el formato `biela-manivela-[frecuenciaHz]-[fecha].txt`.
Como se tomaron **dos corridas por frecuencia**, se añadió el sufijo `-c1` / `-c2`.
La fecha proviene de la cabecera de cada archivo de Logger Pro.

| archivo en `raw/` | clave en los notebooks | hora de captura | estado |
|---|---|---|---|
| biela-manivela-10Hz-08-09-2026-c1.txt | `10Hz-c1` | 13:38:24 | válida |
| biela-manivela-10Hz-08-09-2026-c2.txt | `10Hz-c2` | 13:48:56 | válida (caso detallado) |
| biela-manivela-20Hz-08-09-2026-c1.txt | `20Hz-c1` | 13:40:46 | válida |
| biela-manivela-20Hz-08-09-2026-c2.txt | `20Hz-c2` | 13:49:42 | válida con recorte |
| biela-manivela-30Hz-08-09-2026-c1.txt | `30Hz-c1` | 13:43:10 | **descartada** |
| biela-manivela-30Hz-08-09-2026-c2.txt | `30Hz-c2` | 13:51:02 | válida |

Los seis archivos son los exportados de Logger Pro **sin ninguna modificación**.
Ninguno fue editado, recortado ni corregido en disco. Todo el acotado de intervalos
ocurre dentro de los notebooks y es reversible.

## Corrida descartada: `30Hz-c1`

Excluida del análisis por dos fallas instrumentales verificables:

1. **Arranque sin movimiento.** Durante los primeros ~0.7 s la posición se mantiene
   en 0.500 m (valores entre 0.5004 y 0.5037 m entre t = 0.10 s y t = 0.33 s). El
   carrito estaba quieto: la adquisición empezó antes de que la manivela lo impulsara.

2. **Pico espectral no físico.** El máximo del espectro de la aceleración aparece en
   **3.92 Hz**, un orden de magnitud por encima de la frecuencia de giro real
   (~0.34 Hz). No corresponde a ningún armónico del sistema; es ruido del sensor.

Para verificarlo:

```python
import numpy as np
d = np.genfromtxt("data/raw/biela-manivela-30Hz-08-09-2026-c1.txt",
                  skip_header=7, encoding="utf-8-sig")
t, x = d[:,0], d[:,1]
print(x[(t>=0.1) & (t<=0.35)])        # posicion congelada en 0.50 m
print(100*(x.max()-x.min()))          # recorrido aparente: 34.8 cm en vez de ~22
```

La corrida `30Hz-c2`, tomada 8 minutos después con el mismo montaje, no presenta
ninguno de los dos problemas y es la que se usa para 30 Hz.

## Recorte parcial: `20Hz-c2`

El archivo se usa completo hasta **t = 11.1 s**. A partir de ahí el sensor de
movimiento pierde el carrito: el recorrido por vuelta cae de forma abrupta.

| vuelta | intervalo | recorrido |
|---|---|---|
| 1 | 0.05 – 3.00 s | 22.12 cm |
| 2 | 3.05 – 5.85 s | 20.72 cm |
| 3 | 5.90 – 8.50 s | 21.45 cm |
| 4 | 8.55 – 11.15 s | 19.45 cm |
| 5 | 11.20 – 13.70 s | **13.10 cm** |

El recorrido debe ser 2r ≈ 21.6 cm en todas las vueltas. La quinta registra un 39 %
menos, lo que indica lecturas perdidas y no un cambio real del movimiento. El corte
está declarado en los notebooks mediante la variable `T_FIN`.

## Columnas de los archivos

Orden real de las columnas exportadas, que **difiere** del que asumen las plantillas
oficiales del curso:

| # | columna | símbolo | unidad | sensor |
|---|---|---|---|---|
| 1 | Tiempo | t | s | interfaz Vernier (reloj interno) |
| 2 | Posición | x | m | sensor de movimiento |
| 3 | Velocidad 1 | v | m/s | sensor de movimiento |
| 4 | Aceleración 1 | a | m/s² | sensor de movimiento |
| 5 | Ángulo | θ | rad | sensor rotacional |
| 6 | Velocidad 2 | ω | rad/s | sensor rotacional |
| 7 | Aceleración 2 | α | rad/s² | sensor rotacional |

Las plantillas del curso asumen `Tiempo, Ángulo, ..., Posición` y hacen la DFT sobre
`Aceleración2`, que en estos archivos es la aceleración **angular**. Los notebooks de
este repositorio usan el mapeo correcto.
