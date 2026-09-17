# Notas de análisis

Decisiones metodológicas y hallazgos que no caben en el README.

## 1. Signo del término de segundo armónico

El apéndice A de la guía obtiene

    x(θ) ≈ (L − r²/4L) + r·cos θ − (r²/4L)·cos 2θ

con signo **negativo** en el último término. El desarrollo exacto da signo positivo.
El origen es un cambio de `sin²θ` por `cos²θ` entre la ecuación (14) y la (15) de la guía.

Partiendo de la ecuación (4):

    x = r·cos θ + √(L² − r²·sin²θ)
      ≈ r·cos θ + L·[1 − ½(r/L)²·sin²θ]
      = r·cos θ + L − (r²/2L)·sin²θ

y con sin²θ = (1 − cos 2θ)/2:

    x ≈ (L − r²/4L) + r·cos θ + (r²/4L)·cos 2θ

Verificación numérica con r = 10.9 cm y L = 36 cm, ajustando la ecuación exacta:
A₁ = 0.10900 m, A₂ = **+**0.00845 m, contra r²/4L = 0.00825 m.

**En la práctica no cambia nada**, porque lo que se compara con el experimento es la
magnitud |B| frente a r²/4L. Se documenta aquí por rigor.

## 2. Signo negativo de A en los ajustes

El sensor de movimiento mide la **distancia desde el sensor hasta el carrito**, no la
coordenada x medida desde el eje de la manivela como en la figura 1 de la guía. El eje
está invertido, por lo que el coeficiente del primer armónico sale negativo. Es un
cambio de sistema de referencia, no un error de cálculo.

## 3. Discrepancia en la longitud efectiva de la biela

La amplitud medida del segundo armónico es sistemáticamente **10–27 % menor** que
r²/4L, y en la misma dirección en las cinco corridas. Un error aleatorio se repartiría
hacia ambos lados; esto apunta a la geometría.

Ajustando la ecuación geométrica exacta con L como parámetro libre en las cinco
corridas se obtiene **L = 45.3 ± 3.3 cm**, frente a los 36.0 cm medidos con regla.

Dos explicaciones son compatibles con los datos y **no se distinguen entre sí** (ambas
ajustan con R² prácticamente idéntico):

1. La distancia real **entre centros de perno** es mayor que el largo de barra medido.
2. El eje de la manivela no está a la misma altura que el eje del riel. Con un
   desalineamiento de 1–4 cm el mecanismo deja de ser un biela-manivela alineado y el
   término de segundo armónico se reduce en la proporción observada.

Ninguna de las dos se verificó con una medición adicional, por lo que **no se afirma
cuál es la causa**. Lo que sí se afirma es que la discrepancia es sistemática y de
origen geométrico, no aleatoria.

Nota: la guía indica en la sección 3.2 que r y L "no son utilizadas directamente en los
cálculos posteriores". Esta comparación es un añadido del grupo, no un requisito.

## 4. Elección de los intervalos de análisis

Se usan dos criterios distintos según el objetivo:

- **Ajuste MAS en el tiempo** (notebook 01): ventanas de ~2.5 vueltas, elegidas por
  búsqueda automática del tramo donde el giro fue más parejo. Ventanas más largas
  degradan el ajuste porque ω deriva.
- **DFT** (notebook 02): la ventana se cierra en un número **entero de vueltas** usando
  la columna de ángulo. Así la señal empieza y termina en la misma fase y se minimiza la
  fuga espectral.

Son compromisos opuestos: el ajuste quiere ω estable, la DFT quiere muchos ciclos.

## 5. Corrección de fase en la reconstrucción

Las fases que devuelve la FFT están referidas a la **primera muestra de la ventana**,
no al origen de tiempos del archivo. Si la ventana empieza en t = 1 s y se reconstruye
con `cos(2πf·t + φ)` usando t absoluto, la señal sale desfasada.

Comprobación con una señal sintética de dos cosenos conocidos:

| reconstrucción | error máximo |
|---|---|
| con `tau = t − t[0]` | 0.00000 |
| con `t` absoluto | 0.70531 |

Los notebooks usan `tau`. La plantilla oficial del curso usa `t` absoluto.

## 6. Por qué se añadió la DFT en el dominio del ángulo

La posición x(θ) es puramente geométrica: no depende de ω ni de α. Haciendo la DFT
contra el ángulo en lugar del tiempo, el eje queda en **ciclos por vuelta** y los
armónicos caen en n = 1 y n = 2 exactos, sin importar cómo varió la velocidad de giro.

Es el mismo eje de la figura 8 de la guía ("Armónico (ciclos por vuelta)"), de modo que
la figura de referencia del curso ya está construida en este dominio.

Esta ruta es complementaria, no sustituta: el punto 3 de la guía pide explícitamente la
DFT de la aceleración en el tiempo, y esa está en el notebook 02 sección 2.
