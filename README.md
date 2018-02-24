# Arduino DCC Wilde Maus

## Schaltplan

Das IC der Lauflichtssteuerung muss herrausgenommen werden!

Auf der Adresse **DCC 7** kann mit diesen Code über die F1 bis F9 Tasten die gesammte Wilde Maus gesteuert werden.

|F Taste|Funktion|Pin|
|:-:|:-:|:-:|
|F1|Motor|A0|
|F2|WildeMaus Anzeige An/Aus|A1|
|F3|Birnchen Teil 1|A2|
|F4|Birnchen Teil 2|A3|
|F5|WildeMaus Anzeige Bilnken|A1|
|F6|Laufliche Mode 1|D3,D4,D5|
|F7|Laufliche Mode 2|D3,D4,D5|
|F8|Laufliche Mode 3|D3,D4,D5|
|F9|Laufliche Mode 4|D3,D4,D5|

![Bild](https://raw.githubusercontent.com/ToWipf/DCC-WildeMaus-Arduino/master/WildeMaus_3.PNG)

## Aufbau der Wilden Maus

![Bild](https://raw.githubusercontent.com/ToWipf/DCC-WildeMaus-Arduino/master/WildeMaus.jpg)

``` c
/* 
Wilde Maus Lauflicht Wipf 27.01.2018 bis 24.02.18
DCC Function Decoder
Orginal Author: Ruud Boer - September 2015
This sketch turns an Arduino into a DCC function decoder for F0 - F12
Output pins used: 3-19 (14-19 = A0-A5)
The DCC signal is optically separated and fed to pin 2 (=Interrupt 0).
*/

int decoderAddress = 7; // decoder Addresse
#define F0_pin 13 //    Define the output pin for every Function number in use

#define F1_pin 14 // A0 / F1 Motor
#define F2_pin 15 // A1 / F2 WildeMaus Dauerlicht, F5 Blinken WildeMaus 
#define F3_pin 16 // A2 / F3 Licht 1 Häuschen
#define F4_pin 17 // A3 / F4 Licht 2 Laternen
#define F5_pin 18 // A4 / deaktiviert
#define F6_pin 19 // A5 / deaktiviert
#define F7_pin 3  // D3 / Lauflicht, F6,F7,F8,F9 verschiedene Modi
#define F8_pin 4  // D4 / Lauflicht, F6,F7,F8,F9 verschiedene Modi
#define F9_pin 5  // D5 / Lauflicht, F6,F7,F8,F9 verschiedene Modi
#define F10_pin 6 // D6 / deaktiviert
#define F11_pin 7 // D7 / deaktiviert
#define F12_pin 8 // D8 / deaktiviert

#include <DCC_Decoder.h>                      // Comment for 123d Test !!!
#define kDCC_INTERRUPT 0

byte Func[4]; // 0=L F4-F1, 1=F9-F5, 2=F12-F9, 3=F20-F13, 4=F28-F21
byte instrByte1;
int Address;
int lauflicht;
int blinken;
int merkerL1 = 0;
int merkerL2 = 0;
int merkerL3 = 0;
int merkerL4 = 0;

boolean RawPacket_Handler(byte pktByteCount, byte* dccPacket)
{
  Address = 0;
  if (!bitRead(dccPacket[0], 7))
  { //bit7=0 -> Loc Decoder Short Address
    Address = dccPacket[0];
    instrByte1 = dccPacket[1];
  }
  else if (bitRead(dccPacket[0], 6))
  { //bit7=1 AND bit6=1 -> Loc Decoder Long Address
    Address = 256 * (dccPacket[0] & B00000111) + dccPacket[1];
    instrByte1 = dccPacket[2];
  }

  if (Address == decoderAddress)
  {
    byte instructionType = instrByte1 >> 5;
    switch (instructionType)
    {
      case 4: // Loc Function L-4-3-2-1
        Func[0] = instrByte1 & B00011111;
        break;
      case 5: // Loc Function 8-7-6-5
        if (bitRead(instrByte1, 4))
        {
          Func[1] = instrByte1 & B00001111;
        }
        else { // Loc Function 12-11-10-9
          Func[2] = instrByte1 & B00001111;
        }
        break;
    }
    if (Func[0]&B00010000)
    {
      digitalWrite(F0_pin, HIGH);
    }
    else
    {
      digitalWrite(F0_pin, LOW);
    }
    if (Func[0]&B00000001)
    {
      digitalWrite(F1_pin, HIGH);
    }
    else
    {
      digitalWrite(F1_pin, LOW);
    }
    if (Func[0]&B00000010)
    {
      digitalWrite(F2_pin, HIGH);
    }
    else
    {
      digitalWrite(F2_pin, LOW);
    }
    if (Func[0]&B00000100)
    {
      digitalWrite(F3_pin, HIGH);
    }
    else
    {
      digitalWrite(F3_pin, LOW);
    }
    if (Func[0]&B00001000)
    {
      digitalWrite(F4_pin, HIGH);
    }
    else
    {
      digitalWrite(F4_pin, LOW);
    }
    if (Func[1]&B00000001)/////////F5
    {
      blinken++;
      if (blinken < 100)
      {
        digitalWrite(F2_pin, HIGH);
      }
      else
      {
        digitalWrite(F2_pin, LOW);
      }

      if (blinken > 200)
      {
        blinken = 0;
      }
    }
    else
    {
      //digitalWrite(F2_pin, LOW);
    }

    if (Func[1]&B00000010)//F6
    {
      if (merkerL1 == 0 && merkerL3 == 0 && merkerL4 == 0)
      {
        merkerL2 = 1;
        lauflicht++;
        if (lauflicht > 0 && lauflicht < 50)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 50 && lauflicht < 100)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 100 && lauflicht < 150)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 150)
        {
          lauflicht = 0;
        }
      }
    }
    else
    {
      if (merkerL1 == 0 && merkerL3 == 0 && merkerL4 == 0)
      {
        digitalWrite(F7_pin, HIGH);
        digitalWrite(F8_pin, HIGH);
        digitalWrite(F9_pin, HIGH);
      }
      merkerL2 = 0;
    }

    if (Func[1]&B00000100)//F7
    {
      if (merkerL2 == 0 && merkerL3 == 0 && merkerL4 == 0)
      {
        merkerL1 = 1;
        lauflicht++;
        if (lauflicht > 0 && lauflicht < 15)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 15 && lauflicht < 30)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 30 && lauflicht < 45)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 45)
        {
          lauflicht = 0;
        }
      }
    }
    else
    {
      if (merkerL2 == 0 && merkerL3 == 0 && merkerL4 == 0)
      {
        digitalWrite(F7_pin, HIGH);
        digitalWrite(F8_pin, HIGH);
        digitalWrite(F9_pin, HIGH);
      }
      merkerL1 = 0;
    }
    if (Func[1]&B00001000)//F8
    {
      if (merkerL2 == 0 && merkerL1 == 0 && merkerL4 == 0)
      {
        merkerL3 = 1;
        lauflicht++;
        if (lauflicht > 0 && lauflicht < 15)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 15 && lauflicht < 30)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 30 && lauflicht < 45)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 45)
        {
          lauflicht = 0;
        }
      }
    }
    else
    {
      if (merkerL1 == 0 && merkerL2 == 0 && merkerL4 == 0)
      {
        digitalWrite(F7_pin, HIGH);
        digitalWrite(F8_pin, HIGH);
        digitalWrite(F9_pin, HIGH);
      }
      merkerL3 = 0;
    }

    if (Func[2]&B00000001)//F9
    {
      if (merkerL2 == 0 && merkerL1 == 0 && merkerL3 == 0)
      {
        merkerL4 = 1;
        lauflicht++;
        if (lauflicht > 0 && lauflicht < 15)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 15 && lauflicht < 30)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 30 && lauflicht < 45)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 45 && lauflicht < 60)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 60 && lauflicht < 75)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 75 && lauflicht < 90)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 90 && lauflicht < 105)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 100 && lauflicht < 110)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 110 && lauflicht < 120)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 120 && lauflicht < 130)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 130 && lauflicht < 140)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 150 && lauflicht < 160)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }

        if (lauflicht > 160 && lauflicht < 180)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 180 && lauflicht < 200)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 200 && lauflicht < 220)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
       if (lauflicht > 220 && lauflicht < 240)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }
        if (lauflicht > 240 && lauflicht < 260)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 260 && lauflicht < 280)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }     
        if (lauflicht > 280 && lauflicht < 300)
        {
          digitalWrite(F7_pin, LOW);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 300 && lauflicht < 320)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, LOW);
          digitalWrite(F9_pin, HIGH);
        }
        if (lauflicht > 320 && lauflicht < 340)
        {
          digitalWrite(F7_pin, HIGH);
          digitalWrite(F8_pin, HIGH);
          digitalWrite(F9_pin, LOW);
        }      
        if (lauflicht > 340)
        {
          lauflicht = 0;
        }
      }
    }
    else
    {
      if (merkerL1 == 0 && merkerL2 == 0 && merkerL3 == 0)
      {
        digitalWrite(F7_pin, HIGH);
        digitalWrite(F8_pin, HIGH);
        digitalWrite(F9_pin, HIGH);
      }
      merkerL4 = 0;
    }
    if (Func[2]&B00000010)//F10
    {
      //Tut nichts
    }
    ///
    if (Func[2]&B00000100)
    {
      //Tut nichts
    }
    if (Func[2]&B00001000)
    {
      //Tut nichts
    }

    /*//Print DCC packet bytes for testing purposes
      for (byte n=0; n<5; n++) {
      Serial.print(Func[n],BIN);
      Serial.print(" ");
      }
      Serial.println(" ");
    */
  }
}

void setup() {
  Serial.begin(38400);
  DCC.SetRawPacketHandler(RawPacket_Handler);
  DCC.SetupMonitor( kDCC_INTERRUPT );
  for (byte n = 3; n < 20; n++) pinMode(n, OUTPUT);
}

void loop() {
  DCC.loop();
}
```
