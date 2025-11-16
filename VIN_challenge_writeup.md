# VIN challenge writeup 

## INTRODUCTION (PRE_REQUISITE) 

We use three terminals for this task; the first terminal is to open uds simulator (unified diagnostic service), which is used for communication with the ECU. In the second terminal, use candump to see the messages that are being passed between the  ECU and the simulator. In the third terminal, use cansend to request the ECU to send the VIN of the vehicle.

## The cansend format:

The first digits give the arbitration ID, then a '#', after that comes the number of packets, then the amount of data transmitted in bytes, followed by the data.

## Explanation of the commands used:

Here, since 73B is the reply ID of the ECU, then the sending ID of the ECU should be 733. (73B-8 gives 733)(HEX subtraction). So we use 

733#03022F190000000000 

Here, 733 is the ECU ID, 22 is the service ID, which is known as read data by identifier service, and F190 is the identifier for VIN of a vehicle used in UDS. When I sent this command, it gave 

vcan1 TX - - 733 [8] 03 22 F1 90 00 00 00 00 


vcan1 TX B E 73B [8] 10 14 62 F1 90 4C 55 41

Here 10 14 62 F1 90 4C 55 4,1, the first three nibbles are data length, 62 is 22 +8 gives 62 a service id. F1 90 specifies that it is giving the VIN number, the last three are the first three bytes of the VIN number. Next, we give a cansend command 

cansend vcan1 733#30000000000000000000 


Here we respond by flow control using '3' which tells the ECU to give the remaining data also without waiting. (all at once) Then it gives vcan1 TX - - 73B [8] 21 55 32 41 55 42 33 47 (Here 21 represents it is first in the sequence) vcan1 TX - - 73B [8] 22 45 33 38 33 34 36 37 (Here 22 represents it is second in the sequence). The rest are VIN numbers, so then I used

echo "" |xxd -r -p | hexdump -C to convert the hex character to ASCII values


## Info on ECU:
### Packet structure:
UDS uses a protocol called isotp. These isotp packets are split into 2 sections:
Protcol control unit:(1 byte)
Give info on:
1. What data
2. How much data
3. If there are multiple packets, 
protocol data unit:(7 Bytes)
gives info on the data

### Types of ISOTP message:
It is given by the first nibble of the byte
If the first nibble = 0 entire message can be stored in one frame
If the first nibble = 1 or 2 it means the message is stored in consecutive frames

### Common code in ECU
0x19 - Read Diagnostic Trouble Code (DTC)
0x11 - ECU Reset
0x22 - Read Data by Identifier
0x31 - Routine Control

### To find sender and receiver ID
We can either use the arbitration ID of the ECU to subtract 8 from it, or use the service ID of the ECU to subtract 8 from it, because sometimes the arbitration ID could be different in vehicles.

### Error in UDS request
It is always a 3-byte error; the first byte of the data will be a 7F, indicating an error, the second will be the service identifier the third will be the error code.
Some common error codes:
1. 0x10-General reject
2. 0x11-Service not supported
3. 0x31-Request out of range
