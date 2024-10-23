# Detalle del código

### 1. **Bibliotecas Incluidas**

```cpp
#include <LoRa.h>
#include <PubSubClient.h> // Librería para MQTT
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
```

- **LoRa.h**: Esta biblioteca permite la comunicación a través de LoRa (Long Range), una tecnología de red de baja potencia.
- **PubSubClient.h**: Esta biblioteca se utiliza para conectarse a un servidor MQTT y gestionar la publicación y suscripción de mensajes.
- **WiFi.h**: Esta biblioteca permite conectar el ESP32 a una red Wi-Fi.
- **ESPAsyncWebServer.h**: Se utiliza para manejar un servidor web asíncrono, permitiendo que el ESP32 sirva contenido web sin bloquear el resto del programa.

### 2. **Definición de Variables Globales**

```cpp
const char* ssid = "TU_RED_WIFI"; 
const char* password = "TU_CONTRASEÑA_WIFI";  
AsyncWebServer server(80); 
const char* mqtt_server = "10.10.10.10"; //IP del servidor de MQTT

// Pines LoRa
#define LORA_SCK 5
#define LORA_MISO 19
#define LORA_MOSI 27
#define LORA_SS 18
#define LORA_RST 14
#define LORA_DI0 26

// Mensaje recibido y datos a enviar
String incoming = "";
String longitud = "";   
String latitud = "";
String coordenadas = "";
String fecha = "";
String hora = "";
String altitud = "";
String rumbo = "";
String velocidad = "";
String dispositivo = "";
int tiempoReset = 60000; // Tiempo de reset de LORA 60 segundos

WiFiClient espClient;
PubSubClient client(espClient);
const long interval = 10000;  // Intervalo de 10 segundos (10000 ms)
unsigned long previousMillis = 0;  // Almacena el último tiempo en el que se actualizó el LED

unsigned long currentMillis1 = millis();  // Obtiene el tiempo actual
unsigned long previousMillis1 = 0;
```

- **ssid** y **password**: Credenciales de la red Wi-Fi a la que se conectará el ESP32.
- **AsyncWebServer server(80)**: Inicializa un servidor web que escucha en el puerto 80.
- **mqtt_server**: Dirección del servidor MQTT.
- **Pines LoRa**: Definición de los pines del ESP32 que se utilizarán para la comunicación con el módulo LoRa.
- **Variables de datos**: Variables para almacenar datos recibidos y enviados, como coordenadas, fecha, hora, etc.
- **tiempoReset**: Tiempo en milisegundos antes de reiniciar el módulo LoRa si no se reciben datos.
- **WiFiClient** y **PubSubClient**: Instancias para la conexión a Wi-Fi y a MQTT, respectivamente.
- **interval**, **previousMillis**, **currentMillis1**, y **previousMillis1**: Variables para gestionar el tiempo y los intervalos.

### 3. **Función `setup()`**

```cpp
void setup() {
  // Configura el monitor serie
  Serial.begin(115200);

  // Configura LoRa
  LoRa.setPins(LORA_SS, LORA_RST, LORA_DI0);
  if (!LoRa.begin(868E6)) {  // Frecuencia 868 MHz
    Serial.println("No se pudo inicializar LoRa.");
    while (1);
  }
  Serial.println("LoRa inicializado.");

  // Connect to Wi-Fi 
  WiFi.begin(ssid, password); 
  while (WiFi.status() != WL_CONNECTED) { 
    delay(1000); 
    Serial.println("Connecting to WiFi..."); 
  } 
  Serial.println("Connected to WiFi"); 
  Serial.print("ESP32 Web Server's IP address: "); 
  Serial.println(WiFi.localIP()); 

  // Define a route to serve the HTML page
  server.on("/", HTTP_GET, [](AsyncWebServerRequest* request) {
    Serial.println("ESP32 Web Server: New request received:");  // for debugging

    String html = "<!DOCTYPE HTML>";
    html += "<html>";
    html += "<head>";
    html += "<link rel=\"icon\" href=\"data:,\">";
    html += "</head>";
    html += "<p>";
    html += "Coordenades rebudes: <span style=\"color: red;\">";
    html += incoming;
    html += "</span>";
    html += "</p>";
    html +=  "<a href='https://www.google.com/maps?q=" + latitud + "," + longitud +"'>Coordenades</a>";
    html += "</html>";

    request->send(200, "text/html", html);
  });

  // Start the server
  server.begin();  

  // Conexión al servidor MQTT
  client.setServer(mqtt_server, 1883);

  // Conectar a MQTT
  reconnectMQTT();
}
```

