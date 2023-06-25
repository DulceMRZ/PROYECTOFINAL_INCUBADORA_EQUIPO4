# Proyecto Final: Incubadora de Huevos mediante el modulo ESP32
 

En esta proyecto podemos programar una ESP32 con sensor ultrasonido y DHT22, visualizando los datos mediante NODE-RED y obteniendo una Base de datos para verificar los datos en tiempo real. 

## Contenido 

- Introducción 
- Objetivo
- Material Requerido
- Procedimiento 
- Instrucciones de operación 
- Resultados 



## 1. Introducción

En este proyecto se utilizara una ```Esp32```, la cual nos permite utilizar un entorno de adquision de datos, lo cual en esta practica ocuparemos un ultrasonido (```HC-SR04 Ultrasonic distance sensor```) un sensor (```DTH22```).

### 1.1 Descripción
  La ```incubación artificial```, es la incubación de huevos mediante máquinas incubadoras que brindan un medio ambiente adecuado y controlado para que se desarrollen las crías de aves y reptiles. 
  
  A nivel comercial esta ampliamente difundido el uso de incubación artificial para criar gallinas, pavos, patos y codornices.
 
 ### 1.2 Especificación 

 Es importante considerar que esta practica se estara realiando en un simulador llamado [WOKWI](https://https://wokwi.com/).

 Debes considerar tener instalo el ```XAMPP```, que nos permite activar un ```mysql```

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/XAMPP.PNG?raw=true)


## 2. Objetivo 

Mantener una ```Temperatura y Humedad``` adecuada para garantizar un ambiente eficiente en la incubadora. De igual forma el poder estar monitoreando el ```nivel del agua al que se ecnuentra la incubadora``` y poder obtener un resulto eficiente en la incubación de huevos. 

## 3. Material Requerido

Tomar en cuenta el material necesario para realizar esta práctica:

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP32
- Sensor de ultrasonido de distancia 
  (HC-SR04 Ultrasonic distance sensor)
- Sensor DHT22 (Temperatura/Humedad)
- Programa Node - Red 
- Programa XAMPP

## 4. Procedimiento a realizar 

 - Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).


## Paso 1 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include "DHTesp.h"
#include <LiquidCrystal_I2C.h>

#define I2C_ADDR    0x27  //Programación de LCD 
#define LCD_COLUMNS 20
#define LCD_LINES   4
#define BUILTIN_LED 2

const int DHT_PIN = 15;
DHTesp dhtSensor;  // Update these with values suitable for your network.

LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLUMNS, LCD_LINES);

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt= "Proyectoteam4";
String password_mqtt= "1241";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

const int trigPin = 12;
const int echoPin = 13;

const int LED1 = 32;
const int LED2 = 33;
const int LED3 = 19;
const int LED4 = 18;

// defines variables
long duration;
int nivel;
int safetyNivel;
int safetytemperature;
int safetyhumidity;
int Temperatura;
int Humedad;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    //Turn the LED on (Note that LOW is the voltage level
     //but actually the LED is on; this is because
     //it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  lcd.init();
  lcd.backlight();
  pinMode(trigPin, OUTPUT); //pin como salida **sensor ultrasonico**
  pinMode(echoPin, INPUT);  //pin como entrada
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
  pinMode(LED4, OUTPUT);
  digitalWrite(trigPin, LOW);//Inicializamos el pin con 0

}

