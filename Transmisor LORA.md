# Explicación del código

### 1. Inclusión de Bibliotecas

```cpp
#include <TinyGPS++.h>
#include <SoftwareSerial.h>
#include <LoRa.h>
```

- **TinyGPS++**: Esta biblioteca se utiliza para manejar datos provenientes de un módulo GPS, permitiendo la decodificación de mensajes NMEA y la extracción de información útil como coordenadas, altitud, velocidad, etc.
- **SoftwareSerial**: Permite crear un puerto serie en otros pines que no sean los pines RX/TX predeterminados del microcontrolador. Esto es útil para comunicarse con el módulo GPS sin interferir con el puerto serie principal, que se utiliza para la depuración.
- **LoRa**: Esta biblioteca permite la comunicación de largo alcance utilizando tecnología LoRa. Se encarga de la configuración y el envío de datos a través de este protocolo.

### 2. Definición de Pines y Variables Globales

```cpp
static const int RXPin = 12, TXPin = 34;
static const uint32_t GPSBaud = 9600;
String identificador = "L";
TinyGPSPlus gps;
SoftwareSerial ss(TXPin, RXPin);
String missatge;
unsigned long previousMillis = 0;
const long interval = 10000;
```

- **RXPin y TXPin**: Pines definidos para la comunicación con el GPS. RX es para recibir datos, y TX es para enviar datos (aunque en este caso, solo se utiliza RX).
- **GPSBaud**: Velocidad de transmisión del GPS, configurada a 9600 bps.
- **identificador**: Variable que almacena un identificador para el dispositivo, permitiendo diferenciar entre distintos dispositivos si es necesario.
- **gps**: Instancia de la clase `TinyGPSPlus` que se utilizará para manejar la información del GPS.
- **ss**: Objeto `SoftwareSerial` que permite la comunicación en serie con el módulo GPS a través de los pines especificados.
- **missatge**: Cadena de texto que se utilizará para construir el mensaje que se enviará a través de LoRa.
- **previousMillis**: Variable utilizada para almacenar el tiempo de la última actualización, que ayuda a controlar el intervalo de envío de datos.
- **interval**: Define el intervalo de tiempo en milisegundos entre envíos de datos a través de LoRa.

### 3. Configuración Inicial (setup)

```cpp
void setup() {
    Serial.begin(115200);
    ss.begin(GPSBaud);
    ...
    if (!LoRa.begin(868E6)) {
        Serial.println("No se pudo inicializar LoRa.");
        while (1);
    }
    Serial.println("LoRa inicializado.");
}
```

- **Serial.begin(115200)**: Inicializa la comunicación serie a 115200 bps para depuración y visualización de mensajes en el monitor serie.
- **ss.begin(GPSBaud)**: Inicializa el puerto serie `ss` para recibir datos del GPS a 9600 bps.
- **Configuración de LoRa**: Los pines necesarios para LoRa se configuran, y se intenta inicializar la comunicación LoRa a 868 MHz. Si falla, se imprime un mensaje de error y se detiene el programa en un bucle infinito.

### 4. Bucle Principal (loop)

```cpp
void loop() {
    while (ss.available() > 0) {
        gps.encode(ss.read());
        displayInfo();
        sendInfo();
    }
    if (millis() > 5000 && gps.charsProcessed() < 10) {
        Serial.println(F("No GPS detected: check wiring."));
        delay(1000);
    }
}
```

- **Lectura de Datos GPS**: Se verifica si hay datos disponibles en el puerto serie del GPS. Si hay datos, se leen y se procesan a través de `gps.encode(ss.read())`, que decodifica los datos NMEA.
- **Llamadas a Funciones**: Después de procesar los datos, se llaman a `displayInfo()` y `sendInfo()` para mostrar y enviar la información, respectivamente.
- **Verificación de GPS**: Si no se reciben suficientes datos del GPS en 5 segundos, se imprime un mensaje indicando que no se detecta el GPS, lo que puede ser útil para la depuración de conexiones.

### 5. Mostrar Información del GPS (displayInfo)

```cpp
void displayInfo() {
    missatge = identificador + ";";
    ...
    if (gps.location.isValid()) {
        missatge += String(gps.location.lat(), 6) + "," + String(gps.location.lng(), 6);
    }
    ...
}
```

- **Construcción del Mensaje**: La función inicia la construcción del mensaje que se enviará a través de LoRa. Se agrega el identificador del dispositivo al mensaje.
- **Validación de Datos**: Para cada dato del GPS (coordenadas, fecha, hora, altitud, rumbo y velocidad), se verifica si es válido. Si es válido, se agrega al mensaje; si no, se añaden valores predeterminados (por ejemplo, 0.0 para latitud y longitud no válidas).

### 6. Enviar Información (sendInfo)

```cpp
void sendInfo() {
    if (millis() - previousMillis >= interval) {
        previousMillis = millis();
        if (!missatge.startsWith("0.0,0.0;")) {
            Serial.println("Enviant missatge: " + missatge);
            LoRa.beginPacket();
            LoRa.print(missatge);
            LoRa.endPacket();    
        }
    }
}
```

- **Control de Intervalo**: La función utiliza `millis()` para controlar el tiempo transcurrido desde el último envío. Si ha pasado el tiempo definido por `interval`, se actualiza `previousMillis`.
- **Envío de Mensaje**: Antes de enviar, se verifica que el mensaje no contenga coordenadas inválidas (0.0,0.0). Si el mensaje es válido, se envía a través de LoRa, imprimiendo en el monitor serie un mensaje que indica que se está enviando.

### Resumen del Flujo del Programa

1. **Configuración**: Inicializa la comunicación con el GPS y LoRa.
2. **Lectura de Datos**: En el bucle principal, se leen los datos del GPS y se procesan.
3. **Mostrar y Enviar Información**: Se construye un mensaje con los datos GPS válidos y se envía a través de LoRa a intervalos definidos.
4. **Depuración**: Mensajes de error útiles en caso de que no se detecte el GPS.

Este flujo asegura que el dispositivo pueda recibir y transmitir datos de ubicación de manera efectiva, utilizando LoRa para la comunicación a larga distancia, mientras mantiene una buena práctica de manejo de errores y validación de datos.

