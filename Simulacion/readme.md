![Caratula AyC - TP#4](https://github.com/ISPC-TST-ARQUITECTURA-Y-CONECTIVIDAD/tarea4-grupo-7/assets/46485082/6ea603a9-b18f-4781-a304-7f50de4642cf)

# $\textcolor{green}{Cuestionario:}$

- [x] (4)  Implementar un codigo JSON, para comunicar un sensor de temperatura y humedad con un Arduino Uno, simulando los mismos en Proteus.
Cuales serian los campos minimos para hacer la implementacion?



# $$\textcolor{green}{Codigo\ :}$$

```C++
//Se incluyen las librerias necesarias para que el codigo funcione correctamente
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <ArduinoJson.h>

// Se configuran los parametros de la red Wifi y del servidor MQTT
const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqttServer = "broker.emqx.io";
int port = 1883;
String stMac;
char mac[50];
char clientId[50];

WiFiClient espClient;
PubSubClient client(espClient);

//Se definen los parametros de conexion del sensor DHT22
#define DHTPIN 2
#define DHTTYPE DHT22// Tipo de sensor DHT (DHT22)

DHT dht(DHTPIN, DHTTYPE);


void setup() {
  Serial.begin(115200);
  randomSeed(analogRead(0));

  delay(10);
  Serial.println();
  Serial.print("Conectando a ");
  Serial.println(ssid);

  wifiConnect();// Conexión WiFi

  Serial.println("");
  Serial.println("WiFi conectado");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  Serial.println(WiFi.macAddress());
  stMac = WiFi.macAddress();
  stMac.replace(":", "_");
  Serial.println(stMac);

  client.setServer(mqttServer, port);
  client.setCallback(callback);// Establecer la función de devolución de llamada para recibir mensajes MQTT



  dht.begin();// Iniciar el sensor DHT
}

void wifiConnect() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
}

void mqttReconnect() {
  while (!client.connected()) {
    Serial.print("Intentando la conexión MQTT...");
    long r = random(1000);
    sprintf(clientId, "clientId-%ld", r);
    if (client.connect(clientId)) {
      Serial.print(clientId);
      Serial.println(" conectado");
      client.subscribe("topicName/dht_ISPC_G7");// Suscribirse al tema para recibir datos del sensor
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" intentar de nuevo en 5 segundos");
      delay(5000);
    }
  }
}

void callback(char* topic, byte* message, unsigned int length) {
  Serial.print("Mensaje del topic: ");
  Serial.print(topic);
  Serial.print(". Mensaje: ");
  String stMessage;

  for (int i = 0; i < length; i++) {
    Serial.print((char)message[i]);
    stMessage += (char)message[i];
  }
  Serial.println();
}

void loop() {
  delay(2000);

  if (!client.connected()) {
    mqttReconnect();
  }
  client.loop();

  // Leer la temperatura y humedad del sensor DHT

  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  //Codigo JSON para enviar datos de Temperatura y Humedad del sensor

  StaticJsonDocument<200> jsonDocument;
  jsonDocument["device_id"] = stMac;
  jsonDocument["temperatura"] = String(temperature) + " ºC";
  jsonDocument["humedad"] = String(humidity) + " %";

  char jsonStr[200];
  serializeJson(jsonDocument, jsonStr);

  client.publish("topicName/dht_ISPC_G7", jsonStr); // Publicar los datos en formato JSON
}

```

# $\textcolor{green}{Simulacion\ en\ Wokwi:}$

https://wokwi.com/projects/364566222763822081



# $\textcolor{red}{Imagenes\ de\ la\ simulacion}$

![image](https://github.com/ISPC-TST-ARQUITECTURA-Y-CONECTIVIDAD/tarea4-grupo-7/assets/46485082/787fec0b-8d88-481e-95ed-6b396e9cdfc0)

![image](https://github.com/ISPC-TST-ARQUITECTURA-Y-CONECTIVIDAD/tarea4-grupo-7/assets/46485082/595dec60-2360-4a0e-af31-55c41e017ad6)


# $\textcolor{green}{Vista\ suscripcion\ Broker\ MQTT:}$

![image](https://github.com/ISPC-TST-ARQUITECTURA-Y-CONECTIVIDAD/tarea4-grupo-7/assets/46485082/f38133ec-55ee-4528-a1b2-bf6f88f114b6)

![image](https://github.com/ISPC-TST-ARQUITECTURA-Y-CONECTIVIDAD/tarea4-grupo-7/assets/46485082/bbd3bc0f-1645-4c66-a839-bae0887965a0)

![image](https://github.com/ISPC-TST-ARQUITECTURA-Y-CONECTIVIDAD/tarea4-grupo-7/assets/46485082/a1e89c2a-268d-4a61-9c94-3b5d5b6ae067)


- [x] Aplicacion MyMQTT



# $\textcolor{green}{Conclusion:}$

Resumen sobre las ventajas y desventajas de utilizar código JSON para el envío de datos:

Ventajas de utilizar código JSON:

- [x] Estructura de datos flexible:
      JSON permite representar datos de manera estructurada y flexible, lo que facilita su manipulación y transmisión.
      
- [x] Legibilidad y facilidad de uso: 
      JSON utiliza una sintaxis sencilla y legible para representar datos, lo que facilita la comprensión y el mantenimiento del código.

- [x] Compatibilidad: 
      JSON es ampliamente compatible con diferentes lenguajes de programación y plataformas, lo que permite la interoperabilidad de sistemas y aplicaciones.
      
- [x] Soporte de librerías:
      Existen numerosas librerías y herramientas que facilitan la manipulación y el procesamiento de datos en formato JSON, lo que agiliza el desarrollo de aplicaciones.

Desventajas de utilizar código JSON:

- [x] Sobrecarga de tamaño: JSON puede tener una sobrecarga de tamaño en comparación con otros formatos de datos más compactos como el binario. Esto puede afectar la eficiencia y el rendimiento en situaciones donde la velocidad de transmisión es crítica.

- [x] Parsing y procesamiento: El análisis y procesamiento de datos JSON puede requerir más recursos computacionales en comparación con formatos de datos más simples, lo que puede afectar el rendimiento en dispositivos con recursos limitados.

- [x] Limitaciones en la representación de tipos de datos complejos: JSON tiene limitaciones en la representación de tipos de datos complejos, como fechas, estructuras de datos anidadas o referencias, lo que puede requerir soluciones adicionales para abordar estas limitaciones.

En general, JSON es ampliamente utilizado y es una opción popular para el intercambio de datos debido a su flexibilidad, compatibilidad y facilidad de uso. Sin embargo, es importante considerar las necesidades específicas del proyecto y evaluar si JSON es la mejor opción en términos de eficiencia, rendimiento y capacidad de representación de datos.




