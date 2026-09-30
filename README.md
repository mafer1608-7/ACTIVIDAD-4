# Interfaz de gestos para control mediante ESP32

Aplicación desarrollada en Python que utiliza la cámara del computador para reconocer movimientos de la mano y transformarlos en comandos de control para un ESP32.

El reconocimiento se realiza mediante **MediaPipe Gesture Recognizer**, mientras que **OpenCV** se encarga de obtener las imágenes de la cámara. Una vez identificado el gesto, el programa establece una comunicación serial con el ESP32 y transmite el comando correspondiente.

La aplicación está diseñada para trabajar de manera continua y mostrar en pantalla el gesto reconocido junto con su nivel de confianza.

## Descripción del sistema

El funcionamiento del proyecto se divide en tres etapas principales:

```text
        CAPTURA
           │
           ▼
      Cámara web
           │
           ▼
        OpenCV
           │
           ▼
    MediaPipe Tasks
           │
           ▼
     Gesto detectado
           │
           ▼
     Código asociado
           │
           ▼
     Puerto serial
           │
           ▼
         ESP32
```

La cámara proporciona los fotogramas que serán analizados. MediaPipe determina qué gesto está realizando la persona y Python relaciona ese resultado con un carácter determinado.

## Gestos reconocidos

El programa trabaja con cinco gestos:

| Gesto | Carácter transmitido |
|---|---|
| `Closed_Fist` | `F` |
| `Victory` | `V` |
| `Open_Palm` | `P` |
| `Thumb_Up` | `U` |
| `Thumb_Down` | `D` |

La relación entre los gestos y los comandos se establece en Python mediante el diccionario:

```python
GESTURE_TO_COMMAND = {
    "Closed_Fist": 'F',
    "Victory": 'V',
    "Open_Palm": 'P',
    "Thumb_Up": 'U',
    "Thumb_Down": 'D',
}
```

Los caracteres enviados son interpretados posteriormente por el programa que se encuentra en el ESP32.

## Reconocimiento de la mano

Para realizar la identificación se utiliza el archivo:

```text
gesture_recognizer.task
```

El modelo es cargado mediante MediaPipe Tasks:

```python
options = GestureRecognizerOptions(
    base_options=BaseOptions(
        model_asset_path="gesture_recognizer.task"
    ),
    running_mode=VisionRunningMode.VIDEO,
    num_hands=1
)
```

El sistema está configurado para reconocer una mano frente a la cámara.

Cada fotograma obtenido por OpenCV se transforma de formato BGR a RGB antes de ser procesado por MediaPipe.

## Flujo de procesamiento

El programa realiza las siguientes operaciones:

1. Establece la conexión serial con el ESP32.
2. Espera unos segundos para permitir el inicio de la placa.
3. Carga el modelo de reconocimiento de gestos.
4. Inicializa la cámara del computador.
5. Captura continuamente los fotogramas.
6. Convierte cada imagen al formato requerido por MediaPipe.
7. Genera una marca de tiempo para cada fotograma.
8. Ejecuta el reconocimiento del gesto.
9. Obtiene el gesto con mayor puntuación.
10. Comprueba si el resultado corresponde a uno de los gestos definidos.
11. Muestra el gesto y su porcentaje de confianza sobre la imagen.
12. Convierte el gesto en un comando.
13. Envía el comando al ESP32 cuando existe un cambio de gesto.

El programa utiliza un mecanismo para evitar enviar continuamente el mismo carácter:

```python
last_command_sent = None
```

De esta manera, si la persona mantiene el mismo gesto durante varios fotogramas, no se transmite nuevamente el comando hasta que se detecte otro gesto.

## Comunicación con el ESP32

La conexión serial se realiza utilizando:

```text
Puerto: COM5
Velocidad: 115200 baudios
```

En el programa:

```python
esp32 = serial.Serial(
    'COM5',
    115200,
    timeout=1
)
```

Si el ESP32 se encuentra en otro puerto, solamente es necesario modificar el valor correspondiente.

Por ejemplo:

```python
esp32 = serial.Serial(
    'COM6',
    115200,
    timeout=1
)
```

Los comandos se transmiten como caracteres individuales.

Por ejemplo:

```text
F
```

o:

```text
U
```

El envío se realiza mediante:

```python
esp32.write(command.encode('utf-8'))
```

## Requisitos

### Componentes físicos

- ESP32
- Computador
- Cámara web
- LEDs y elementos de conexión utilizados por el sistema de salida
- Cable USB para comunicación con el ESP32

### Herramientas de software

- Python 3.9 a 3.12
- Visual Studio Code
- PlatformIO
- OpenCV
- MediaPipe
- PySerial
- Arduino Framework

## Organización de archivos

Una organización recomendada para el proyecto es:

```text
.
├── docs/
│   ├── MINIATURA.png
│   ├── ACTIVIDAD4_VIDEO.mp4
│   ├── reconocimiento_gestos.png
│   └── montaje.png
│
├── python/
│   ├── ACTIVIDAD4.py
│   └── gesture_recognizer.task
│
├── esp32/
│   ├── platformio.ini
│   └── src/
│       └── main.cpp
│
└── README.md
```

El archivo `ACTIVIDAD4.py` contiene el procesamiento de la cámara, reconocimiento de gestos y comunicación serial.

El archivo `gesture_recognizer.task` corresponde al modelo utilizado por MediaPipe.

