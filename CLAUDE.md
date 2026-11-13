# Práctica de juegos

Minijuegos web de práctica que imitan la mecánica (no el aspecto ni el nombre) de los juegos diarios de una app móvil. El usuario no es programador: explícale las cosas en español y sin jerga.

## Estructura

- `index.html`: menú con un enlace por juego.
- `franja/index.html`: "Franja" (tocar para subir y pasar por huecos entre columnas).
- `carriles/index.html`: "Carriles" (3 carriles, tocar izquierda/derecha, esquivar coches; el real se llama Lane Rush).
- Cada juego es una página única, sin dependencias ni compilación, con el mismo estilo visual (el CSS está copiado en cada página).
- Se publica con GitHub Pages desde `main`, carpeta raíz: https://delatorrezgz-design.github.io/playus-practica-juegos/

## Convenciones de los juegos

- Unidades lógicas propias del campo (Franja: 400 de ancho por 620 de alto), escaladas a la pantalla con proporción fija.
- Física a paso fijo de 1/120 s con integración exacta (`y += v*dt + g*dt²/2`).
- Marcas y ajustes en `localStorage`, siempre con try/catch. Si cambian los valores por defecto, sube `CFGV` para que se apliquen sin borrar las marcas.
- Panel de Ajustes con un deslizador por parámetro físico y botón "Valores iniciales".
- Modos: reto del día (3 intentos, cuenta el mejor) y práctica libre.

## Cómo calibrar un juego con un vídeo

El usuario sube grabaciones de pantalla del juego real. Lo que funcionó:

1. Pedirle grabación propia a pantalla completa, 60 fps, con "Show taps" activado (Xiaomi: Developer options → Input). Los vídeos de YouTube a 30 fps dieron gravedad e impulso erróneos.
2. `ffprobe` para tiempos reales de cada fotograma y `ffmpeg -fps_mode passthrough` volcando fotogramas en crudo a Python (numpy).
3. Localizar los bordes del campo por color, y seguir cada objeto por color fotograma a fotograma.
4. Ajustar una parábola a los tramos de vuelo libre usando el índice de fotograma a 60 Hz (el juego avanza por fotogramas; el grabador mete duplicados).
5. Tabla por obstáculo: tamaño, posición, velocidad y separación, para sacar la progresión.
6. Mirar ampliados los fotogramas del choque para saber la zona de impacto real.
7. Sacar los sonidos del audio (instantes y frecuencia dominante).
8. Validar: reproducir en el juego la misma secuencia y comparar posición a posición con la grabada.

Pedirle intentos "de prueba" baratos: un solo toque y dejar caer, y otro más largo para la progresión. La app tiene modo práctica, pero su recorrido no coincide con el de los intentos reales.

## Estado de Carriles (segunda versión)

Medido en tres grabaciones a 60 fps del modo práctica (ocho intentos cortos, hasta 98 m):

- Velocidad: 14,2 m/s al empezar, subiendo 0,68 m/s cada segundo (67 m a los 4,3 s y 98 m a los 6,0 s). El marcador son metros.
- Carriles a 357 px en pantalla de 1080. Cambio de carril exponencial con constante de 59 ms (cada fotograma ~24,5 % de lo que falta).
  Un segundo toque durante el movimiento cuenta y redirige al instante; así se cruzan dos carriles.
- En práctica el arranque es siempre igual: primer coche en el carril central (choque a los 67 m) y segundo, una furgoneta,
  en el derecho (choque a los 98 m, 1,75 s después).
- Aviso: 1,09 s desde que asoma hasta que toca. Tamaño en pantalla = 0,40/(0,40 + segundos que faltan); altura en pantalla lineal
  con ese tamaño (px reales: y = 1082 + 720·s), tapada por el cambio de rasante en y = 1492.
- Zona de choque: de lado, a menos de ~0,66 carriles del centro del coche (0,60 choca, 0,72 no). En el tiempo, desde el primer contacto
  hasta ~0,13 s después (a 0,10 s todavía choca de lado; a 0,17 s ya se puede entrar en su carril).
- Las rayas de la carretera corren ~4 veces más rápido que se acercan los coches.

Estimado (vídeo de TikTok, no fiable): cuánto crece la aceleración, velocidad máxima, separación entre oleadas a partir de la tercera,
cuándo empiezan las parejas, cuánto se acorta el aviso con la velocidad y en qué carril sale cada coche después del segundo.
Falta una partida larga propia para medirlo. El detector de cambios del marcador cuenta ~12 % de más: usar la marca final y el tiempo.
