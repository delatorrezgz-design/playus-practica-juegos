# Práctica de juegos

Minijuegos web de práctica que imitan la mecánica (no el aspecto ni el nombre) de los juegos diarios de una app móvil. El usuario no es programador: explícale las cosas en español y sin jerga.

## Estructura

- `index.html`: menú con un enlace por juego.
- `franja/index.html`: "Franja" (tocar para subir y pasar por huecos entre columnas).
- `carriles/index.html`: "Carriles" (3 carriles, tocar izquierda/derecha, esquivar coches; el real se llama Lane Rush).
- `diana/index.html`: "Diana" (lanzar dardos a una diana que gira sin dar a los ya clavados; el real se llama Knife Throw).
- `justo/index.html`: "Justo" (tocar para arrancar un cronómetro y parar lo más cerca de 7 s sin pasarse; el real se llama On time).
- Cada juego es una página única, sin dependencias ni compilación, con el mismo estilo visual (el CSS está copiado en cada página).
- Se publica con GitHub Pages desde `main`, carpeta raíz: https://delatorrezgz-design.github.io/playus-practica-juegos/

## Convenciones de los juegos

- Unidades lógicas propias del campo (Franja: 400 de ancho por 620 de alto), escaladas a la pantalla con proporción fija.
- Física a paso fijo de 1/120 s con integración exacta (`y += v*dt + g*dt²/2`).
- Marcas y ajustes en `localStorage`, siempre con try/catch. Si cambian los valores por defecto, sube `CFGV` para que se apliquen sin borrar las marcas.
- Panel de Ajustes con un deslizador por parámetro físico y botón "Valores iniciales".
- Franja: reto del día (3 intentos, cuenta el mejor) y práctica libre. Carriles, Diana y Justo: solo práctica, con récord, media de las últimas 10 partidas, barritas de las 10 últimas y botón "Reiniciar estadísticas" con confirmación (el usuario lo pidió así).

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

## Estado de Diana (cuarta versión: YouTube + tres grabaciones propias de práctica a 60 fps, dos con choque, una de 37 puntos)

Mecánica: la diana gira; tocar lanza un dardo recto hacia arriba. Queda clavado (y gira con la diana) 150 ms después del toque;
en ese mismo instante choca (fin, la diana se para en seco) si hay otro dardo a menos de `gap` grados, o coge las monedas del
borde que estén a menos de `coinw` (+2 cada una; en el real, manzanas). Al acabar los dardos de la ronda salen despedidos y,
tras 1,05 s, empieza otra con un dardo más, más giro y algún dardo ya clavado.
`diana/index.html#demo` juega solo. `Diana.replay({angle:[[t,grados]...], rounds:[{pre:[..],coins:[..]}], taps:[t...]})`
repite una partida grabada (giro medido, dardos previos y toques) y devuelve puntos y ángulo de cada clavada.

Medido en grabaciones propias (práctica, 60 fps, toques visibles):
- Del punto de "Mostrar toques" a que sale el cuchillo: 3–5 fotogramas (45–85 ms); de ahí a clavado y girando, 80–105 ms.
  Del toque a clavado: mediana 150 ms (27 toques, 99–186). Esa variación es del juego real: a 270 °/s son ±8°.
- Choque: chocó a ~10° y a ~5°; no chocaron parejas a 15,2°, 16,3° y 17,1°. Zona puesta en 12,5°.
- Manzana: se corta al quedar clavado; contó a 4–14° de su centro (7 manzanas). Zona puesta en 15°. Suma +2.
- Giro: cada 2,0–2,5 s elige una velocidad objetivo y se acerca a ella exponencialmente con τ = 1,0 s (error de 2 °/s).
  El primer cambio llega ~2,2 s después de empezar la ronda; al empezar ronda salta de golpe a su velocidad, en sentido horario.
  Inicio de ronda: ronda 1 entre 140 y 163 (variable); ronda 2: 160 (en las cuatro partidas); 3: 178–180; 4: 197–200; 5: 212.
  Objetivos entre 0,9 y 1,45 veces el inicio de la ronda, en el sentido actual.
- Cambio de sentido: nunca en la ronda 1 (0 de ~12 cambios); desde la 2, en 6 de ~27 cambios (22 %). Puede seguir al revés
  varios cambios: en la ronda 4 de la partida larga estuvo 10 s al revés (−285, −197, −265, −250) y volvió a +255.
- Rondas de 5, 6, 7, 8, 9 cuchillos; ya clavados al empezar 0, 1, 2, 2, 3 (separados ≥ 33°).
- Manzanas: ronda 1 siempre una (4 de 4); después 0, 1 o 2 (en 9 rondas: 5 sin, 3 con una, 1 con dos).
- Toques a 80–110 ms entre sí lanzan todos. No hay límite de tiempo.
- Repeticiones: práctica 5 puntos; choque 7 (manzana en el 1.º, choque en el 1.º de la ronda 2); YouTube 26; partida larga 37
  (4 manzanas, choque en el 4.º de la ronda 5) usando el tiempo de reacción típico. Con los toques tal cual, la larga coincide
  hasta el cuchillo 23: luego la variación del propio juego (126 frente a 179 ms) mueve dos cuchillos ~6° y provoca un choque.