## Preparación del entorno

### Python

Desde una terminal de VS Code se pueden instalar las bibliotecas necesarias mediante:

```bash
pip install opencv-python mediapipe pyserial
```

Después de la instalación se puede comprobar que Python reconoce las bibliotecas utilizadas en el programa.

## Modelo de reconocimiento

El archivo:

```text
gesture_recognizer.task
```

debe estar disponible para que el programa pueda iniciar el reconocedor.

El código busca el modelo mediante:

```python
model_asset_path="gesture_recognizer.task"
```

Por esta razón, la ubicación del archivo debe coincidir con la ruta utilizada por el programa.

## Puesta en funcionamiento

### Paso 1. Conectar el ESP32

Conectar el ESP32 al computador mediante el cable USB.

Comprobar en el sistema operativo qué puerto COM fue asignado.

### Paso 2. Cargar el programa del ESP32

Abrir el proyecto correspondiente en Visual Studio Code mediante PlatformIO.

Compilar y cargar el firmware en la placa.

### Paso 3. Liberar el puerto serial

Antes de ejecutar el programa Python, cerrar el monitor serial de PlatformIO.

Esto evita que dos aplicaciones intenten utilizar simultáneamente el mismo puerto.

### Paso 4. Ejecutar la aplicación

Ubicarse en la carpeta donde está `ACTIVIDAD4.py` y ejecutar:

```bash
python ACTIVIDAD4.py
```

La cámara se abrirá y comenzará el reconocimiento.

### Paso 5. Realizar los gestos

Colocar la mano frente a la cámara y realizar alguno de los gestos configurados.

La ventana mostrará información similar a:

```text
Gesto: Victory (95.4%)
```

El porcentaje corresponde a la puntuación proporcionada por el reconocedor.

## Código principal

El archivo utilizado para esta actividad es:

```text
ACTIVIDAD4.py
```

El programa comienza importando las bibliotecas necesarias:

```python
import cv2
import mediapipe as mp
import time
import serial
```

Posteriormente establece la comunicación con el ESP32 y configura el reconocedor de gestos.

La cámara se inicializa mediante:

```python
cap = cv2.VideoCapture(0)
```

El reconocimiento se ejecuta sobre cada fotograma:

```python
result = recognizer.recognize_for_video(
    mp_image,
    timestamp_ms
)
```

Finalmente, cuando se identifica un gesto válido, se obtiene su comando y se transmite mediante el puerto serial.

## Visualización del resultado

El programa muestra el gesto reconocido directamente sobre la imagen:

```python
cv2.putText(
    frame,
    f"Gesto: {gesture_name} ({score:.1%})",
    (20, 50),
    cv2.FONT_HERSHEY_SIMPLEX,
    1,
    (0, 255, 0),
    2
)
```

Esto permite observar simultáneamente la imagen de la cámara, el gesto identificado y la confianza obtenida.

## Finalización del programa

Para terminar la ejecución se puede presionar:

```text
q
```

El programa libera posteriormente la cámara y cierra la conexión serial.

Las operaciones de cierre utilizadas son:

```python
cap.release()
cv2.destroyAllWindows()

if esp32 is not None:
    esp32.close()
```

## Situaciones frecuentes

| Situación | Verificación |
|---|---|
| No se puede abrir COM5 | Comprobar que el ESP32 esté conectado y que el puerto sea correcto. |
| Puerto ocupado | Cerrar el monitor serial de PlatformIO. |
| No aparece la cámara | Revisar la conexión de la cámara y el índice utilizado por `VideoCapture()`. |
| No se reconoce la mano | Mejorar la iluminación y mantener la mano dentro del campo de visión. |
| Falta `gesture_recognizer.task` | Colocar el modelo en la ubicación indicada por `model_asset_path`. |
| No se envían comandos | Comprobar la conexión serial y verificar que el gesto reconocido pertenezca a `TARGET_GESTURES`. |
| El programa no encuentra una biblioteca | Instalar las dependencias mediante `pip`. |

## Posibles modificaciones

El proyecto puede ampliarse modificando diferentes elementos del programa.

### Cambiar los gestos

Los gestos utilizados se encuentran en:

```python
TARGET_GESTURES = [
    "Closed_Fist",
    "Victory",
    "Open_Palm",
    "Thumb_Up",
    "Thumb_Down"
]
```

### Cambiar los comandos

La relación entre gesto y comando se modifica en:

```python
GESTURE_TO_COMMAND
```

### Cambiar la cámara

Actualmente se utiliza:

```python
cv2.VideoCapture(0)
```

Si existen varias cámaras, se puede probar con:

```python
cv2.VideoCapture(1)
```

o:

```python
cv2.VideoCapture(2)
```

## Tecnologías empleadas

- Python
- OpenCV
- MediaPipe Tasks
- Gesture Recognizer
- PySerial
- ESP32
- PlatformIO
- Arduino Framework
- Comunicación serial

## Resultado

La actividad integra visión artificial y comunicación con hardware. La cámara permite obtener información visual de la mano, MediaPipe identifica el gesto realizado y Python transforma ese resultado en un comando que puede ser interpretado por el ESP32.

El sistema establece así una comunicación entre el reconocimiento de gestos realizado en el computador y el dispositivo físico conectado mediante el puerto serial.

## Autor

Desarrollado por <Mafe Peñuela> — [@mafer1608-7](https://github.com/mafer1608-7)
