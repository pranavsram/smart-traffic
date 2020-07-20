# smart-traffic
#python code
	import serial
ser = serial.Serial('COM4', 9600)
if ser.isOpen() == False:
        ser.open()
print ("[Info] ser.isOpen() = " + str(ser.isOpen()))
print ("")

from tkinter import *
from tkinter.ttk import *

window = Tk()
window.title("Traffic Control")
window.geometry('350x200')

lbl = Label (window, text = "   Mode  ")
lbl.grid(column=0, row=0)

laneSelection = IntVar()

def mode_clicked_AUTOMATIC ():
    lbl.configure (text = "Automatic")
    ser.write ('A'.encode())
    print ("A")

def mode_clicked_MANUAL ():
    lbl.configure (text = "  Manual ")
    ser.write ('M'.encode())
    print ("M")

def lane_clicked ():
    getValue = laneSelection.get ()
    if getValue == 1:
        ser.write ('A'.encode())
        print ("A")
    elif getValue == 2:
        ser.write ('B'.encode())
        print ("B")
    elif getValue == 3:
        ser.write ('C'.encode())
        print ("C")
    elif getValue == 4:
        ser.write ('D'.encode())
        print ("D")

modA = Button (window, text = "Automatic", command = mode_clicked_AUTOMATIC)
modM = Button (window, text = "Manual", command = mode_clicked_MANUAL)

modA.grid(column=2, row=0)
modM.grid(column=3, row=0)

lane_a = Radiobutton(window, text = 'Lane A', value = 1, variable = laneSelection)
lane_b = Radiobutton(window, text = 'Lane B', value = 2, variable = laneSelection)
lane_c = Radiobutton(window, text = 'Lane C', value = 3, variable = laneSelection)
lane_d = Radiobutton(window, text = 'Lane D', value = 4, variable = laneSelection)
btn = Button (window, text = "Go", command = lane_clicked)

lane_a.grid (column = 0, row = 3)
lane_b.grid (column = 1, row = 3)
lane_c.grid (column = 2, row = 3)
lane_d.grid (column = 3, row = 3)
btn.grid (column = 2, row = 4)


#arduino code
#include <Wire.h> 
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

#define A1_echo       7
#define A1_trigger    6
#define A2_echo       5
#define A2_trigger    4

#define B1_echo       3
#define B1_trigger    2
#define B2_echo       14
#define B2_trigger    15

#define C1_echo       A4
#define C1_trigger    A5
#define C2_echo       A6
#define C2_trigger    A7

#define D1_echo       A8
#define D1_trigger    A9
#define D2_echo       A10
#define D2_trigger    A11

#define A_red         12
#define A_yellow      11
#define A_green       10

#define C_red         A0
#define C_yellow      A1
#define C_green       A2

#define B_red         27
#define B_yellow      29
#define B_green       31

#define D_red         33
#define D_yellow      35
#define D_green       37

#define buzzer        13

unsigned char laneFlag = 'A';
int defaultDuration = 10, count = 0;
int stage1 = 5, stage2 = 10, stage3 = 15, stage4 = 20;
int sen_1 = 0, sen_2 = 0;
int laneA_count = 0, laneB_count = 0, laneC_count = 0, laneD_count = 0;
String runMode = "";

int ultraSoundReading (int trigger, int echo){
  digitalWrite(trigger, LOW);
  delayMicroseconds(2);
  
  digitalWrite(trigger, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigger, LOW);
  
  int pulse_duration = pulseIn(echo, HIGH);
  int dist = pulse_duration*0.034/2;
  return (dist);
}

void pauseFn (int pauseVal) {
  int dispTime = pauseVal;
  
  for (int x = 0; x < pauseVal; x++){
    dispTime = dispTime - 1;
    delay (1000);
    Serial.print (String (dispTime) + ",");
    
    lcd.setCursor(7,0);
    lcd.print(dispTime);
    lcd.print(" ");
  }
  Serial.println ("");
  lcd.clear();
  
  digitalWrite (A_red, LOW);
  digitalWrite (A_yellow, HIGH);
  digitalWrite (A_green, LOW);
  
  digitalWrite (B_red, LOW);
  digitalWrite (B_yellow, HIGH);
  digitalWrite (B_green, LOW);

  digitalWrite (C_red, LOW);
  digitalWrite (C_yellow, HIGH);
  digitalWrite (C_green, LOW);
  
  digitalWrite (D_red, LOW);
  digitalWrite (D_yellow, HIGH);
  digitalWrite (D_green, LOW);
  delay (2500);
}