Cómo se midió: centro del tronco por ajuste de círculo al borde de ladrillos (1080×2414: 539,5; 1067; radio 268 px; 720×1610:
359,7; 711,3; radio 178,5); desenrollado polar (`cv2.remap`) y correlación circular de la cara del tronco entre fotogramas para
el giro, repartiendo los saltos dobles del grabador entre fotogramas de 60 Hz; objetivo de velocidad = v + τ·v'. Mangos (marrón,
R−G > 45) en un anillo fuera del borde, en coordenadas del tronco: ángulo de cada cuchillo; el instante en que se clava sale de
cuándo ese ángulo pasa por abajo. Manzanas: rojo intenso (R > 190, G < 40) en el anillo del borde. Al chocar, el giro medido
se vuelve 0. Toques: disco claro pequeño que aparece de golpe a la derecha de la fila de cuchillos; comprobarlos ampliando.
Vídeos de más de 30 MB no se pueden subir: grabar a 720p con calidad baja y 60 fps (43 s ocupan 23 MB), o trocear.

## Estado de Justo (segunda versión: Short de YouTube + un intento de práctica propio a 60 fps y dos partidas grabadas en Justo)

Mecánica: un toque pone en marcha el cronómetro (dos decimales) y el anillo; otro toque lo para. El resultado sale de la hora
exacta de cada toque (`event.timeStamp`), no de los fotogramas, y se enseña con cuatro decimales. Si llega a 7 sin tocar, o el
segundo toque es después de 7, "te has pasado" (se guarda como −1). Puntos = lo que faltó (7 − resultado); menos es mejor; la
tarjeta dice también lo que puntuaría en el real (redondeado a dos decimales). Récord = lo que menos faltó; la media de las 10
últimas no cuenta las pasadas. `Justo.replay({taps:[inicio, parada]})` (en s) devuelve resultado exacto, lo que faltó y el marcador.

El usuario solo tiene un intento de práctica al día (no se puede repetir) y los reales no los graba: no pedir pruebas de fallar.
Lo que se pueda medir, pedirlo en el intento de práctica (jugando normal) y en grabaciones de Justo, que no gastan nada.

Medido:
- Cronómetro a velocidad de reloj: 0,99996 (práctica 60 fps) y 1,0005 ± 0,0004 (Short). Cambia en cada imagen de 60 Hz.
- Anillo: una vuelta en 7 s, lineal, empieza arriba, sentido del reloj; va con el número. En el real se pone rojo al parar
  lejos (0,1781) y se queda verde muy cerca (0,0024); el umbral no se sabe y Justo no lo imita.
- Arranca al apoyar el dedo: con el dedo apoyado 1,1 s, el número empezó a contar 2 imágenes después de aparecer el punto.
- Marcador y puntuación redondeados a dos decimales: 6,9976 → "7.00"; 6,8219 → "6.82"; falta 0,1781 → "YOU SCORED 0,18".
  Tras parar enseña "OFF BY 0.1781s" (en español "POR 0.0024s"; con 7.00 en pantalla añade "Resultado exacto: 6.9976").
- El resultado es el tiempo entre los dos puntos de "Mostrar toques": práctica 6,8407 frente a 6,8219 (19 ms, una imagen);
  Justo 6,9914 frente a 6,9801 y 6,9407 frente a 6,9403. El vídeo no da más precisión que una imagen (~16 ms) por toque.
- Retraso de los números (del punto del toque a que el número "empieza" en la recta ajustada): real 39 ms al arrancar y 22 ms
  al parar; Justo 20 y 17 ms al arrancar, 9 y 15 ms al parar (Chrome del mismo Xiaomi). El real va ~15 ms más retrasado:
  ajuste "Retraso de los números" = 15 ms (CFGV 2). El real congela el número en la misma imagen del toque; Justo, 1–2 después.
- Sonido: el audio de la práctica son voces de fondo; no hay tic durante la cuenta ni sonido al arrancar; hay un golpe corto
  ~0,1 s después de parar (y al pulsar "Start"). Justo ya no suena al arrancar. El grabador sí recoge el sonido del juego (se
  oyen los pitidos de 990 Hz de Justo al parar).
- Pantallas: "Tap to start, tap again at 7s" con un disco gris que late (adorno, no es el toque); durante la cuenta "TAP AT 7".
  Tras parar, ~1,4–1,75 s hasta la pantalla final.

Marcador con menos de 2 decimales (ajuste para entrenar el ritmo): corta en vez de redondear; redondeando, el "7" salía a los 6,5 s.

Sin medir: qué pasa al llegar a 7 sin tocar o al tocar después (Justo lo da como "te has pasado"; el texto del juego dice "Si
esperas demasiado, la ronda termina").

Cómo se midió: anillo muestreado en un círculo (centro 239,5; 563,5; radio 161 en 480×1072) por color; números con plantillas
por casillas fijas entrenadas con el valor que da el anillo (`ocr2.py` de la sesión); el punto de "Mostrar toques" es un disco
claro pequeño que aparece de golpe (diferencia entre imágenes); recta valor = tiempo − cero por mitades, porque los tiempos del
grabador derivan ~15 ms.
