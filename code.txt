#define BLYNK_PRINT Serial
#define BLYNK_TEMPLATE_ID "TMPLCfpXi-4B"
#define BLYNK_DEVICE_NAME "Plant Monitoring "

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <OneWire.h>
#include <SPI.h>
#include <DHT.h>
#include <DallasTemperature.h>


#define ONE_WIRE_BUS D2
OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

// Defining credentials
char auth[] = "lZBNyNMQOS4v-Q8Az4Lb57XlcL4Ul-0h"; // the auth code that you got on your gmail
char ssid[] = "Kunal"; // username or ssid of your WI-FI
char pass[] = "12345678"; // password of your Wi-Fi

#define DHTPIN 2
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
SimpleTimer timer;

int relay = 12;
// Variable for humidity and temperature
float h, t;
int sensor = 0;
int output = 0;
int buttonValue;

void setup(){
  Serial.begin(9600);
  pinMode(relay, OUTPUT);
  dht.begin();
//  digitalWrite(relay, HIGH);

  
  timer.setInterval(1000L, sendSensor);
  Blynk.begin(auth, ssid, pass);
  sensors.begin();
  Blynk.virtualWrite(V3,0);
  Blynk.syncAll();
}


// Reading value from DHT11 Sensor
void sendSensor() {
  h = dht.readHumidity();
  t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Blynk.virtualWrite(V5, h); //V5 is for Humidity
  Blynk.virtualWrite(V6, t); //V6 is for Temperature
}



void fun() {
   if (h > 30 && t > 20 && output > 40) {
       digitalWrite(relay, LOW);
       Serial.println("Water Flowing");
       Blynk.virtualWrite(V3,10);
       // Blynk.logEvent("water_required");
       while(output < 60) {
           readMoisture();
           sendSensor();
       }
   }
    digitalWrite(relay,HIGH);
    Blynk.virtualWrite(V3,0);
    delay(10000);

    
}






// For turning on and off motor
BLYNK_WRITE(V3)
{
  buttonValue = param.asInt();
  if (buttonValue == 10) {
    digitalWrite(relay, LOW);
    Serial.println("Water Flowing");
//    while(output < 60) {
//        readMoisture();
//    }
    delay(5000);
    Blynk.virtualWrite(V3,0);
    
    digitalWrite(relay, HIGH);
    // Blynk.logEvent("water_required");
  }
  else if (buttonValue == 0) {
    digitalWrite(relay, HIGH);
    Serial.println("Water not Flowing");
  }
}

void readMoisture() {
    sensor = analogRead(A0);
    Serial.print("Sensor value " );
    Serial.println(sensor);
    output = (145 - map(sensor, 0, 1023, 0, 100)); //in place 145 there is 100(it change with the change in sensor)
    delay(1000);
}

// Reading value from Moisture Sensor
void sendTemps(){
  readMoisture();
//   sensors.requestTemperatures();
//  float temp = sensors.getTempCByIndex(0);
//  Serial.println(temp);
  Serial.print("moisture = ");
  Serial.print(output);
  Serial.println("%");
//  Blynk.virtualWrite(V1, temp);
  Blynk.virtualWrite(V2, output);
  delay(1000);
}

void loop()
{
  Blynk.run();
  timer.run();
  sendTemps();
  sendSensor();
  fun();
}
