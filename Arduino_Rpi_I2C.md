Raspberry Pi and Arduino Connected Using I2C
============================================

Voltage level issue
-------------------
###Direct connection###
The Raspberry Pi is running at 3.3 Volts while the Arduino is running at 5 Volts. 
There are tutorials suggest using a level converter for the I2C communication. 
This is **NOT** needed if the Raspberry Pi is running as "master" and the Arduino is running as "slave".

The reason it works is because the Arduino does not have any pull-ups resistors installed, 
but the P1 header on the Raspberry Pi has 1k8 ohms resistors to the 3.3 volts power rail. 
Data is transmitted by pulling the lines to 0v, for a "high" logic signal. 
For "low" logic signal, it is pulled up to the supply rail voltage level. 
Because there is no pull-up resistors in the Arduino and because 3.3 volts is within the 
"low" logic level range for the Arduino everything works as it should.

RPI          |    | Arduino (Uno/Duemillanove)
-------------|----|-------------------------
GPIO 0 (SDA) |<-->| Pin 4 (SDA)
GPIO 1 (SCL) |<-->| Pin 5 (SCL)
Ground       |<-->| Ground

###Option of using level shifter###

RPI            |             |Arduino (Uno/Duemillanove)
---------------|-------------|--------------
GPIO0 (SDA) -- | TX1  -- TX0 | -- A4 (SDA)
GPIO1 (SCL) -- | RX0  -- RX1 | -- A5 (SCL)
3.3V        -- | LV   -- HV  | -- 5V
GND         -- | GND  -- GND | -- GND

Arduino as Slave
----------------

Configure Arduino As Slave Device For I2C

Load this sketch on the Arduino. We basically define an address for the slave (in this case, 4)
and callback functions for sending data, and receiving data. When we receive a digit, 
we acknowledge by sending it back. If the digit happens to be '1', we switch on the LED.

This program has only been tested with Arduino IDE 1.0.

```cpp
#include <Wire.h>

#define SLAVE_ADDRESS 0x04
int number = 0;
int state = 0;

void setup() {
  pinMode(13, OUTPUT);
  Serial.begin(9600); // start serial for output
  // initialize i2c as slave
  Wire.begin(SLAVE_ADDRESS);

  // define callbacks for i2c communication
  Wire.onReceive(receiveData);
  Wire.onRequest(sendData);

  Serial.println("Ready!");
}

void loop() {
  delay(100);
}

// callback for received data
void receiveData(int byteCount){

  while(Wire.available()) {
    number = Wire.read();
    Serial.print("data received: ");
    Serial.println(number);

    if (number == 1){

      if (state == 0){
        digitalWrite(13, HIGH); // set the LED on
        state = 1;
      }
      else{
        digitalWrite(13, LOW); // set the LED off
        state = 0;
      }
    }
  }
}

// callback for sending data
void sendData(){
  Wire.write(number);
}
```

Configure Raspberry Pi As Master Device
---------------------------------------

Since we have a listening Arduino slave, we now need a I2C master.

I have written this testing program in Python. This is what it does:
the Raspberry Pi asks you to enter a digit and sends it to the Arduino, 
the Arduino acknowledges the received data by send the exact same number back.

In the video, I used a built-in programming tool called “IDLE” in Raspberry Pi for compiling.

```python

import smbus
import time
# for RPI version 1, use "bus = smbus.SMBus(0)"
bus = smbus.SMBus(1)

# This is the address we setup in the Arduino Program
address = 0x04

def writeNumber(value):
  bus.write_byte(address, value)
  # bus.write_byte_data(address, 0, value)
  return -1

def readNumber():
  number = bus.read_byte(address)
  # number = bus.read_byte_data(address, 1)
  return number

while True:
  var = input(“Enter 1 – 9: “)
  if not var:
    continue

  writeNumber(var)
  print "RPI: Hi Arduino, I sent you ", var
  # sleep one second
  time.sleep(1)

  number = readNumber()
  print "Arduino: Hey RPI, I received a digit ", number
  print ""
```

References
----------
https://oscarliang.com/raspberry-pi-arduino-connected-i2c/