void setup() {
  Serial.begin (9600);
  
  lcd.init();
  lcd.backlight();
  lcd.print(" TRAFFIC SYSTEM ");
  delay(3000);
  lcd.clear();
  
  pinMode (A1_trigger, OUTPUT);
  pinMode (A2_trigger, OUTPUT);
  pinMode (B1_trigger, OUTPUT);
  pinMode (B2_trigger, OUTPUT);
  pinMode (C1_trigger, OUTPUT);
  pinMode (C2_trigger, OUTPUT);
  pinMode (D1_trigger, OUTPUT);
  pinMode (D2_trigger, OUTPUT);

  pinMode (A1_echo, INPUT);
  pinMode (A2_echo, INPUT);
  pinMode (B1_echo, INPUT);
  pinMode (B2_echo, INPUT);
  pinMode (C1_echo, INPUT);
  pinMode (C2_echo, INPUT);
  pinMode (D1_echo, INPUT);
  pinMode (D2_echo, INPUT);

  pinMode (A_red, OUTPUT);
  pinMode (A_yellow, OUTPUT);
  pinMode (A_green, OUTPUT);
  pinMode (B_red, OUTPUT);
  pinMode (B_yellow, OUTPUT);
  pinMode (B_green, OUTPUT);
  pinMode (C_red, OUTPUT);
  pinMode (C_yellow, OUTPUT);
  pinMode (C_green, OUTPUT);
  pinMode (D_red, OUTPUT);
  pinMode (D_yellow, OUTPUT);
  pinMode (D_green, OUTPUT);
  
  pinMode (buzzer, OUTPUT);

  digitalWrite (A_red, LOW);
  digitalWrite (A_yellow, HIGH);
  digitalWrite (A_green, LOW);
  
  digitalWrite (B_red, LOW);
  digitalWrite (B_yellow, HIGH);
  digitalWrite (B_green, LOW);

  digitalWrite (C_red, LOW);
  digitalWrite (C_yellow, HIGH);
  digitalWrite (C_green, LOW);
  
  digitalWrite (D_red, LOW);
  digitalWrite (D_yellow, HIGH);
  digitalWrite (D_green, LOW);

  bool waitFlag = true;
  while (waitFlag == true) {
    if (Serial.available () > 0) {
      char recVal = Serial.read ();

      if (recVal == 'A') {
        runMode = "Automatic";
        waitFlag = false;
        Serial.println ("Automatic Mode enabled");
        
        lcd.print("    AUTOMATIC   ");
        delay(3000);
        lcd.clear();
      }
      else if (recVal == 'M') {
        runMode = "Manual";
        waitFlag = false;
        Serial.println ("Manual Mode enabled");
        
        lcd.print("     MANUAL     ");
        delay(3000);
        lcd.clear();
      }
    }
  }
}

