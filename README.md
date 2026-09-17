# Análisis armónico en el sistema biela-manivela

Laboratorio 1 — **Óptica y Ondas**, Departamento de Ciencias
Universidad de Ingeniería y Tecnología (UTEC), ciclo 2026-II

Datos tomados el **8 de septiembre de 2026**.

---

## Enlaces del entregable

| | |
|---|---|
| Video de explicación (YouTube) | `PEGAR ENLACE AQUÍ` |
| Datos experimentales | este repositorio, carpeta [`data/raw/`](data/raw) |
| Guía del laboratorio | [`docs/guia-laboratorio.pdf`](docs/guia-laboratorio.pdf) |

**Integrantes:** `COMPLETAR`

---

## Objetivo

Analizar el primer y segundo armónico en el sistema biela-manivela, verificando
experimentalmente que el movimiento del pistón **no** es un Movimiento Armónico Simple
(MAS) ideal, sino que contiene un término anarmónico originado por la longitud finita
de la biela.

## Conclusión principal

El movimiento del carrito se aparta de un MAS de forma medible y sistemática. El ajuste
de la posición contra el ángulo de la manivela con un solo armónico deja un residuo de
**5.0 mm rms** que no es ruido: oscila dos veces por vuelta. Al incorporar el término
`cos 2θ` predicho por la geometría, el residuo baja a **1.4 mm rms**, o sea el segundo
armónico explica cerca del **90 %** de la varianza que el MAS no lograba explicar.

El radio de la manivela obtenido de las cinco corridas válidas es
**r = 10.80 ± 0.23 cm** (dispersión de 2.1 % entre tres frecuencias de muestreo
distintas), con r/L = 0.300, dentro del rango de validez de la aproximación.

## Palabras clave

MAS · aceleración · biela · armónico · anarmónico · pistón · Vernier · Fourier · DFT

---

## Parámetros del sistema

| Magnitud | Valor | Origen |
|---|---|---|
| Longitud de la biela, L | 36.0 cm | medida con regla |
| Radio de la manivela, r | 12.5 cm | medida con regla |
| Radio de la manivela, r | **10.80 ± 0.23 cm** | del recorrido del carrito, 5 corridas |
| Razón r/L | 0.300 | cumple r/L < 1/3 |
| Frecuencia fundamental f₁ | 0.34 – 0.37 Hz | espectro de la aceleración |

La discrepancia entre los dos valores de r está discutida en
[`docs/NOTAS-ANALISIS.md`](docs/NOTAS-ANALISIS.md).

---

## Estructura

```
.
├── data/
│   ├── raw/                     6 archivos .txt exportados de Logger Pro, SIN editar
│   └── NOTAS.md                 corridas descartadas y por qué
├── notebooks/
│   ├── 01_movimiento_y_ajuste_MAS.ipynb    puntos 1 y 2 de la guía
│   ├── 02_analisis_espectral.ipynb         puntos 3 y 4 de la guía
│   └── 03_comparacion_frecuencias.ipynb    las 5 corridas y el resumen estadístico
├── figuras/                     PNG generados por los notebooks
├── fotos/                       fotografías tomadas durante el laboratorio
└── docs/
    ├── guia-laboratorio.pdf
    ├── NOTAS-ANALISIS.md        decisiones metodológicas y hallazgos
    └── tabla_resumen.{csv,md}   generados por el notebook 03
```

## Cómo reproducir

```bash
pip install -r requirements.txt
cd notebooks
jupyter lab
```

Los notebooks encuentran `data/raw/` automáticamente, se ejecuten desde `notebooks/`
o desde la raíz. En los notebooks 01 y 02 se cambia una sola línea para analizar otra
corrida:

```python
CORRIDA = "10Hz-c2"    # 10Hz-c1 | 10Hz-c2 | 20Hz-c1 | 20Hz-c2 | 30Hz-c2
```

El notebook 03 procesa las cinco de una pasada y escribe la tabla resumen en `docs/`.

### En Google Colab

Sube la carpeta completa a Drive y monta el Drive, o bien sube los 6 `.txt` al
directorio de trabajo del notebook: la función `dir_datos()` también busca en `.`.

---

## Resultados

r = **10.80 ± 0.23 cm** (n = 5, tres frecuencias de muestreo).

| corrida | fs (Hz) | vueltas | r (cm) | B (cm) | B teór. (cm) | desv. ω (%) | f₂/f₁ | rms MAS → +2°arm (mm) |
|---|---|---|---|---|---|---|---|---|
| 10Hz-c1 | 10 | 5 | 11.04 | 0.691 | 0.847 | 24.1 | 2.4 | 5.45 → 2.26 |
| 10Hz-c2 | 10 | 5 | 10.85 | 0.648 | 0.817 | 16.0 | 2.2 | 4.84 → 1.45 |
| 20Hz-c1 | 20 | 5 | 11.00 | 0.750 | 0.840 | 25.5 | 2.2 | 6.95 → 4.42 |
| 20Hz-c2 | 20 | 3 | 10.57 | 0.583 | 0.776 | 17.8 | 2.0 | 6.01 → 4.36 |
| 30Hz-c2 | 30 | 4 | 10.57 | 0.493 | 0.776 | 14.6 | 2.0 | 6.34 → 5.31 |

`B` es la amplitud del término `cos 2θ` en la posición; su valor teórico es r²/4L.

**La corrida `10Hz-c2` es la que se usa como caso detallado en el video:** es la única
cuya reconstrucción con dos armónicos alcanza R² > 0.9 y la de menor contaminación por
giro no uniforme.

## Hallazgos

1. **El segundo armónico es visible sin necesidad de transformada.** El residuo del
   ajuste MAS de x(θ) oscila dos veces por vuelta. Incorporar `cos 2θ` reduce el residuo
   a un tercio.

2. **La velocidad angular varía 15–25 % respecto a su media.** El giro manual es
   *aproximadamente* uniforme, no MCU estricto, y eso ensancha los picos del espectro.

3. **El espectro de la aceleración es poco fiable en este montaje, y se puede demostrar
   por qué.** Si ω no es constante, la regla de la cadena da
   `a = f''(θ)·ω² + f'(θ)·α`. El segundo término existe solo por el giro manual y se midió
   entre 0.8 y 1.8 veces el tamaño del segundo armónico geométrico: contamina al mismo
   orden de magnitud.

4. **Subir la frecuencia de muestreo no mejoró la señal de aceleración.** Logger Pro la
   obtiene derivando la posición dos veces, y el ruido de cuantización se amplifica como
   1/Δt². El R² del ajuste MAS de a(t) cae de 0.93 a 10 Hz hasta ~0.53 a 30 Hz.

5. **Dos sensores independientes coinciden.** ω del ajuste MAS sobre el sensor de
   movimiento = 2.386 ± 0.005 rad/s; media del sensor rotacional = 2.334 rad/s.
   Coinciden dentro del 2 %.

---

## Integridad de los datos

Los seis archivos en `data/raw/` son los exportados directamente de Logger Pro, sin
modificación alguna. Dos corridas se excluyeron del análisis por fallas instrumentales
identificables y medibles; el criterio está documentado en
[`data/NOTAS.md`](data/NOTAS.md). Los archivos completos se conservan en el repositorio
para que cualquiera pueda verificar la decisión.
