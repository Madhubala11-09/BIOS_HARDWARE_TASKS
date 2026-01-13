# Types of communication methods:
1. UART
2. I2C
3. SPI


# Intro
Electronic devices talk to each other by sending bits of data through wires physically connected between devices.In a system operating at 5 V, a 0 bit is communicated as a short pulse of 0 V, and a 1 bit is communicated by a short pulse of 5 V.

## Types:

### serial and parallel
In parallel communication, the bits of data are sent all at the same time, each through a separate wire.In serial communication, the bits are sent one by one through a single wire.

### synchronous and asynchronous
Any communication protocol where devices share a clock signal is known as synchronous. SPI is a synchronous communication protocol. There are also asynchronous methods that don’t use a clock signal. For example, in UART communication, both sides are set to a pre-configured baud rate that dictates the speed and timing of data transmission.

# UART communication
Universal Asynchronous Receiver-Transmitter. both devices must be pre-configured to operate at the same speed, known as the Baud Rate.Since there is no clock to tell the receiver when data is arriving, UART uses a specific packet structure to wrap the data. When no data is being sent, the line is held at a "High" voltage.`
## HARDWARE SETUP:
1. TX of first arduino to RX of another
2. RX of first arduino to TX of another
3. GND of first arduino to GND of another

## WORKING:
A standard UART frame consists of:


Start Bit: The sender pulls the line Low (Logic 0) for one bit-period to "wake up" the receiver.


Data Frame: Usually 5 to 9 bits (typically 8 bits or 1 byte). This is the actual information being sent.


Parity Bit: An optional bit used for simple error checking (even or odd parity).


Stop Bit(s): The sender pulls the line High (Logic 1) for 1 or 2 bit-periods to signal the end of the packet.

## Code:
### transmitter: 

void setup() {
  Serial.begin(9600);

}

void loop() {
  Serial.println("Hello");
  delay(1000);
}

### receiver:

void setup() {
  Serial.begin(9600);
  Serial.begin(9600);

}

void loop() {
  if (Serial.available()){
    char message=Serial.read();
    Serial.println(message);
  }
  
}

# I2C communication
## HARDWARE SETUP:
1. SDA 
2. SCL
3. GND
4. 5V
## WORKING:
Start Condition: The Controller pulls the SDA line low while SCL is still high. This tells everyone on the bus, "Pay attention!"


Address Frame: The Controller sends the 7-bit address of the specific device it wants to talk to, plus a bit to say if it wants to Read or Write.


ACK/NACK: The device with that address pulls the SDA line low to say, "I'm here!" (Acknowledgment).


ACK: ACKnowledgement
In I2C: The receiver pulls the SDA line Low during the 9th clock pulse.


NACK (or NAK): Negative ACKnowledgement (sometimes also called "Non-Acknowledgement")
In I2C: The receiver leaves the SDA line High (Idle) during the 9th clock pulse.


Data Frames: Data is sent 8 bits at a time. After every byte, the receiver must send an ACK bit.


Stop Condition: The Controller releases the lines to signal the end of the conversation.
Up to 127 devices can be connected.

## code
### transmitter:

#include<Wire.h>

#define peripher_arduino 8

void setup(){
  Wire.begin();
}


void loop() {
  Wire.beginTransmission(peripher_arduino);
  Wire.write("Hello");
  Wire.endTransmission();
  delay(500);
  
}
### receiver:

#include<Wire.h>

#define peripher_arduino 8

void setup(){
  Wire.begin(periph0er_arduino);
  Wire.onReceive(doing);
  Serial.begin(115200);
  Serial.println("Reciever is ready");
  
}

void doing(int howmany){
  while(Wire.available()){
    char c=Wire.read();
    Serial.print(c);
  }
  Serial.println();
}

void loop(){
  
}

# SPI communication
## HARDWARE SETUP:
1. connect d13
2. connect d12
3. connect d11
3. connect d10 of master to d10 of slave1
4. connect d9 of master to d10 of slave2
5. connect gnd

## CODE:
### MASTER:
#include <SPI.h>

#define SLAVE1 10   
#define SLAVE2 9    

void setup() {
  Serial.begin(9600);

  pinMode(SLAVE1, OUTPUT);
  pinMode(SLAVE2, OUTPUT);

  digitalWrite(SLAVE1, HIGH);
  digitalWrite(SLAVE2, HIGH);

  SPI.begin(); 
}

void loop() {

  // Talk to SLAVE 1
  digitalWrite(SLAVE1, LOW);
  byte r1 = SPI.transfer(7);
  digitalWrite(SLAVE1, HIGH);

  Serial.print("Slave 1 replied: ");
  Serial.println(r1);

  delay(1000);

  // Talk to SLAVE 2
  digitalWrite(SLAVE2, LOW);
  byte r2 = SPI.transfer(3);
  digitalWrite(SLAVE2, HIGH);

  Serial.print("Slave 2 replied: ");
  Serial.println(r2);

  delay(2000);
}
### SLAVE1:
#include <SPI.h>

volatile byte received;

void setup() {
  Serial.begin(9600);
  pinMode(MISO, OUTPUT);
  SPCR |= _BV(SPE);       // Enable SPI
  SPI.attachInterrupt();
}

ISR (SPI_STC_vect) {
  received = SPDR;
  SPDR = received + 1;   // Reply
}

void loop() {
  Serial.print("Slave 1 got: ");
  Serial.println(received);
  delay(500);
}
### SLAVE2:
#include <SPI.h>

volatile byte received;

void setup() {
  Serial.begin(9600);
  pinMode(MISO, OUTPUT);
  SPCR |= _BV(SPE);
  SPI.attachInterrupt();
}

ISR (SPI_STC_vect) {
  received = SPDR;
  SPDR = received ;   // Different reply
}

void loop() {
  Serial.print("Slave 2 got: ");
  Serial.println(received);
  delay(500);
}
## WORKING:
### External parts:
MOSI (Master Output/Slave Input) – Line for the master to send data to the slave.


MISO (Master Input/Slave Output) – Line for the slave to send data to the master.


SCLK (Clock) – Line for the clock signal.


SS/CS (Slave Select/Chip Select) – Line for the master to select which slave to send data to.


The clock signal synchronizes the output of data bits from the master to the sampling of bits by the slave. One bit of data is transferred in each clock cycle, so the speed of data transfer is determined by the frequency of the clock signal. SPI communication is always initiated by the master since the master configures and generates the clock signal.
The master can choose which slave it wants to talk to by setting the slave’s CS/SS line to a low voltage level. In the idle, non-transmitting state, the slave select line is kept at a high voltage level.


The master sends data to the slave bit by bit, in serial through the MOSI line. The slave receives the data sent from the master at the MOSI pin. Data sent from the master to the slave is usually sent with the most significant bit first.



The slave can also send data back to the master through the MISO line in serial. The data sent from the slave back to the master is usually sent with the least significant bit first.


### Internal parts:

#### A. The Shift Register
The most important thing to realize about SPI is that it doesn't "send" data in the traditional sense—it "exchanges" it. Imagine the Master has the letter 'A' and the Slave has the letter 'B'. After 8 clock pulses:


The Master's 'A' has moved through the MOSI wire and is now sitting in the Slave's register.


The Slave's 'B' has moved through the MISO wire and is now sitting in the Master's register.


Key takeaway: Every time you send a byte, you receive a byte. Even if the Master just wants to send data and doesn't care about the Slave's response, the hardware still performs this swap.

#### B. The Clock Generator
Only the Master uses this part. It takes the main system clock (16MHz on an Uno) and divides it down using a "Prescaler" to a speed the Slave can handle. This is what you control using the SPR0 and SPR1 bits in the SPCR register.

#### C. The Control Logic & Registers
These are the "brain" of the SPI module. They monitor the pins and manage the data:
SPCR (Control Register): The settings dashboard (Speed, Mode, Master/Slave).
SPSR (Status Register): Tells you if the transfer is finished or if there was an error.
SPDR (Data Register): This is the "mailbox." When you write SPDR = 'A';, the hardware takes that 'A', puts it in the Shift Register, and starts the clock.

##### SPCR
1. SPIE (SPI Interrupt Enable) - Bit 7

0: SPI interrupts are disabled.


1: SPI interrupts are enabled.


What it does: If this is 1, the Arduino will automatically jump to an Interrupt Service Routine (ISR) the moment a byte finishes transferring. This is great for Slaves so they don't have to "poll" the status register constantly.In programming, "polling" is like a kid in the backseat of a car constantly asking, "Are we there yet? Are we there yet?"

2. SPE (SPI Enable) - Bit 6

0: SPI hardware is off.


1: SPI hardware is on.


What it does: This is the main power switch. When you set this to 1, pins 10, 11, 12, and 13 on your Uno stop acting like normal digital pins and start acting as SS, MOSI, MISO, and SCK.

3. DORD (Data Order) - Bit 5

0: MSB (Most Significant Bit) is sent first.


1: LSB (Least Significant Bit) is sent first.


What it does: It decides if the "start" of your byte is the left side (128) or the right side (1). Most devices use MSB (0).

4. MSTR (Master/Slave Select) - Bit 4

0: Arduino acts as a Slave.


1: Arduino acts as the Master.


What it does: This is the most important setting. The Master controls the clock (SCK). The Slave just listens.

5. CPOL (Clock Polarity) - Bit 3

0: SCK is LOW when idle.


1: SCK is HIGH when idle.


What it does: It sets the "starting state" of the clock wire.

6. CPHA (Clock Phase) - Bit 2

0: Data is sampled on the Leading (first) edge of the clock.


1: Data is sampled on the Trailing (second) edge of the clock.


What it does: Along with CPOL, this determines the "SPI Mode" (0, 1, 2, or 3).

7. SPR1 and SPR0 (SPI Clock Rate) - Bits 1 & 0

What they do: These two bits work together to set the speed of the clock (only for the Master).

