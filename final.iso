#include <SoftwareSerial.h>
#include <ESP8266WiFi.h>
#include <PZEM004Tv30.h>//https://github.com/mandulaj/PZEM-004T-v30
#include <Wire.h> 
PZEM004Tv30 pzem(&Serial);

// Replace with your WiFi credentials
const char* ssid = "ECE 4A";
const char* password = "#ece4ever";

// ThingSpeak settings
const char* thingspeakApiKey = "FNWTN8WLEWSJY462";
const char* thingspeakHost = "api.thingspeak.com";


const char* iftttnomovement = "No_motion";

const char* iftttPowercrossed = "Power_crossed";

const char* iftttKey = "nt52DsnL-9CYrPXCPZ-GFYyGNBdRLIs1qZq6KLlACa1";


const char* serverName = "maker.ifttt.com";//pir 

// PIR sensor settings
const int pirPin = D7;
unsigned long noMovementStartTime = 0;
bool movementDetected = false;
const unsigned long noMovementTimeout = 10000; // 30 minutes in milliseconds

// PZEM-004T settings
// SoftwareSerial pzemSerial(D1, D2);  // RX, TX
float voltage=1000;
float current=0;
float power=0;
float energy=0;
float frequency=0;
float pf=0;

void setup() {
  Serial.begin(9600);
  // pzemSerial.begin(9600);

  delay(1000);
  pinMode(pirPin, INPUT);
  delay(1000);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }

  Serial.println("Connected to WiFi");
}

void loop() {
  readPZEM();
  delay(1000);
  sendToThingSpeak();
  delay(10000); // Send data every 10 seconds

  checkpir();
}

// void readPZEM() {
//   pzemSerial.write(0xB0);
//   delay(500);

//   if (pzemSerial.available() >= 10) {
//     byte response[10];
//     pzemSerial.readBytes(response, 10);

//     if (response[0] == 0xA0 && response[9] == 0xA5) {
//       voltage = ((response[2] << 8) + response[3]) / 10.0;
//       current = ((response[4] << 8) + response[5]) / 1000.0;
//       power = ((response[6] << 8) + response[7]) / 10.0;
//     }
//   }
// }
void checkpir()
{
  if (digitalRead(pirPin) == HIGH) {
    movementDetected = true;
    Serial.println("detected");
    //lcd.print("detected");

    noMovementStartTime = millis();
  } else {
    if (movementDetected && millis() - noMovementStartTime >= noMovementTimeout) {
      Serial.println("no");
      //lcd.print("no");
      sendNoMovementNotification();
      movementDetected = false;
    }
  }

}

