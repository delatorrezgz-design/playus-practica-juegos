# Práctica de juegos

Minijuegos de práctica para entrenar antes de jugar los retos diarios. Cada juego imita la mecánica (física, velocidad, dificultad) con un aspecto propio.

## Juegos

| Juego | Archivo | Mecánica |
|---|---|---|
| Franja | `index.html` | Toca para subir la ficha y pasar por el hueco de cada columna. Con cada columna sube la velocidad y se estrecha el hueco. Tocar techo o suelo es fallo. |

## Cómo jugar

Abre `index.html` en el navegador del móvil, o activa GitHub Pages (Settings → Pages → Branch `main`, carpeta `/root`) y entra en la dirección que te da GitHub.

- **Reto del día:** 3 intentos, cuenta la mejor marca.
- **Práctica libre:** intentos ilimitados.
- **Ajustes:** gravedad, impulso, tamaño del hueco, velocidad… Los valores por defecto están medidos en un vídeo de una partida real.

## Datos

No hay servidor ni base de datos. Las marcas y los ajustes se guardan en el almacenamiento del navegador (`localStorage`) de cada dispositivo.
