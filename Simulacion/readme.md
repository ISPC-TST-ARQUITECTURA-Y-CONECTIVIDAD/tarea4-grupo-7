![Caratula AyC - TP#4](https://github.com/ISPC-TST-ARQUITECTURA-Y-CONECTIVIDAD/tarea4-grupo-7/assets/46485082/6ea603a9-b18f-4781-a304-7f50de4642cf)

# $\textcolor{green}{Cuestionario:}$

- [x] (4)  Implementar un codigo JSON, para comunicar un sensor de temperatura y humedad con un Arduino Uno, simulando los mismos en Proteus.
Cuales serian los campos minimos para hacer la implementacion?



# $$\textcolor{green}{Codigo\ :}$$

//
C++
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



//


# $\textcolor{green}{Módulo\ II:}$

# $\textcolor{red}{Familia\ de\ Protocolos\ IoT\ -\ II}$

Cada práctica se desarrollará en forma grupal, debiendo subir el
desarrollo de la misma al repositorio (respetando la estructura de
monorepositorio) establecido por grupo. Los ejercicios serán
implementados de forma que a cada integrante le corresponda 1 o más
tareas (issues); por lo que deberán crear el proyecto correspondiente,
con la documentación asociada si hiciera falta, y asignar los issues por
integrante. De esta forma quedara documentada la colaboración de
cada alumno.

# $\textcolor{green}{Cuestionario:}$



- [x] (4)  Implementar un codigo JSON, para comunicar un sensor de temperatura y humedad con un Arduino Uno, simulando los mismos en Proteus.
Cuales serian los campos minimos para hacer la implementacion?



# $\textcolor{green}{Bibliografia:}$


Para el desarrollo de la actividad se realizaron consultas a las siguientes fuentes de informacion:

- [ ] Material academico
- [ ] Internet
- [ ] Bibliografia especifica.
