### Paso 1: Generar un certificado CA (Autoridad Certificadora)

1. **Instalar OpenSSL**:
   - Asegúrate de tener OpenSSL instalado en tu PC. Puedes descargarlo de [OpenSSL](https://www.openssl.org/).

2. **Crear la CA**:
   - Abre la terminal (o símbolo del sistema) y ejecuta:
     ```bash
     openssl genrsa -out ca.key 2048
     openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt
     ```
   - Durante este proceso, te pedirá información para el certificado. Puedes usar valores predeterminados o rellenar según sea necesario.

### Paso 2: Crear un certificado para el servidor MQTT

1. **Generar una clave privada para el servidor**:
   ```bash
   openssl genrsa -out mqtt.key 2048
   ```

2. **Crear una solicitud de firma de certificado (CSR)**:
   ```bash
   openssl req -new -key mqtt.key -out mqtt.csr
   ```

3. **Firmar el certificado con la CA**:
   ```bash
   openssl x509 -req -in mqtt.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mqtt.crt -days 365 -sha256
   ```

### Paso 3: Configurar el servidor Mosquitto para usar SSL/TLS

1. **Mover los certificados al contenedor Docker**:
   - Copia `ca.crt`, `mqtt.crt`, y `mqtt.key` a una ubicación en tu sistema que pueda ser accesible desde Docker. Por ejemplo, `/home/tu_usuario/mosquitto/certs/`.

2. **Modificar la configuración de Mosquitto**:
   - Abre o crea el archivo de configuración de Mosquitto (por ejemplo, `mosquitto.conf`).
   - Agrega las siguientes líneas:
     ```bash
     listener 8883
     allow_anonymous false
     cafile /mosquitto/certs/ca.crt
     certfile /mosquitto/certs/mqtt.crt
     keyfile /mosquitto/certs/mqtt.key
     ```

3. **Ejecutar el contenedor Mosquitto**:
   - Asegúrate de que el Docker esté configurado para usar el volumen donde has copiado los certificados. Puedes agregar una línea a tu `docker-compose.yml` para que el contenedor acceda a esa ruta y que el puerto sea el 8883
   ```yaml
   ports:
    - '1883:1883'  # Puerto MQTT
    - '8883:8883'  # Puerto seguro MQTT
   volumes:
     - /home/tu_usuario/mosquitto/certs:/mosquitto/certs
   ```

### Paso 4: Configurar el ESP32 para usar el certificado

1. **Incluir el certificado CA en el código del ESP32**:
   - Abre tu proyecto en Arduino IDE y define el certificado:
     ```cpp
     const char* ca_cert = \
       "-----BEGIN CERTIFICATE-----\n" \
       "Contenido del certificado CA\n" \
       "-----END CERTIFICATE-----\n";
     ```

2. **Configurar el cliente MQTT para usar SSL/TLS**:
   - En la parte del código donde configuras el cliente MQTT, agrega el certificado CA:
     ```cpp
     espClient.setCACert(ca_cert);
     client.setServer(mqtt_server, 8883); // Asegúrate de usar el puerto 8883 para SSL
     ```

### Resumen

- **Genera los certificados** en tu PC utilizando OpenSSL.
- **Configura Mosquitto** para usar esos certificados modificando su archivo de configuración.
- **Carga el certificado CA en el ESP32** y configura el cliente MQTT para usar SSL/TLS.

Con estos pasos, podrás asegurar la conexión entre tu ESP32 y el servidor Mosquitto utilizando certificados SSL/TLS.
