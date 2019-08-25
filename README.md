# arduino-code
arduino code for a smart poultry farm
#include<SoftwareSerial.h>
#define sound A0
SoftwareSerial m(11,10);//RX TX
String data1;
void setup() {
// put your setup code here, to run once:
m.begin(115200);
Serial.begin(115200);

}
void loop() {
float sound_val=analogRead(sound);
sound_val= (sound_val+83.2073) / 11.003;;
Serial.println(sound_val);
delay(1000);
String data="#"+String(sound_val)+"@";
m.print(data);
Serial.println("data sent");
delay(1000);
}


#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <MQ2.h>
#include <Wire.h> 
#include<SoftwareSerial.h>
SoftwareSerial m(D6,D7);
String data;
int Analog_Input = A0;
float lpg, co, smoke;
#include <Servo.h>
int T;
Servo myservo; 
MQ2 mq2(Analog_Input); 
//-------- Customise these values -----------
const char* ssid = "Nitham4gc2";
const char* password = "Nitham4gc2";
#include "DHT.h"
#define DHTPIN D0   // what pin we're connected to
#define DHTTYPE DHT11   // define type of sensor DHT 11
DHT dht (DHTPIN, DHTTYPE);
 
#define ORG "q09c66"
#define DEVICE_TYPE "mah"
#define DEVICE_ID "61099"
#define TOKEN "160217735026"
//-------- Customise the above values --------
int pos; 
char server[] = ORG ".messaging.internetofthings.ibmcloud.com";
char topic[] = "iot-2/evt/Data/fmt/json";
char authMethod[] = "use-token-auth";
char token[] = TOKEN;
char clientId[] = "d:" ORG ":" DEVICE_TYPE ":" DEVICE_ID;
 
WiFiClient wifiClient;
PubSubClient client(server, 1883,wifiClient);

void setup() {
  m.begin(115200);
 Serial.begin(115200);
 Serial.println();
 dht.begin();
 mq2.begin();
 myservo.attach(D3);
 Serial.print("Connecting to ");
 Serial.print(ssid);
 WiFi.begin(ssid, password);//connecting mcu to the wifi
 while (WiFi.status() != WL_CONNECTED) {
 delay(500);
 Serial.print(".");
 } 
 Serial.println("");
 
 Serial.print("WiFi connected, IP address: ");
 Serial.println(WiFi.localIP());
 while (!m) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

}
 
void loop() {
 if (m.available()) {
    m.write(m.read());
  }

float h = dht.readHumidity();
float t = dht.readTemperature();

 float* values= mq2.read(true); //set it false if you don't want to print the values in the Serial
  //lpg = values[0];
  lpg = mq2.readLPG();
  //co = values[1];
  co = mq2.readCO();
  //smoke = values[2];
  smoke = mq2.readSmoke();
//  lcd.setCursor(0,0);
  Serial.println("LPG:");
  Serial.println(lpg);
  Serial.println("CO:");
  Serial.println(co);
  Serial.println("SMOKE:");
  Serial.println(smoke);
  Serial.println("PPM");
  delay(5000);

if (isnan(h) || isnan(t)||isnan(lpg))
{
Serial.println("Failed to read from DHT sensor!");
delay(1000);
return;
}
PublishData(t,h,lpg);
delay(100);
}

void PublishData(float t, float h,float lpg){
 if (!!!client.connected()) {
 Serial.print("Reconnecting client to ");
 Serial.println(server);
 while (!!!client.connect(clientId, authMethod, token)) {
 Serial.print(".");
 delay(500);
 }
 Serial.println();
 }
  String payload = "{\"d\":{\"t\":";
  payload += t;
  payload +="," "\"h\":";
  payload += h;
  payload +="," "\"lpg\":";
  payload += lpg;
  
  payload += "}}";
 Serial.print("Sending payload: ");
 Serial.println(payload);
  
 if (client.publish(topic, (char*) payload.c_str())) {
 Serial.println("Publish ok");
 } else {
 Serial.println("Publish failed");
 }
 while(t==25)
 {t=T;}
 if(t>=25)
 for (pos = 0; pos <= 180; pos += 1) { // goes from 0 degrees to 180 degrees
    // in steps of 1 degree
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
  
}

 // run over and over
