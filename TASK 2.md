# TASK 2
## FIRST TASK:
Description:
The first task was to make a set of LEDs blink. To avoid hardcoding, I stored the pin numbers in an array for easier access. The following functions were used:
pinMode() – to configure the respective pins as outputs.
digitalWrite() – to switch the LEDs on and off.
delay() – to introduce a pause before executing the next instruction.
I implemented two for loops: the first loop lights up the LEDs sequentially from left to right, while the second loop turns them on in reverse order, from right to left. This creates a simple running light effect. The complete code is provided below.

void setup() {
  int q[5];
  q[0]=26;
  q[1]=27;
  q[2]=14;
  q[3]=12;
  q[4]=13;
  for (int i=0;i<5;i++){
    pinMode(q[i], OUTPUT);
  }
 
}


void loop() {
  int q[5];
  q[0]=26;
  q[1]=27;
  q[2]=14;
  q[3]=12;
  q[4]=13;
  for (int i=0;i<5;i++){
    digitalWrite(q[i], HIGH);
    delay(1000);
    digitalWrite(q[i],LOW);
  }
 
  for (int i=4;i>-1;i--){
    digitalWrite(q[i], HIGH);
    delay(1000);
    digitalWrite(q[i],LOW);
  }
 
}






## Second task:
Description:
The task was to display digits from 0 to 9 on a seven-segment display. To simplify the program, I created an array to store the pin numbers, allowing easy access and control of each segment. I used another array to hold the binary values corresponding to each digit, where 1 turns a segment on and 0 turns it off. Since each segment of the display is associated with a specific pin, activating the correct pattern of 1s and 0s produces the desired number. To implement this, I used two nested for loops: the outer loop iterates through the digits (0–9), while the inner loop iterates through the binary values for each digit, switching the respective pins on or off to display the number.



int ports[]={2,4,14,12,13,5,18};
void setup() {
  for (int i=0;i<7;i++){
    pinMode(ports[i], OUTPUT);
  }


}
byte digits[10][7] = {
  {1,1,1,1,1,1,0}, // 0
  {0,1,1,0,0,0,0}, // 1
  {1,1,0,1,1,0,1}, // 2
  {1,1,1,1,0,0,1}, // 3
  {0,1,1,0,0,1,1}, // 4
  {1,0,1,1,0,1,1}, // 5
  {1,0,1,1,1,1,1}, // 6
  {1,1,1,0,0,0,0}, // 7
  {1,1,1,1,1,1,1}, // 8
  {1,1,1,1,0,1,1}  // 9
};






void loop() {
  for(int num=0; num<10; num++) {
    for(int i=0; i<7; i++) {
    digitalWrite(ports[i], digits[num][i]);
  }
    delay(1000);
  }
}


Third task:
Description:
The circuit uses a 5V battery as the power supply, with VSS (GND) connected to the breadboard's black rail and VDD (+5V) to the red rail. A potentiometer is used to adjust the LCD contrast, while data pins DB0–DB3 are tied to GND to ensure they always read 0. Data pins DB4–DB7 are connected to pushbuttons through resistors, enabling manual input to the LCD. Control pins RS, RW, and E are properly connected to manage LCD operations, and the resistors help limit current and protect the circuit. This configuration creates a 4-bit manual input interface for the LCD using pushbuttons.


Fourth task:
Description:
In this task, I included the libraries ESP32Servo.h and stdio.h to enable servo functionality. I declared servo objects using the Servo <servoname> syntax and initialized them with <servoname>.attach(pin). For communication between the user and the microcontroller, I used Serial.begin(9600), where 9600 represents the baud rate. The Serial.available() function checks whether input data has been received, while parseInt() extracts only the numerical values from the input and stores them in a variable. Additionally, I used the map() function, which takes four parameters: the value to be mapped, the input range (start and end values), and the output range (servo angle limits). This allows the input values to be scaled appropriately to control servo movement.

#include <ESP32Servo.h>
#include<stdio.h>
Servo minuteservo;
Servo hourservo;
void setup() {
  Serial.begin(9600);
  minuteservo.attach(37);
  hourservo.attach(36);
  Serial.println("Enter hours and minutes");
}
void loop() {
  int hours=0;
  int minutes=0;
  if (Serial.available()>0){
    hours=Serial.parseInt();
    minutes=Serial.parseInt();
    Serial.print("You typed hours ");
    Serial.print(hours);
    Serial.print(" and ");
    Serial.print(minutes);
  }
  minuteservo.write(0);
  hourservo.write(0);
  delay(5000);
  int minangle=map(minutes,0,60,0,180);
  minuteservo.write(minangle);
  int hourangle=map(hours,0,12,0,180);
  hourservo.write(hourangle);
  delay(10000);
  /* Tried a sample code to see how servo moves
  servo.write(0);
  delay(1000);
  servo.write(90);
  delay(1000);
  servo.write(180);
  delay(1000);
  servo.write(270);
  delay(1000);*/
}

Fifth task
1.Blink in built led
Description:
In this code, the #define function is used to associate a pin with its corresponding memory address. The statement DDRB |= (1 << DDB5); configures pin 5 of Port B as an output. Here, DDRB represents the Data Direction Register for Port B, and the bitwise OR (|) operation ensures that only bit 5 is modified without affecting the other bits. The expression 1 << DDB5 shifts the value 1 five positions to the left, thereby setting bit 5 to 1. The for loop is used as an alternative to a delay function, creating a simple time pause in execution. Finally, the statement PORTB &= ~(1 << PORTB5); resets bit 5 back to 0, turning the output low while preserving the other bits in Port B.
#define DDRB*((volatile uint8_t*)0x24)
#define PORTB*((volatile uint8_t*)0x25)
//#include <util/delay.h>
void setup() {
  DDRB|=(1<<DDB5);
}


void loop() {
  PORTB|=(1<<PORTB5);
  //_delay_ms(500);
  for(volatile long i=0;i<1000000;i++){PORTB=32;}
  PORTB|=~(1<<PORTB5);
  //_delay_ms(500);
  for(volatile long i=0;i<1000000;i++){PORTB=0;}
}



https://github.com/Madhubala11-09/BIOS_HARDWARE_TASK_2.git

