#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2); // I2C address 0x27, 16 column and 2 rows
const int trigPin = 14;   // TRIG pin
const int echoPin = 16;

float duration, dist_mm;

const int leverMin = 0;
const int leverMax = 600;

int sensorR = 0;

String ponto;
String thrust;

const int chave1 = 4; //Landing gear
const int chave2 = 3; //Landing gear lock
const int chave3 = 2; //Landing gear lock confirmation

int valor1 = digitalRead(chave1);
int valor2 = digitalRead(chave2);
int valor3 = digitalRead(chave3);

int Lever = analogRead(A1);
int Leverrange = map(Lever, leverMin, leverMax, 0, 3);

const int Led1 = 5; // Retraído
const int Led2 = 6; // Contraído
const int Led3 = 7; // Trava -> Pouso e decolagem
const int Led4 = 8; // Trava -> Taxi
const int Led5 = 9; // Trava -> Voo
const int Led6 = 10; // Alarme -> Retraído na posição HIGH do Landing gear
const int Led7 = 11; // Alarme -> Contraído na posição LOW do Landing gear
const int Led8 = 12; // Alarme -> Contraído durante APROX
const int Led9 = 13; // Alarme -> Landing gear em HIGH em SOLO
const int Led10 =17; // Alarme -> Landing Gear Lock em LOW durante APROX
const int Led11 = 18; // Alarme -> Landing Gear em LOW com manetes fora de CL

const int LedMOTOR = 19;


void setup() {
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(chave1, INPUT);
  pinMode(chave2, INPUT);
  pinMode(chave3, INPUT);

  pinMode(Led1, OUTPUT);
  pinMode(Led2, OUTPUT);
  pinMode(Led3, OUTPUT);
  pinMode(Led4, OUTPUT);
  pinMode(Led5, OUTPUT);
  pinMode(Led6, OUTPUT);
  pinMode(Led7, OUTPUT);
  pinMode(Led8, OUTPUT);
  pinMode(Led9, OUTPUT);
  pinMode(Led10, OUTPUT);
  pinMode(Led11, OUTPUT);

  pinMode(LedMOTOR, OUTPUT);

}


void loop() {
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(1);
  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  dist_mm = duration * 0.034 / 2; // Convert to mm
  lcd.clear();
  lcd.setCursor(0, 0); // start to print at the first row
  lcd.print("Distance: ");
  lcd.print(dist_mm);
  delay(10);

  void AltimeterRead(){
      while(dist_mm >= 200){
        ponto = "VOO";
      }
    sensorR=dist_mm;
      while(dist_mm <  sensorR){
        sensorR=dist_mm;
        Serial.println(ponto = "APRX");

        if(dist_mm = 0){
        Serial.println(ponto = "SOLO");
        break;
        }
      }
  }

  void LeverThrustRead(){
        if (Leverrange == 0) {
        thrust = "IDLE";
      } else if (Leverrange == 1) {
        thrust = "THROTTLE";
      } else if (Leverrange == 2) {
        thrust = "CL";
      } else if (Leverrange == 3) {
        thrust = "TOGA";
      }
  }

  void Conditions(){
      if(valor1 == HIGH){
        if(ponto == "SOLO"){
          digitalWrite(LedMOTOR, LOW);
          digitalWrite(Led9, HIGH);
        }
        else if(ponto == "APROX" || "VOO"){
          digitalWrite(LedMOTOR, HIGH);
          if(valor3 == HIGH){
            digitalWrite(Led2, HIGH);
            digitalWrite(Led5, HIGH);
          }
          else(valor3 == LOW);{
            digitalWrite(Led6, HIGH);
          }
        }
        else(ponto == "APROX");{
            digitalWrite(LedMOTOR, HIGH);
            digitalWrite(Led8, HIGH);
        }
      }

      else if(valor1 == LOW){
        while(ponto == "VOO" || ponto == "APROX"){
          digitalWrite(LedMOTOR, HIGH);
          if(valor3 == HIGH){
            digitalWrite(Led1, HIGH);
            if(valor2 == HIGH){
              digitalWrite(Led3, HIGH);
            }
            else(valor2 == LOW);{
              digitalWrite(Led4, HIGH);
            }
          }
          else(valor3 == LOW);{
            digitalWrite(Led7, HIGH);
          }
        }
      }

      else if(valor2 == LOW && ponto == "APROX"){
        digitalWrite(Led10, HIGH);
      }

      else(ponto == "APROX" && thrust == "TOGA");{
        digitalWrite(Led11, HIGH);
        delay(3000);
        if(valor3 == HIGH){
          digitalWrite(LedMOTOR, HIGH);
          digitalWrite(Led1, HIGH);
        }
        else(valor3 == LOW);{
          digitalWrite(LedMOTOR, LOW);
          digitalWrite(Led8, HIGH);
        }
      }
  }
}
