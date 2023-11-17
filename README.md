# Soil-Sensor-
To Test the Soil on demand for Small Farm Communities
//Heidi Johnson
//Sip Project 
//Soil Sample using the NPK Soil Tester

//Reference: https://how2electronics.com/measure-soil-nutrient-using-arduino-soil-npk-sensor/

//Libraries that I need to run the code
#include <SoftwareSerial.h>
#include <Wire.h>
//Libraries to usr the OLED these need to be downloaded from website
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
 
 //This needs redefined to reflect the LED screen I have on hand.
#define SCREEN_WIDTH 128    // OLED display width, in pixels
#define SCREEN_HEIGHT 64    // OLED display height, in pixels
#define OLED_RESET -1       // Reset pin # (or -1 if sharing Arduino reset pin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
 
 //These are used in the RS 485 modbus commmunication
 //Receiver Output. Connects to a serial RX pin on the microcontroller
#define RE 8
//Driver Enable. Active HIGH. Typically jumpered to RE Pin.
#define DE 7
 
 //Declares and initalizes the arrays used for the sensor to read.
//const byte code[]= {0x01, 0x03, 0x00, 0x1e, 0x00, 0x03, 0x65, 0xCD};
const byte nitro[] = {0x01,0x03, 0x00, 0x1e, 0x00, 0x01, 0xe4, 0x0c};
const byte phos[] = {0x01,0x03, 0x00, 0x1f, 0x00, 0x01, 0xb5, 0xcc};
const byte pota[] = {0x01,0x03, 0x00, 0x20, 0x00, 0x01, 0x85, 0xc0};
//I am adding this line for the soil tester can read the ph Balance of the soil.
const byte pH[] = {0x01,0x03, 0x00, 0x??, 0x00, 0x01, 0x??, 0x??};
 
 //Initalizes the byte array to hold a response
byte values[11];
SoftwareSerial mod(2,3);




void setup() {
  // put your setup code here, to run once:

  //initializes the hardware serial port and software serial port at a baud rate of 9600.
  Serial.begin(9600);
  mod.begin(9600);
  //setting the receiver and driver enable to output.
  pinMode(RE, OUTPUT);
  pinMode(DE, OUTPUT);
  
  //Initalizes the OLED and displays a message
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C); //initialize with the I2C addr 0x3C (128x64)
  delay(500);
  display.clearDisplay();
  display.setCursor(25, 15);
  display.setTextSize(1);
  display.setTextColor(WHITE);
  display.println(" NPK Sensor");
  display.setCursor(25, 35);
  display.setTextSize(1);
  display.print("Initializing");
  display.display();
  delay(3000);

}

void loop() {
  // put your main code here, to run repeatedly:
  //Where it continually requests and retrieves Nitrogen, Phosphorous, and Potassium values
  //from the sensor using the respective functions. 
  byte val1,val2,val3;
  val1 = nitrogen();
  delay(250);
  val2 = phosphorous();
  delay(250);
  val3 = potassium();
  delay(250);
  //ph Balance
  val4 = pHBalance();
  delay(250);
  
  //prints the values to the serial monitor 
  Serial.print("Nitrogen: ");
  Serial.print(val1);
  Serial.println(" mg/kg");
  Serial.print("Phosphorous: ");
  Serial.print(val2);
  Serial.println(" mg/kg");
  Serial.print("Potassium: ");
  Serial.print(val3);
  Serial.println(" mg/kg");
  //Printing my ph Balance of the soil
  Serial.print("pH Balance: ");
  Serial.print(val4);
  Serial.print(" pH");
  delay(2000);

 //Update sand dsplays values on the OLED
  display.clearDisplay();
  
 
  display.setTextSize(2);
  display.setCursor(0, 5);
  display.print("N: ");
  display.print(val1);
  display.setTextSize(1);
  display.print(" mg/kg");
 
  display.setTextSize(2);
  display.setCursor(0, 25);
  display.print("P: ");
  display.print(val2);
  display.setTextSize(1);
  display.print(" mg/kg");
 
  display.setTextSize(2);
  display.setCursor(0, 45);
  display.print("K: ");
  display.print(val3);
  display.setTextSize(1);
  display.print(" mg/kg");

  //Displaying the ph Balance
  display.setTextSize(2);
  display.setCursor(0, ??);
  display.print("pH: ");
  display.print(val4);
  display.setTextSize(1);
  display.print(" pH");
 
  display.display();
}
 
 //Functions that handle sending the request byte sequences to the sensor and reading the response.
byte nitrogen(){
  digitalWrite(DE,HIGH);
  digitalWrite(RE,HIGH);
  delay(10);
  if(mod.write(nitro,sizeof(nitro))==8){
    digitalWrite(DE,LOW);
    digitalWrite(RE,LOW);
    for(byte i=0;i<7;i++){
    //Serial.print(mod.read(),HEX);
    values[i] = mod.read();
    Serial.print(values[i],HEX);
    }
    Serial.println();
  }
  return values[4];
}
 
byte phosphorous(){
  digitalWrite(DE,HIGH);
  digitalWrite(RE,HIGH);
  delay(10);
  if(mod.write(phos,sizeof(phos))==8){
    digitalWrite(DE,LOW);
    digitalWrite(RE,LOW);
    for(byte i=0;i<7;i++){
    //Serial.print(mod.read(),HEX);
    values[i] = mod.read();
    Serial.print(values[i],HEX);
    }
    Serial.println();
  }
  return values[4];
}
 
byte potassium(){
  digitalWrite(DE,HIGH);
  digitalWrite(RE,HIGH);
  delay(10);
  if(mod.write(pota,sizeof(pota))==8){
    digitalWrite(DE,LOW);
    digitalWrite(RE,LOW);
    for(byte i=0;i<7;i++){
    //Serial.print(mod.read(),HEX);
    values[i] = mod.read();
    Serial.print(values[i],HEX);
    }
    Serial.println();
  }

  //My code that I have added to this reference code
  byte phBalance(){
    digitalWrite (DE,HIGH);
    digitalWrite(RE,HIGH);
    delay(10);
    if(mod.write(pH,sizeof(pH))==8){
      digitalWrite(DE,LOW);
      digitalWrite(RE,LOW);
      for(byte i=0;i<7;i++){
      //Serial.print(mod.read(),HEX);
      values[i] = mod.read();
      Serial.print(values[i],HEX);  
    }
     Serial.println();
  }//End of pH Balance Function

  return values[4];

}
