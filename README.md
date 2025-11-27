# 🛡️ Server Room Guardian (Embedded Security and Monitoring)

## 🎥 DEMONSTRAȚIE VIDEO (Demo Video)

**Vă rugăm să vizionați clipul demonstrativ al funcționalității sistemului:**

[CLICK AICI PENTRU VIDEO DEMO COMPLET](https://youtu.be/kEjlJp-8QzE)

---

## 🛠️ Detalii Proiect
Acest proiect este un Proof of Concept dezvoltat pe platforma Arduino Uno, având ca scop monitorizarea în timp real a condițiilor critice dintr-un Data Center.

### Competențe Cheie Demonstrarte:
* Senzorică Analogică și Digitală (DHT11, KY-038).
* Programare C++ (Logica de control și calibrare).
* Filtrare zgomot și control de feedback.


 CODUL FOLOSIT IN ARDUINO IDE
 
#include "DHT.h"

#define DHTPIN 2        
#define SOUNDPIN A0      
#define MOTORPIN 4      
#define LED_VERDE 5     
#define LED_ROSU 6      

#define DHTTYPE DHT11   
#define TEMP_LIMITA 30.0 


int PRAG_ZGOMOT = 6; 

DHT dht(DHTPIN, DHTTYPE);
unsigned long ultimulTimpTemp = 0;
unsigned long ultimulTimpPrint = 0;
float ultimaTemperatura = 0;
int bazaLiniste = 0; 

void setup() {
  Serial.begin(9600);
  dht.begin();
  
  pinMode(MOTORPIN, OUTPUT);
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_ROSU, OUTPUT);
  
  Serial.println("Calibrare senzor... (Liniste 1 secunda)");
  long suma = 0;
  for(int i=0; i<50; i++) {
    suma += analogRead(SOUNDPIN);
    delay(10);
  }
  bazaLiniste = suma / 50;
  Serial.println("Sistem Gata. Baza stabilita.");
  
  digitalWrite(LED_VERDE, HIGH); 
}

void loop() {
  int valoareSenzor = analogRead(SOUNDPIN);
  int intensitate = abs(valoareSenzor - bazaLiniste);
  
  if (millis() - ultimulTimpTemp > 2000) {
    ultimaTemperatura = dht.readTemperature();
    ultimulTimpTemp = millis();
  }

  if (intensitate > PRAG_ZGOMOT) {
    
    Serial.print("Temperatura: ");
    Serial.print(ultimaTemperatura);
    Serial.println(" grade C | Zgomot: DA");
    
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_ROSU, HIGH);
    digitalWrite(MOTORPIN, HIGH);
    
    delay(3000); 
    
    digitalWrite(LED_ROSU, LOW);
    digitalWrite(MOTORPIN, LOW);
    
    delay(500);
    
    digitalWrite(LED_VERDE, HIGH);
  }
  else if (ultimaTemperatura > TEMP_LIMITA) {
    Serial.println("!!! SUPRAINCALZIRE DETECTATA !!!");
    
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_ROSU, HIGH);
    digitalWrite(MOTORPIN, HIGH);
    delay(500);
    digitalWrite(LED_ROSU, LOW);
    digitalWrite(MOTORPIN, LOW);
  }
  else {
    if (millis() - ultimulTimpPrint > 1000) {
       Serial.print("Temperatura: ");
       Serial.print(ultimaTemperatura);
       Serial.println(" grade C | Zgomot: NU");
       ultimulTimpPrint = millis();
    }
    
    digitalWrite(LED_VERDE, HIGH);
    digitalWrite(LED_ROSU, LOW);
    digitalWrite(MOTORPIN, LOW);
  }
  
  delay(10); 
}
