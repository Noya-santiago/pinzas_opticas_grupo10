# Tracking de partículas brownianas

Este código obtiene las trayectorias de partículas libres en videos de movimiento browniano.
Lo usamos en el Laboratorio 5 para medir el coeficiente de difusión de partículas de látex y de gotas de grasa.
Todo el procedimiento está en `tracking.ipynb`, dividido en pasos.

## Uso

1. Poner los videos en `videos/<muestra>/`, junto con el video de fondo de esa muestra (filmado sin partículas).
2. En la celda de parámetros, elegir `MUESTRA` (`latex_glycerin` o `water_cream`) y `VIDEO`.
3. Correr todas las celdas. Los resultados quedan en `resultados/<video>/`.

## Qué hace cada paso

**1. Dividir por el fondo.**
Cada frame se divide por la mediana del fondo.

**2. Centro por simetría radial.**
El método de Parthasarathy (2012) busca el punto más cercano a esas rectas, con precisión subpíxel.
Además mide cuánto se aparta la partícula de la simetría radial, y esa medida sirve para descartar detecciones.

**3. Detección.**
En el látex, `trackpy` busca discos oscuros de unos 40 px en la imagen invertida.
En la crema, las gotas desenfocadas se ven como anillos y `trackpy` las parte en pedazos.
Por eso buscamos zonas donde la imagen se aparta del fondo más que el ruido.
En ambos casos descartamos las detecciones alargadas o poco simétricas, que suelen ser partículas pegadas o suciedad.

**4. Unión en tracks.**
`trackpy` une las detecciones de frames sucesivos.
Busca cada partícula cerca de su posición prevista
La posición prevista sale de la velocidad reciente de la partícula, para seguir bien la deriva del fluido.

**5. Cortar uniones ambiguas.**
Si hay dos candidatas ambiguas a un linkeo, no se toma ninguna. 

**6. Diámetro aparente **
Se promedia la imagen de la partícula a lo largo de su track y se mide el diámetro del disco oscuro, se filtra en base a esto

## Resultados

- `tracks.csv`: columnas `frame, x, y, mass, size, track_id`, con las posiciones en píxeles (0,0948 µm/px).
- `detecciones.pkl`: todas las detecciones, también las descartadas.
- `seleccion_latex.csv`: diámetro aparente de cada track y si corresponde a una partícula de látex.

## Referencias

- J. C. Crocker y D. G. Grier, *J. Colloid Interface Sci.* **179**, 298 (1996).
- R. Parthasarathy, *Nat. Methods* **9**, 724 (2012).
- D. B. Allan et al., *trackpy*, doi:10.5281/zenodo.3492186.
