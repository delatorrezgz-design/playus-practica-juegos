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
- Franja: reto del día (3 intentos, cuenta el mejor) y práctica libre. Carriles: solo práctica, con récord, media de las últimas 10 partidas y botón "Reiniciar estadísticas" (el usuario lo pidió así).

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

## Estado de Carriles (cuarta versión, calibrada con una partida real de 438 m)

El juego funciona en metros: los coches están colocados en la carretera y se acercan a la velocidad del jugador.
`carriles/index.html#demo` activa un piloto automático para probar partidas largas (escribe la marca en la consola).

Medido en grabaciones propias a 60 fps (ocho intentos cortos de práctica y una partida de 438 m en 19,55 s):

- Velocidad: v = 14,1 + 0,70·t + 0,0117·t² m/s (ajuste al marcador leído metro a metro, error típico 0,2 m).
  A los 19 s va a 31,5 m/s. El marcador son metros.
- Carriles a 357 px en pantalla de 1080. Cambio de carril exponencial con constante de 59 ms (cada fotograma ~24,5 % de lo que falta).
  Un segundo toque durante el movimiento cuenta y redirige al instante; así se cruzan dos carriles.
- El arranque es siempre igual (práctica y partida larga): primer coche en el carril central (choque a los 67 m) y segundo,
  una furgoneta, en el derecho (choque a los 98 m). Después el carril es al azar y no depende de dónde esté el jugador.
- Metros a los que llegó cada oleada en la partida larga: 67, 98, 124, 148, 176, 200, 228 (pareja), 252, 272 (pareja), 297,
  319 (pareja izquierda+derecha), 338, 361, 378, 400, 420, 438 (pareja). Separación ≈ 30,5 − 2,7 por cada 100 m, ± 3 m al azar.
- Parejas: 4 de las 11 oleadas a partir de los 228 m; pueden ser carriles no contiguos y a veces un coche va ~2 m por detrás.
- Los coches se acercan a la velocidad del marcador (el tiempo entre dos líneas de la pantalla baja al subir la velocidad).
  Se ven desde ~18 m antes. Tamaño en pantalla ≈ 7,5/(7,5 + metros que faltan); altura lineal con ese tamaño
  (px reales: y = 1082 + 720·s), tapada por el cambio de rasante en y = 1492.
- Zona de choque: de lado, a menos de ~0,66 carriles del centro del coche (0,60 choca, 0,72 no). A lo largo, desde el primer contacto
  hasta ~2,3 m después.
- Vehículos: coche naranja, furgoneta blanca (más alta), ranchera azul, coche beige. Todos ocupan un carril y miden parecido.
- Las rayas de la carretera corren ~4 veces más rápido que se acercan los coches.

Sin medir (sale de un vídeo de TikTok de 923 m, acelerado ~×1,3 desde los 95 m): separación mínima de ~15 m entre oleadas
hacia los 850 m, la mitad de oleadas en pareja al final, y una posible velocidad máxima hacia 40 m/s.

Leer el marcador: umbral de blanco (canal mínimo > 210), separar dígitos por columnas y compararlos con plantillas
(`ocr.py` de la sesión); da el metro exacto de cada fotograma.
