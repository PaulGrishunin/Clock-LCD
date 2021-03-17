https://www.tinkercad.com/things/2b2LamLocRM-proj1/editel

https://www.tinkercad.com/things/2b2LamLocRM-proj1/editel?sharecode=5gd_PxZ80_04jN6_v3E4_EXl0z8LPg1gli35xUYVpTE

#define F_CPU   16000000UL
#include <avr/delay.h>
#include <avr/io.h>
#include <string.h>
#include <avr/interrupt.h>
#include <LiquidCrystal.h>
#include <Wire.h>
#define MODE_AM 1    // numbers of display mode (from 0 to 1 )
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

const short PotenciometrPin = 0; // select the input pin for the potentiometer
int Value=0;
float Value_volt=0;
float XXXX = 0.01;   // Each defined time (XXXX in seconds) you should save a measurement together with a time in  array(s).
byte mode = 0;       //start display mode 
float voltArray[100];
int hoursArray[100];
int minutesArray[100];
int secondsArray[100];
int m = 0;
int h = 0;
int min = 0;
int s = 0;         
int HH = 2;            // Set HH:MM:SS for  Send all saved measurements  (together with times) via serial interface
int MM = 3;
int Sec = 1;
const short buttMode = 6;
const short buttSetH = 9;
const short buttSetM = 10;
int myTimer = 0;
int inactivTimer = 0;
boolean btnStateMode, btnFlagMode;
boolean btnStateSetH, btnFlagSetH;
boolean btnStateSetM, btnFlagSetM;
volatile unsigned char seconds;
volatile unsigned char minutes;
volatile unsigned char hours;

void update_clock()           //standart run time
   {
  seconds++;
    if (seconds == 60)
    {
        seconds = 0;
        minutes++;
    }
    if(minutes==60)
    {
      minutes=0;
      hours++;
    }
    if(hours>23)
    {
      hours=0;
    }
}

ISR(TIMER1_COMPA_vect)
{
  update_clock();     
}

void setup() 
{                
  // initialize Timer1
  pinMode(buttMode, INPUT_PULLUP);
  pinMode(buttSetH, INPUT_PULLUP);
  pinMode(buttSetM, INPUT_PULLUP);
  cli();          // disable global interrupts
  TCCR1A = 0;     // set entire TCCR1A register to 0
  TCCR1B = 0;     // same for TCCR1B

  // set compare match register to desired timer count:
  OCR1A = 15624;
  // turn on CTC mode:
  TCCR1B |= (1 << WGM12);
  // Set CS10 and CS12 bits for 1024 prescaler:
  TCCR1B |= (1 << CS10);
  TCCR1B |= (1 << CS12);
  // enable timer compare interrupt:
  TIMSK1 |= (1 << OCIE1A);
  sei();          // enable global interrupts
  lcd.begin(16, 2);
  lcd.print("HH:MM:SS");
  Serial.begin(9600);   
}

void display_time_on_lcd()     // display actual time on LCD 
{
  lcd.setCursor(0, 0);
  lcd.print("HH:MM:SS");
  lcd.setCursor(0, 1);
  if (hours<10)
    lcd.print("0");
  lcd.print(hours);
  lcd.setCursor(2,1);
  lcd.print(":");
  lcd.setCursor(3,1);  
  if (minutes<10)
    lcd.print("0");
  lcd.print(minutes);
  lcd.setCursor(5,1);
  lcd.print(":");
  lcd.setCursor(6,1);
  if (seconds<10)
    lcd.print("0");
  lcd.print(seconds);
}

void display_potentiometer()
  {
 lcd.setCursor(0, 0);
 lcd.print("Potentiometer"); 
lcd.setCursor(4, 1);
lcd.print(" V       "); 
lcd.setCursor(0, 1);
lcd.print(Value_volt);
delay(50);
}

void set_time()           //set time by buttons
{
   inactivTimer = millis();
    while(millis() - inactivTimer <= 5000) {      // after 5 second of inactivity on buttons
  lcd.setCursor(0,0);
  lcd.print("SET TIME         ");
  btnStateSetH = !digitalRead(buttSetH);  //set minutes by push button buttSetH
      if(btnStateSetH==1) {        
       hours++;
    if (hours > 23){
      hours = 0;  }   
   /* lcd.setCursor(0, 1);
  if (hours<10)
    lcd.print("0");
  lcd.print(hours);
  lcd.setCursor(2,1);
  lcd.print(":");  */
  inactivTimer = millis();
      }
  btnStateSetM = !digitalRead(buttSetM);  //set minutes by push button buttSetM
  if(btnStateSetM==1){ 
      minutes++;
     if (minutes > 59)
        minutes = 0;
  /*  lcd.setCursor(3,1);  
  if (minutes<10)
    lcd.print("0");
  lcd.print(minutes);
  lcd.setCursor(5,1);
  lcd.print(":");  */
   inactivTimer = millis();
  }
      lcd.setCursor(0, 1);
  if (hours<10)
    lcd.print("0");
  lcd.print(hours);
  lcd.setCursor(2,1);
  lcd.print(":");
  lcd.setCursor(3,1);  
  if (minutes<10)
    lcd.print("0");
  lcd.print(minutes);
  lcd.setCursor(5,1);
  lcd.print(":");
  lcd.setCursor(6,1);
  if (seconds<10)
    lcd.print("0");
  lcd.print(seconds);
 }     
  seconds = 0;    //seconds should be zeroed and you should leave the subroutine 
}
   

void loop()
{  
  Value = analogRead(PotenciometrPin);
  Value_volt=(float)5/1025*Value;   // constant counting of volts on a potentiometer
  
    if (millis() - myTimer >= XXXX*1000) {      // pushes data to an array 1 times per second. Changeable
    myTimer = millis(); 
    voltArray[m++] = Value_volt;
    hoursArray[h++] = hours;  
    minutesArray[min++] = minutes;  
    secondsArray[s++] = seconds;  
       if(m == 99){
        m = 0;
       }
    }
  if (hours == HH && minutes == MM && seconds == Sec){
        for (int i = 0; i < 99; i++) {   // Send all saved measurements  (together with times) via serial interface at a predefined time (HH:MM:SS) 
          if (hoursArray[i]<10)
          Serial.print("0");
          Serial.print(hoursArray[i]);
          Serial.print(":");
          if (minutesArray[i]<10)
          Serial.print("0");
          Serial.print(minutesArray[i]);
          Serial.print(":");
          if (secondsArray[i]<10)
          Serial.print("0");
          Serial.print(secondsArray[i]); 
          Serial.print(" - ");  
          Serial.print(voltArray[i]);
          Serial.println(" V ");         
        }
      }
  
  btnStateMode = !digitalRead(buttMode);    
  if (btnStateMode && !btnFlagMode) {    
    btnFlagMode = true;                 
    ++mode;
    lcd.clear();
        if (mode > MODE_AM) mode = 0;    
      }
  if (!btnStateMode && btnFlagMode) {    
    btnFlagMode = false;                    
  }
  
  switch (mode) {                  // change display mode on lcd
    case 0: display_time_on_lcd();  
      break;
    case 1: display_potentiometer();
      break;
  }
  
  btnStateSetH = !digitalRead(buttSetH);  //set minutes by push button buttSetH
  if(btnStateSetH==1) 
{   
  set_time();    
} 
  
}