void readPZEM()
{
  
  //Blynk.run();
  
    voltage = pzem.voltage();
    if( !isnan(voltage) ){
        Serial.print("Voltage: "); Serial.print(voltage,1); Serial.println("V");
        //lcd.setCursor(0,0); lcd.print("Voltage:      Volts"); lcd.setCursor(8,0); lcd.print(voltage,1);
    } else {
        Serial.println("Error reading voltage");
        // lcd.clear();
        // lcd.setCursor(0,0); lcd.print("Voltage = 0.00Volts "); 
        // lcd.setCursor(0,1); lcd.print("Current = 0 Amps  "); 
        // lcd.setCursor(0,3); lcd.print("  No Power  Supply  ");
    }

    current = pzem.current();
    if( !isnan(current) ){
        Serial.print("Current: "); Serial.print(current,3); Serial.println("A");
        //lcd.setCursor(0,1); lcd.print("Current:      Amps"); lcd.setCursor(8,1); lcd.print(current,3);
    } else {
        Serial.println("Error reading current");
        
    }

    float power = pzem.power();
    if( !isnan(power) ){
        Serial.print("Power: "); Serial.print(power,0); Serial.println("W");
        if(power == 0)
        {
          sendPowerCrossed();
        }
        //lcd.setCursor(0,2); lcd.print("     Watts"); lcd.setCursor(0,2); lcd.print(power,0);
    } else {
        Serial.println("Error reading power");
            }

    energy = pzem.energy();
    if( !isnan(energy) ){
        Serial.print("Energy: "); Serial.print(energy, 3); Serial.println("kWh");
        //lcd.setCursor(12,2); lcd.print("     Kwh"); lcd.setCursor(12,2); lcd.print(energy,3);
    } else {
        Serial.println("Error reading energy");
    }

    frequency = pzem.frequency();
    if( !isnan(frequency) ){
        Serial.print("Frequency: "); Serial.print(frequency,1); Serial.println("Hz");
        //lcd.setCursor(0,3); lcd.print("Freq:    Hz"); lcd.setCursor(5,3); lcd.print(frequency,1);
    } else {
        Serial.println("Error reading frequency");
    }

    float pf = pzem.pf();
    if( !isnan(pf) ){
        Serial.print("PF: "); Serial.println(pf);
        //lcd.setCursor(12,3); lcd.print("PF: "); lcd.setCursor(15,3); lcd.print(pf);
    } else {
        Serial.println("Error reading power factor");
        
    }

    Serial.println();
    delay(2000);
    



      //Publish data every 5 seconds (5000 milliseconds). Change this value to publish at a different interval.
          // if (millis() - lastMillis > 5000) {
          //   lastMillis = millis();
            
          //   Blynk.virtualWrite(V1, voltage);
          //   Blynk.virtualWrite(V2, current);            
          //   Blynk.virtualWrite(V3, power);
          //   Blynk.virtualWrite(V4, energy);
          //   Blynk.virtualWrite(V5, frequency);
          //   Blynk.virtualWrite(V6, pf);            

          // }         
  
}



void sendToThingSpeak() {
  WiFiClient client;
  
  if (client.connect(thingspeakHost, 80)) {
    String data = "api_key=" + String(thingspeakApiKey) +
                  "&field1=" + String(energy);

    client.println("POST /update HTTP/1.1");
    client.println("Host: " + String(thingspeakHost));
    client.println("Connection: close");
    client.println("Content-Type: application/x-www-form-urlencoded");
    client.println("Content-Length: " + String(data.length()));
    client.println();
    client.println(data);

    int timeout = millis() + 5000; // 5-second timeout
    while (!client.available() && millis() < timeout) {
      delay(1);
    }

    while (client.available()) {
      char c = client.read();
      Serial.print(c);
    }
    
    client.stop();
  }
}




void sendNoMovementNotification() {
  // Create HTTP client
  WiFiClient client;
  if (client.connect(serverName, 80)) {
    String url = "/trigger/" + String(iftttnomovement) + "/with/key/" + String(iftttKey);

    // Send HTTP GET request
    client.print(String("GET ") + url + " HTTP/1.1\r\n" +
                 "Host: " + serverName + "\r\n" +
                 "Connection: close\r\n\r\n");

    Serial.println("HTTP GET request sent to IFTTT");

    // Wait for response
    while (client.connected()) {
      if (client.available()) {
        String line = client.readStringUntil('\r');
        Serial.print(line);
      }
    }
  }

  
  
  client.stop();

  
}














void sendPowerCrossed()
  {
      WiFiClient client;
    if (client.connect(serverName, 80)) {
      String url = "/trigger/" + String(iftttPowercrossed) + "/with/key/" + String(iftttKey);

      // Send HTTP GET request
      client.print(String("GET ") + url + " HTTP/1.1\r\n" +
                  "Host: " + serverName + "\r\n" +
                  "Connection: close\r\n\r\n");

      Serial.println("HTTP GET request sent to IFTTT");

      // Wait for response
      while (client.connected()) {
        if (client.available()) {
          String line = client.readStringUntil('\r');
          Serial.print(line);
        }
      }
    }

    
    
    client.stop();
  }
