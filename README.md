# Tracking de partículas brownianas

Este código obtiene las trayectorias de partículas libres en videos de microscopía de campo claro.
Lo usamos en el Laboratorio 5 (FCEN-UBA) para medir el coeficiente de difusión de partículas de látex de 3,24 µm en agua con glicerina y de gotas de grasa de crema diluida en agua.
Todo el procedimiento está en `tracking.ipynb`, dividido en pasos.

## Uso

1. Poner los videos en `videos/<muestra>/`, junto con el video de fondo de esa muestra (filmado sin partículas).
2. En la celda de parámetros, elegir `MUESTRA` (`latex_glycerin` o `water_cream`) y `VIDEO`.
3. Correr todas las celdas. Los resultados quedan en `resultados/<video>/`.

Requiere Python 3 con `numpy`, `pandas`, `scipy`, `matplotlib`, `opencv-python` y `trackpy`.

## Qué hace cada paso

**1. Dividir por el fondo.**
Cada frame se divide por la mediana del video de fondo.
Así desaparecen la iluminación no uniforme y las manchas fijas de la óptica, y las partículas resaltan.
En la crema también restamos la mediana de los frames vecinos, que elimina lo que no se mueve.

**2. Centro por simetría radial.**
En una partícula redonda, los gradientes de intensidad apuntan todos hacia el centro.
El método de Parthasarathy (2012) busca el punto más cercano a esas rectas, con precisión subpíxel.
Además mide cuánto se aparta la partícula de la simetría radial, y esa medida sirve para descartar detecciones.

**3. Detección.**
En el látex, `trackpy` busca discos oscuros de unos 40 px en la imagen invertida.
En la crema, las gotas desenfocadas se ven como anillos y `trackpy` las parte en pedazos.
Por eso buscamos zonas donde la imagen se aparta del fondo más que el ruido.
En ambos casos descartamos las detecciones alargadas o poco simétricas, que suelen ser partículas pegadas o suciedad.

**4. Unión en tracks.**
`trackpy` une las detecciones de frames sucesivos.
Busca cada partícula cerca de su posición prevista: a menos de 8 px en el látex y de 15 px en la crema.
La posición prevista sale de la velocidad reciente de la partícula, para seguir bien la deriva del fluido.
Una partícula puede desaparecer hasta 5 frames sin que se corte su track.

**5. Cortar uniones ambiguas.**
Si había dos candidatas cerca de la posición prevista, no elegimos ninguna y cortamos el track.
Preferimos perder datos antes que unir mal, porque una unión equivocada agrega un salto falso al MSD.

**6. Limpiar los tracks.**
Descartamos los tracks de menos de 20 frames y los de partículas pegadas al vidrio, cuyo MSD no crece con el lag.

**7. Diámetro aparente (solo látex).**
En el látex también hay manchas oscuras más chicas que se mueven.
Para separarlas, promediamos la imagen de la partícula a lo largo de su track y medimos el diámetro del disco oscuro.
Una partícula de 3,24 µm no puede verse menor que su tamaño, así que descartamos los tracks de menos de 3,0 µm.

## Resultados

- `tracks.csv`: columnas `frame, x, y, mass, size, track_id`, con las posiciones en píxeles (0,0948 µm/px).
- `detecciones.pkl`: todas las detecciones, también las descartadas.
- `seleccion_latex.csv`: diámetro aparente de cada track y si corresponde a una partícula de látex.

## Referencias

- J. C. Crocker y D. G. Grier, *J. Colloid Interface Sci.* **179**, 298 (1996).
- R. Parthasarathy, *Nat. Methods* **9**, 724 (2012).
- D. B. Allan et al., *trackpy*, doi:10.5281/zenodo.3492186.
