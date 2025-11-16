#VIN challenge writeup 

##INTRODUCTION (PRE_REQUISITE) 

We use three terminals for this task, the first terminal is to open uds simulator (unified diagnostic service) which is used for communication with the ECU.In the second terminal use candump to see the messages that are being passed between ECU and the simulator. In the third terminal use cansend to request the ECU to send the VIN of the vehicle.

##The cansend format:

The first digits give the arbitration id then a '#', after that comes number of packets then, the amount of data transmitted in bytes followed by the data.

##Command we use explanation: 

Here since 73B is the reply id of the ECU then the sending id of the ECU should be 733. (73B-8 gives 733)(HEX subtraction)

So we use 733#03022F190000000000 here 733 is the ecu id then 22 is the service number then F190 is used to specify the VIN. When I sent this command it gives vcan1 TX - - 733 [8] 03 22 F1 90 00 00 00 00 vcan1 TX B E 73B [8] 10 14 62 F1 90 4C 55 41

Here 10 14 62 F1 90 4C 55 41 the first three nibbles are data length, 62 is 22 +8 gives 62 a service id. then F1 90 is to specify it is giving VIN number then the last three are the first three bytes of the VIN number.

Next we give a cansend command cansend vcan1 733#30000000000000000000 
Here we respond by flow control using '3' which tells the ecu to give the remaining data also without waiting. (all at once) then it gives vcan1 TX - - 73B [8] 21 55 32 41 55 42 33 47 (Here 21 represent it is first in the sequence) vcan1 TX - - 73B [8] 22 45 33 38 33 34 36 37 (Here 22 represents it is second in the sequence) The rest are VIN numbers so then I used

echo "" |xxd -r -p | hexdump -C to convert the hex character to ASCII values
