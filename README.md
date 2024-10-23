![top](assets/astromoon2.jpeg)
# README - SONDA METEREOLÓGICA
---
Este documento describe cómo llevar a cabo el lanzamiento de una sonda meteorológica de bajo costo utilizando un globo de helio, con el fin de registrar y monitorizar en tiempo real variables como la posición, temperatura y presión atmosférica, además de grabar imágenes durante su ascenso y descenso. El sistema incluye el uso de tecnologías como LoRa y MQTT para la transmisión de datos, y ofrece dos métodos de seguimiento: uno mediante un servidor en la nube con visualización en Grafana y otro a través del sistema de seguimiento de WhatsApp. Se detalla la infraestructura necesaria, incluyendo la configuración de servidores en máquinas virtuales con Ubuntu Server 24.04.

---
**APRENDERÁS:**

- **Arduino IDE**: Programación del TBeam 1.2 con GPS y LoRa.
- **LoRa**: Configuración y transmisión de datos a largas distancias.
- **MQTT**: Envío y recepción de datos en tiempo real entre la sonda y el servidor.
- **Comunicaciones serie**: Interacción con sensores y módulos desde el TBeam.
- **Docker Compose**: Despliegue de contenedores para **Mosquitto**, **Telegraf**, **InfluxDB**, **Grafana** y **NGINX**.
- **InfluxDB**: Almacenamiento de datos atmosféricos y de posición.
- **Grafana**: Visualización de la trayectoria de la sonda con **GEOMAP**.
- **Certificados SSL (Let's Encrypt)**: Configuración de seguridad para el acceso al servidor.
- **WhatsApp Tracking**: Uso del móvil en la sonda para seguimiento adicional.
- **Ubuntu Server 24.04**: Configuración y gestión de máquinas virtuales para los servidores.

---
## Visión general

El lanzamiento de una sonda meteorológica utilizando un globo de helio es un proyecto que, aunque en otro tiempo habría sido inalcanzable para muchos, hoy se encuentra al alcance de todos gracias a los avances tecnológicos y la accesibilidad de herramientas de bajo costo. En el contexto actual, la proliferación de dispositivos como Arduino, ESP8266, ESP32 o chips GPS, el auge de tecnologías como LoRa y MQTT, y la disponibilidad de plataformas de visualización de datos en tiempo real como Grafana, hacen que proyectos de este tipo sean no solo viables, sino también altamente educativos y enriquecedores.

El costo de los componentes involucrados, desde el globo de helio hasta los módulos GPS y las cámaras, es relativamente asequible incluso para proyectos académicos o experimentales a nivel personal o en pequeños grupos. Además, la infraestructura necesaria para gestionar los datos, como los servidores Docker con Mosquitto, Telegraf e InfluxDB, es fácilmente escalable y se puede montar en máquinas virtuales con recursos limitados, haciendo que los costos de implementación y mantenimiento sean mínimos.

Este experimento no solo permite recolectar datos atmosféricos valiosos —como la temperatura y la presión a gran altitud— sino que también brinda la oportunidad de aplicar conceptos de comunicación a larga distancia mediante LoRa, una tecnología con un amplio campo de aplicación en la monitorización remota. Además, la integración de plataformas de visualización en tiempo real como Grafana permite una comprensión profunda de los datos, tanto para fines científicos como educativos.

Se ha optado por utilizar **LoRa** en lugar de **LoRaWAN** para la transmisión de datos debido a la simplicidad del proyecto, ya que solo están involucrados dos dispositivos: el transmisor en la sonda y el receptor en tierra. La implementación de LoRa proporciona una conexión directa y eficiente entre ambos sin necesidad de una infraestructura adicional, como sería el caso de LoRaWAN. La posibilidad de encontrar un gateway LoRaWAN en las áreas de lanzamiento o aterrizaje es baja o prácticamente nula, lo que refuerza la elección de esta tecnología como la más adecuada para asegurar una comunicación continua y confiable durante todo el vuelo.

Los participantes que realizan este proyecto no solo aprenden sobre las condiciones atmosféricas a gran altitud, sino que también adquieren habilidades críticas en la integración de tecnologías de hardware y software, tales como el uso de Arduino IDE, la configuración de servidores en la nube, la transmisión de datos mediante redes LoRa y la seguridad de las comunicaciones.

Este tipo de experimentos también se sitúa en la intersección entre la ciencia ciudadana y la investigación avanzada. Los resultados pueden ser aplicables a estudios en meteorología, telecomunicaciones, o incluso en logística, y la experiencia práctica ayuda a comprender mejor el uso de sistemas distribuidos y autónomos para la recolección de datos. En resumen, tenemos hoy las herramientas y la tecnología adecuada para explorar el espacio cercano, hacer ciencia práctica a bajo costo y, en el proceso, ganar un profundo conocimiento de múltiples disciplinas tecnológicas.

---
## Esquema del proyecto

**1. Diseño de la sonda:**
La sonda está equipada con:
- **Cámaras** para grabar videos durante la misión.
- **[[TBeam 1.2]] con GPS y LoRa para enviar la ubicación y datos meteorológicos en tiempo real.
- **Móvil**, que transmite la ubicación de la sonda de dos formas:
  - A través de **MQTT** hacia un servidor para su procesamiento y visualización.
  - Utilizando el sistema de seguimiento de posición de **WhatsApp** para una segunda vía de rastreo.

**2. Monitorización y recepción de datos:**
El sistema de monitoreo en tierra incluye:
- Un **TBeam 1.2** que recibe la señal LoRa de la sonda..
- Un **móvil**, conectado al TBeam por WiFi, que envía los datos al servidor vía 4G utilizando el protocolo MQTT.

El **ESP32** del TBeam 1.2 dentro de la sonda también guarda los datos en un **archivo de log**, accesible tras la recuperación de la sonda, como respaldo adicional de la información transmitida.

**3. Infraestructura de servidor y visualización de datos:**
La infraestructura del servidor está alojada en **máquinas virtuales con Ubuntu Server 24.04**. Para la implementación de los servicios, se utiliza **Docker Compose** que despliega los siguientes contenedores:
- **Mosquitto** como broker MQTT, gestionando las comunicaciones entre la sonda y el sistema terrestre.
- **Telegraf**, encargado del procesamiento y transmisión de los datos hacia la base de datos.
- **InfluxDB**, donde se almacenan las métricas de temperatura, presión, altitud y posición de la sonda.
- **Grafana**, que mediante el plugin **GEOMAP** permite visualizar en un mapa la trayectoria de la sonda y las condiciones atmosféricas en tiempo real.

**4. Seguridad y optimización de la infraestructura:**
El servidor de Grafana está protegido por un **proxy inverso NGINX**, montado en otra máquina virtual, también con **Ubuntu Server 24.04**. Este proxy está configurado con **certificados Let's Encrypt** para asegurar las comunicaciones mediante HTTPS y configurado mediante reglas de **firewall** que mitigan ataques simples de DDoS.

**Propuesta de mejoras:**
- **Redundancia en el sistema de rastreo:** Además del seguimiento por LoRa y WhatsApp, se podrían añadir módulos satelitales para garantizar una cobertura global, más móviles o dispositivos T-Beam.
- **Optimización de la energía:** Considerar fuentes adicionales de energía, como paneles solares, para prolongar la autonomía de la sonda. En el caso de tardar más de lo previsto en encontrar la sonda una vez cae a tierra, pueden ser de vital importancia para alargar el tiempo en que la sonda nos transmite su posición.
- **Transmisión de imágenes:** Aunque no es posible transmitir videos en tiempo real a través de LoRa, se podrían almacenar videos de alta calidad en el móvil y transmitirlos aun no siendo posible la recuperación de la sonda  y siempre y cuando haya conectividad suficiente.