void loop() {
  while (runMode == "Automatic") {
    sen_1 = ultraSoundReading (A1_trigger, A1_echo);
    sen_2 = ultraSoundReading (A2_trigger, A2_echo);
    Serial.println ("[Lane A] " + String (sen_1) + "," + String (sen_2));
    
    laneA_count = 0;
    if (sen_1 > 1 && sen_1 < 20) {
      laneA_count = laneA_count + 5;
    }
    if (sen_2 > 1 && sen_2 < 20) {
      laneA_count = laneA_count + 5;
    }
    
    sen_1 = ultraSoundReading (B1_trigger, B1_echo);
    sen_2 = ultraSoundReading (B2_trigger, B2_echo);
    Serial.println ("[Lane B] " + String (sen_1) + "," + String (sen_2));
  
    laneB_count = 0;
    if (sen_1 > 1 && sen_1 < 20) {
      laneB_count = laneB_count + 5;
    }
    if (sen_2 > 1 && sen_2 < 20) {
      laneB_count = laneB_count + 5;
    }
    
    sen_1 = ultraSoundReading (C1_trigger, C1_echo);
    sen_2 = ultraSoundReading (C2_trigger, C2_echo);
    Serial.println ("[Lane C] " + String (sen_1) + "," + String (sen_2));
    
    laneC_count = 0;
    if (sen_1 > 1 && sen_1 < 20) {
      laneC_count = laneC_count + 5;
    }
    if (sen_2 > 1 && sen_2 < 20) {
      laneC_count = laneC_count + 5;
    }
    
    sen_1 = ultraSoundReading (D1_trigger, D1_echo);
    sen_2 = ultraSoundReading (D2_trigger, D2_echo);
    Serial.println ("[Lane D] " + String (sen_1) + "," + String (sen_2));
  
    laneD_count = 0;
    if (sen_1 > 1 && sen_1 < 20) {
      laneD_count = laneD_count + 5;
    }
    if (sen_2 > 1 && sen_2 < 20) {
      laneD_count = laneD_count + 5;
    }

    
    if(!laneA_count && !laneB_count && !laneC_count && !laneD_count)
    {
      digitalWrite (A_red, HIGH);
      digitalWrite (A_yellow, LOW);
      digitalWrite (A_green, LOW);
      
      digitalWrite (B_red, HIGH);
      digitalWrite (B_yellow, LOW);
      digitalWrite (B_green, LOW);
      
      digitalWrite (C_red, HIGH);
      digitalWrite (C_yellow, LOW);
      digitalWrite (C_green, LOW);
      
      digitalWrite (D_red, HIGH);
      digitalWrite (D_yellow, LOW);
      digitalWrite (D_green, LOW);
    }
    else{
      if(laneA_count > 0 && laneB_count > 0 && laneC_count > 0 && laneD_count > 0)
      {
        
      }
      else if(laneA_count >0){
        laneFlag = 'A';
      }
      else if(laneB_count >0){
        laneFlag = 'B';
      }
      else if(laneC_count >0){
        laneFlag = 'C';
      }
      else if(laneD_count >0){
        laneFlag = 'D';
      }
      
      int diffCount = 0, smallest = 0, percentage = 0, duration = 0;
      if (laneFlag == 'A') {
        smallest = laneA_count;
        if (smallest > laneB_count) {
          smallest = laneB_count;
        }
        if (smallest > laneC_count) {
          smallest = laneC_count;
        }
        if (smallest > laneD_count) {
          smallest = laneD_count;
        }
        diffCount = laneA_count - smallest;
        
        if (diffCount > 0) {
          percentage = (diffCount * 100) / 10;
          if (percentage <= 25) {
            duration = defaultDuration + stage1;
          }
          else if (percentage > 25 && percentage <= 50) {
            duration = defaultDuration + stage2;
          }
          else if (percentage > 50 && percentage <= 75) {
            duration = defaultDuration + stage3;
          }
          else {
            duration = defaultDuration + stage4;
          }
        }
        else {
          duration = defaultDuration;
        }
        
        laneFlag = 'B';
        Serial.println ("[Difference] " + String (diffCount));
        Serial.println ("[Percentage] " + String (percentage));
        Serial.println ("[Duration] " + String (duration));
        Serial.println ("Lane A Green Time");
        
        digitalWrite (A_red, LOW);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, HIGH);
        
        digitalWrite (B_red, HIGH);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, LOW);
        
        digitalWrite (C_red, HIGH);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, HIGH);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, LOW);
        
        pauseFn (duration);
      }
      else if (laneFlag == 'B') {
        smallest = laneB_count;
        if (smallest > laneA_count) {
          smallest = laneA_count;
        }
        if (smallest > laneC_count) {
          smallest = laneC_count;
        }
        if (smallest > laneD_count) {
          smallest = laneD_count;
        }
        
        diffCount = laneB_count - smallest;
        if (diffCount > 0) {
          percentage = (diffCount * 100) / 10;
          if (percentage <= 25) {
            duration = defaultDuration + stage1;
          }
          else if (percentage > 25 && percentage <= 50) {
            duration = defaultDuration + stage2;
          }
          else if (percentage > 50 && percentage <= 75) {
            duration = defaultDuration + stage3;
          }
          else {
            duration = defaultDuration + stage4;
          }
        }
        else {
          duration = defaultDuration;
        }
        
        laneFlag = 'C';
        Serial.println ("[Difference] " + String (diffCount));
        Serial.println ("[Percentage] " + String (percentage));
        Serial.println ("[Duration] " + String (duration));
        Serial.println ("Lane B Green Time");
        
        digitalWrite (A_red, HIGH);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, LOW);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, HIGH);
        
        digitalWrite (C_red, HIGH);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, HIGH);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, LOW);
        pauseFn (duration);
      }
      else if (laneFlag == 'C') {
        smallest = laneC_count;
        if (smallest > laneA_count) {
          smallest = laneA_count;
        }
        if (smallest > laneB_count) {
          smallest = laneB_count;
        }
        if (smallest > laneD_count) {
          smallest = laneD_count;
        }
        
        diffCount = laneC_count - smallest;
        if (diffCount > 0) {
          percentage = (diffCount * 100) / 10;
          if (percentage <= 25) {
            duration = defaultDuration + stage1;
          }
          else if (percentage > 25 && percentage <= 50) {
            duration = defaultDuration + stage2;
          }
          else if (percentage > 50 && percentage <= 75) {
            duration = defaultDuration + stage3;
          }
          else {
            duration = defaultDuration + stage4;
          }
        }
        else {
          duration = defaultDuration;
        }
        
        laneFlag = 'D';
        Serial.println ("[Difference] " + String (diffCount));
        Serial.println ("[Percentage] " + String (percentage));
        Serial.println ("[Duration] " + String (duration));
        Serial.println ("Lane C Green Time");
        
        digitalWrite (A_red, HIGH);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, HIGH);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, LOW);
        
        digitalWrite (C_red, LOW);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, HIGH);
        
        digitalWrite (D_red, HIGH);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, LOW);
    
        pauseFn (duration);
      }
      else if (laneFlag == 'D') {
        smallest = laneD_count;
        if (smallest > laneA_count) {
          smallest = laneA_count;
        }
        if (smallest > laneB_count) {
          smallest = laneB_count;
        }
        if (smallest > laneC_count) {
          smallest = laneC_count;
        }
        
        diffCount = laneD_count - smallest;
        if (diffCount > 0) {
          percentage = (diffCount * 100) / 10;
          if (percentage <= 25) {
            duration = defaultDuration + stage1;
          }
          else if (percentage > 25 && percentage <= 50) {
            duration = defaultDuration + stage2;
          }
          else if (percentage > 50 && percentage <= 75) {
            duration = defaultDuration + stage3;
          }
          else {
            duration = defaultDuration + stage4;
          }
        }
        else {
          duration = defaultDuration;
        }
        
        laneFlag = 'A';
        Serial.println ("[Difference] " + String (diffCount));
        Serial.println ("[Percentage] " + String (percentage));
        Serial.println ("[Duration] " + String (duration));
        Serial.println ("Lane D Green Time");
        
        digitalWrite (A_red, HIGH);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, HIGH);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, LOW);
        
        digitalWrite (C_red, HIGH);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, LOW);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, HIGH);
    
        pauseFn (duration);
      }
    }
  }
  while (runMode == "Manual") {
    if (Serial.available () > 0) {
      char recVal = Serial.read ();

      if (recVal == 'A') {
        Serial.println ("Lane A selected");
        
        digitalWrite (A_red, LOW);
        digitalWrite (A_yellow, HIGH);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, LOW);
        digitalWrite (B_yellow, HIGH);
        digitalWrite (B_green, LOW);
      
        digitalWrite (C_red, LOW);
        digitalWrite (C_yellow, HIGH);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, LOW);
        digitalWrite (D_yellow, HIGH);
        digitalWrite (D_green, LOW);

        delay (2500);

        digitalWrite (A_red, LOW);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, HIGH);
        
        digitalWrite (B_red, HIGH);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, LOW);
        
        digitalWrite (C_red, HIGH);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, HIGH);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, LOW);
      }
      else if (recVal == 'B') {
        Serial.println ("Lane B selected");
        
        digitalWrite (A_red, LOW);
        digitalWrite (A_yellow, HIGH);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, LOW);
        digitalWrite (B_yellow, HIGH);
        digitalWrite (B_green, LOW);
      
        digitalWrite (C_red, LOW);
        digitalWrite (C_yellow, HIGH);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, LOW);
        digitalWrite (D_yellow, HIGH);
        digitalWrite (D_green, LOW);

        delay (2500);

        digitalWrite (A_red, HIGH);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, LOW);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, HIGH);
        
        digitalWrite (C_red, HIGH);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, HIGH);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, LOW);
      }
      else if (recVal == 'C') {
        Serial.println ("Lane C selected");
        
        digitalWrite (A_red, LOW);
        digitalWrite (A_yellow, HIGH);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, LOW);
        digitalWrite (B_yellow, HIGH);
        digitalWrite (B_green, LOW);
      
        digitalWrite (C_red, LOW);
        digitalWrite (C_yellow, HIGH);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, LOW);
        digitalWrite (D_yellow, HIGH);
        digitalWrite (D_green, LOW);

        delay (2500);

        digitalWrite (A_red, HIGH);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, HIGH);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, LOW);
        
        digitalWrite (C_red, LOW);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, HIGH);
        
        digitalWrite (D_red, HIGH);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, LOW);
      }
      else if (recVal == 'D') {
        Serial.println ("Lane D selected");
        
        digitalWrite (A_red, LOW);
        digitalWrite (A_yellow, HIGH);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, LOW);
        digitalWrite (B_yellow, HIGH);
        digitalWrite (B_green, LOW);
      
        digitalWrite (C_red, LOW);
        digitalWrite (C_yellow, HIGH);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, LOW);
        digitalWrite (D_yellow, HIGH);
        digitalWrite (D_green, LOW);

        delay (2500);

        digitalWrite (A_red, HIGH);
        digitalWrite (A_yellow, LOW);
        digitalWrite (A_green, LOW);
        
        digitalWrite (B_red, HIGH);
        digitalWrite (B_yellow, LOW);
        digitalWrite (B_green, LOW);
        
        digitalWrite (C_red, HIGH);
        digitalWrite (C_yellow, LOW);
        digitalWrite (C_green, LOW);
        
        digitalWrite (D_red, LOW);
        digitalWrite (D_yellow, LOW);
        digitalWrite (D_green, HIGH);
      }
    }
  }
}