void loop() {
  //Condiciones de Nivel de agua 
digitalWrite(trigPin, LOW);
delayMicroseconds(2);

// Sets the trigPin on HIGH state for 10 micro seconds
digitalWrite(trigPin, HIGH);
delayMicroseconds(10);
digitalWrite(trigPin, LOW);

// Reads the echoPin, returns the sound wave travel time in microseconds
duration = pulseIn(echoPin, HIGH);
// Calculating the distance
nivel= duration*0.034/2;

//Condiciones de lógica nivel de agua
safetyNivel = nivel;
if (safetyNivel>=2 && safetyNivel<=100)  //nivel bajo de agua 
{
  digitalWrite(LED1, LOW);
  digitalWrite(LED2, LOW);
}
else if(safetyNivel>=100 && safetyNivel<=200) 
{
  digitalWrite(LED1, HIGH);
  digitalWrite(LED2, LOW);
}
else
{
 digitalWrite(LED1,  LOW);
  digitalWrite(LED2, HIGH);
}
// Prints the distance on the Serial Monitor
Serial.print("Nivel: ");
Serial.println(nivel);
delay (2000);

//Condiciones de Temperatura y Humedad 

safetytemperature = Temperatura;
if (safetytemperature = 37)
{
  digitalWrite(LED3, HIGH);
  digitalWrite(LED4, LOW);
}
else if(safetytemperature>=37) 
{
  digitalWrite(LED3, LOW);
  digitalWrite(LED4,LOW);
}
else
{
 digitalWrite(LED3,  LOW);
 digitalWrite(LED4, LOW);
}
// Prints the distance on the Serial Monitor
Serial.print("Temperatura: ");
Serial.println(Temperatura);

//safetyhumidity = Humedad;
//if (safetyhumidity>=0 && safetyhumidity<=20)
//{
  //digitalWrite(LED4, LOW);
   //digitalWrite(LED6, LOW);
//}
//else if(safetyhumidity >=21 && safetyhumidity<=40) 
//{
 // digitalWrite(LED4, HIGH);
   //digitalWrite(LED3, LOW);
//}
//else
//{
// digitalWrite(LED4,  LOW);
  //digitalWrite(LED3, LOW);
//}
// Prints the distance on the Serial Monitor
//Serial.print("Humedad: ");
//Serial.println(Humedad);
//delay(1000);
  //long t; //timepo   
  //long d; //nivel del agua de la incubadora 
  
  //digitalWrite(Trigger, HIGH); 
  //delayMicroseconds(10);          //Enviamos un pulso de 10us
  //digitalWrite(Trigger, LOW); 
  //t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  //d = t/59; 

//DHT22
  TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  Serial.println("Temp: " + String(data.temperature, 1) + "°C");
  Serial.println("Humidity: " + String(data.humidity, 1) + "%");
  Serial.println("---");

  lcd.setCursor(0, 0);
  lcd.print("  Temp: " + String(data.temperature, 1) + "\xDF"+"C  ");
  lcd.setCursor(0, 1);
  lcd.print(" Humidity: " + String(data.humidity, 1) + "% ");
  lcd.print("Wokwi Online IoT");
  delay(3000);

  lcd.setCursor(0, 0);
  lcd.print("PROYECTO         " );
  lcd.setCursor(0, 1);
  lcd.print(" EQUIPO 4        " );
  delay(2000);

  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["NIVEL"] = nivel;
   
    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("equipo4", output.c_str());
  }
}

```

## Paso 2 

2. Instalar las librerias 
   **ArduinoJson** , **DHT SensorLibrary for ESPx**, 
   **PubSubClient**, **LiquidCristal**

Como se muestra en la siguente imagen.

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/libririas.PNG?raw=true)

## Paso 3

3. Hacer la conexion de **LCD, DTH22, HC-SR04 Ultrasonic distance sensor** con la **ESP32** como se muestra en la siguentes imagenes:

3.1 Es importante considerar que para **ESP32** se maneja un ```Voltaje de trabajo 3.3 VDC```. 

a) Observar conexión de  **ESP32**. 

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/CONEXION.PNG?raw=true)

b) Corrobora que el simulador compile bien el programa 

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/COMPILACION.PNG?raw=true)


### 5. Conexión Diagrama en Node - Red

En Node Red, nos estaremos apoyando para poder ver los datos en tiempo real, al momento de simularlos. 

###### Nota: es importante ya contantar con Node Red previamente descargado, y la IP, con la que estaras trabajando. 

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/CONEXION%20NODE%20RED.PNG?raw=true)

Al inciar tu diagrama de bloques debes ajustar información importante en cada Topico: 


a) Para el **Bloque - mqtt in** es importante Seleccionar la opción de "equipo4" y la IP que se tiene en el programa (44.195.202.69), en tu caso anexaras tus datos. 

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/Topico.PNG?raw=true)

###### Nota: Es importante recordar que la IP se estara utilizando la de un servidor publico. Se puede tener una propia pero ello tiene un costo adicional. 

b) Para el **Bloque - JSON** es importante Seleccionar la opción de "Always convert to JavaScript Object"

![](https://github.com/DulceMRZ/PRACTICA7_ESP32_ULTRASONIC/blob/main/TOPIC_JSON.PNG?raw=true)


### 6. Instrucciónes de operación

1. Iniciar simulador.
2. Visualizar los datos en el monitor serial.
3. Colocar la temperatura y humedad dando *doble click* al sensor **DHT22** 
4. Monitero del Nivel de agua al que el sensor ultrasonico esta operando.


## 7. Resultados

Cuando haya funcionado, verás los valores dentro del monitor serial como se muestra en la siguente imagen.

a) **Conexión del Circuito en ESP32 corriendo** 

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/RESULTADOS.PNG?raw=true)




b) **Datos de la ESP32 visualizados en el Dashboard de Node - Red**

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/DASHBOARD.PNG?raw=true)

c) **Base de datos de la incubadora**

![](https://github.com/DulceMRZ/PROYECTOFINAL_INCUBADORA_EQUIPO4/blob/main/LOCALHOST.PNG?raw=true)


# Créditos

Desarrollado por David Vargas, Dulce M Rodriguez y Rodrigo Vega

- [GitHub](https://github.com/DulceMRZ)