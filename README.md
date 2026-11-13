# Práctica de juegos

Minijuegos de práctica para entrenar antes de jugar los retos diarios. Cada juego imita la mecánica (física, velocidad, dificultad) con un aspecto propio.

## Juegos

| Juego | Archivo | Mecánica |
|---|---|---|
| Franja | `franja/index.html` | Toca para subir la ficha y pasar por el hueco de cada columna. Con cada columna sube la velocidad y se estrecha el hueco. Tocar techo o suelo es fallo. |
| Carriles | `carriles/index.html` | Tres carriles. Toca a izquierda o derecha para cambiar un carril cada vez y esquiva los coches. Los metros recorridos son los puntos; cada vez va más rápido y llegan coches de dos en dos. |

## Cómo jugar

Abre `index.html` (el menú) en el navegador del móvil, o activa GitHub Pages (Settings → Pages → Branch `main`, carpeta `/root`) y entra en la dirección que te da GitHub.

- **Reto del día:** intentos limitados (Franja 3, Carriles 2), cuenta la mejor marca.
- **Práctica libre:** intentos ilimitados.
- **Ajustes:** gravedad, impulso, tamaño del hueco, velocidad… Los valores por defecto están medidos en un vídeo de una partida real.

## Datos

No hay servidor ni base de datos. Las marcas y los ajustes se guardan en el almacenamiento del navegador (`localStorage`) de cada dispositivo.