- **Serial.begin(115200)**: Inicia el monitor serie para la depuración.
- **LoRa.setPins(...)**: Configura los pines del módulo LoRa.
- **LoRa.begin(868E6)**: Inicializa el módulo LoRa en la frecuencia de 868 MHz.
- **WiFi.begin(...)**: Inicia la conexión Wi-Fi.
- **while (WiFi.status() != WL_CONNECTED)**: Espera hasta que el ESP32 esté conectado a la red Wi-Fi.
- **server.on(...)**: Define una ruta en el servidor web que responderá a las solicitudes GET en la raíz (`/`). La respuesta es una página HTML que muestra las coordenadas recibidas.
- **server.begin()**: Inicia el servidor web.
- **client.setServer(...)**: Configura el cliente MQTT para conectarse al servidor especificado.
- **reconnectMQTT()**: Intenta establecer la conexión con el servidor MQTT.

### 4. **Función `loop()`**

```cpp
void loop() {
  // Verificar si hay un paquete LoRa disponible
  int packetSize = LoRa.parsePacket();
  if (packetSize) {
    // Recibir datos del paquete
    incoming = "";
    bool salir = false; 
    while (LoRa.available()) {
      incoming += (char)LoRa.read();
    }
    // Mostrar el mensaje recibido
    Serial.println(incoming);

    //Extraemos las coordenadas del paquete recibido por LORA
    extraeCoordenadas();

    // Enviamos las coordenadas al servidor MQTT
    enviaDatosMQTT();
    
  }
  else {
    // Si no recibimos nada en LORA
    // Comprueba si han pasado más de tiempoReset y si es así, reiniciamos LORA
    currentMillis1 = millis();
    if (currentMillis1 - previousMillis1 >= tiempoReset) {  
      previousMillis1 = currentMillis1;      
      Serial.println("Sense rebre dades durant 60 segons, reiniciam lora");
      incoming = "Sense rebre dades durant 60 segons, reiniciam lora";
      LoRa.end();  // Parar LoRa
      delay(1000); // Esperar un segundo antes de reiniciar
      if (!LoRa.begin(868E6)) {
        Serial.println("Error al reiniciar LoRa");
        incoming += "Error al reiniciar LoRa";
      } else {
        Serial.println("LoRa reiniciado correctamente");
        incoming += "LoRa reiniciado correctamente";
        reconnectMQTT();
      }
    }   
  }   
}
```

- **LoRa.parsePacket()**: Verifica si hay un paquete disponible para leer. Si hay uno, lo procesa.
- **incoming += (char)LoRa.read()**: Lee el contenido del paquete recibido y lo almacena en la variable `incoming`.
- **extraeCoordenadas()**: Llama a una función que procesa `incoming` para extraer coordenadas y otros datos.
- **enviaDatosMQTT()**: Llama a la función que envía los datos extraídos al servidor MQTT.
- **else**: Si no hay paquete LoRa disponible, verifica si ha pasado el tiempo de espera definido por `tiempoReset`. Si es así, reinicia el módulo LoRa.

### 5. **Funciones Auxiliares**

#### `extraeCoordenadas()`

Esta función se encarga de analizar la cadena `incoming` y extraer los datos relevantes. Utiliza separadores para dividir la cadena y almacenar los datos en las variables correspondientes.

#### `enviaDatosMQTT()`

Envía los datos extraídos al servidor MQTT. Comprueba si el cliente MQTT está conectado y, si no, llama a `reconnectMQTT()` para intentar reconectar. Envía un mensaje JSON con los datos de ubicación y otros detalles.

#### `reconnectMQTT()`

Intenta reconectar al servidor MQTT si la conexión se ha perdido. Imprime el estado de la conexión en el monitor serie y espera 2 segundos entre intentos de reconexión.
