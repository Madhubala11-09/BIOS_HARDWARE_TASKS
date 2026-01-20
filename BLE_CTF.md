# FLASHING:
 I followed instructions from this blog 
 https://mahmoudjadaan.blogspot.com/2025/09/bluetooth-low-energy-lab-1-hands-on-lab.html
 
 #Enable BLE on Linux
$ sudo rfkill unblock all
$ sudo btmgmt le on

 #Check that you have a bluetooth antenna device
$ hcitool dev
Devices:
	hci0	B8:27:EB:76:11:25

 #if necessary, restart bluetooth service
$ sudo service bluetooth restart
$ sudo service dbus restart

 #disable/enable bluetooth device
$ sudo hciconfig hci0 down
$ sudo hciconfig hci0 up

 #finding the MAC of your device
$ sudo hcitool lescan

# FLAG 1:

This task was just to find the score, once we check the score the blue light is turned on representing the device is connected. then we submit the flag.

# FLAG 2:

This task was to read handels i   learned handels are represented as ascii or hex values
so in gatttool the command for reading was just how e check the score, gatttool -b D0:EF:76:32:4F:66 --char-read -a 0x002e excluding the other parts.

# FLAG 3:

This task asked us to read a handle and the execute that command, when i read the handle it asked to change device name to MD5 md5 is a way to encrypt the data, so to encrypt it we use the command echo -n "<DEVICENAME>" | md5sum
echo -n "2b00042f7481c7b056c4b410d28f33c" | md5sum

then we submit the first 20 character of this result as the flag.

# FLAG 4:
1800 (Generic Access)
Everything starts with "gatttool -b <MAC_ADDRESS> -I" to get the interactive mode.
In interactive mode we use connect to connect with the device, and then primary to see the services.

device information service is with uuid 0x180A

UUID defines the properties and metadata of the attribute we’ll be accessing, whereas the handle gives the address of a particular attribute.

characteristics to access the characteristics of a service, we use attr handle and end grp handle along with characteristics

Manufacturer Name String Characteristic has a standard UUID of 0x2A29

custom service hosts all characteristics starting with handle 0x0028. The handles below this are part of the standard service. in the characterstics list we only have 15,19,2 when i checked each handle value seperately I got the flag.

# FLAG 5:
This task asked to read handle 32, when we do it told write anyhting here, so from the flag submission command i figured the syntax to write data hence, 
gatttool -b D0:EF:76:32:4F:66 --char-write-req -a 0x32 -n $(echo -n "Hello")

# FLAG 6:
This task was to write an ascii value into the handle 34 hence we use the same command as before  gatttool -b D0:EF:76:32:4F:66 --char-write-req -a 0x34 -n $(echo -n "yo")

# FLAG 7:
This task was to write an hex value 0x007 into the handle 36 hence we use the command
gatttool -b D0:EF:76:32:4F:66 --char-write-req -a 0x36 -n 07

# FLAG 8:
This task was to write and reading handles in a different method. I learnt that in gatttool we can represent the handle as both hex and decimal the similar way or command as above worked
gatttool -b D0:EF:76:32:4F:66 --char-write-req -a 58 -n c9

# FLAG 9:
This task was related to bash scripting 
we have to write the hex value from 0 to ff that is 255 values and read the handle after writing to reveal the flag, here the correct hex value to reveal flag is not known hence we brute force it by writing a bash script.

MAC="D0:EF:76:32:4F:66"
HANDLE="0x003c"

for i in {0..255}; do
    HEX_VAL=$(printf '%02x' $i)
    
    echo "Trying hex value: $HEX_VAL"
    gatttool -b $MAC --char-write-req -a $HANDLE -n $HEX_VAL
    gatttool -b $MAC --char-read -a $HANDLE
done


# FLAG 10:          
MAC="D0:EF:76:32:4F:66"
HANDLE="0x003e"

for i in {1..1000}; do
    echo "Reading time: $i"
    gatttool -b $MAC --char-read -a $HANDLE
done



# FLAG 11:
This task was to listen a particular handle for a single notification, when i went through the tools that gatttool have to offer with the help command i found a option called --listen when i searched for syntax in google i figured that this should be used when we write something in the handle also we use 0100 to get notification hence the command is,
gatttool -b D0:EF:76:32:4F:66 --char-write-req -a 0x0040 -n 0100 --listen

# FLAG 12:
This task was to listen for a single indication, this time 0200 for indication and 0300 for both indication and notifications the command we use is:
 gatttool -b D0:EF:76:32:4F:66 --char-write-req -a 0x0044 -n 0200
 
# FLAG 13:
This is to get multiple notification the same command as before challenge will work,
in the return messages the second one is the flag. the command we use is gatttool -b  D0:EF:76:32:4F:66 --char-write-req -a 0x46 -n 0300 --listen

# FLAG 14:
This task is to get multiple indication the same command works, in the return message the second one is the flag. the command we use is gatttool -b  D0:EF:76:32:4F:66 --char-write-req -a 0x4a -n 0100 --listen

# FLAG 15: Unsupported 

# FLAG 16:
This task requires us to connect to a particular mtu. Open the interactive mode of gatttool then we used mtu 247 then while we read the flag handle it gave Set your connection MTU to 444, so when i changed to 444 i got the flag.

ATT Maximum Transmission Unit (MTU) is the maximum length of an ATT packet.

# FLAG 17:

This task was to write and respond 
gatttool -b D0:EF:76:32:4F:66 --char-write-req -a 0x50 -n $(echo -n "hello" | xxd -p) --listen

# FLAG 18:
I just listened after writing and got the flag. 

# FLAG 19: 
This task was to figure out the flags parts hidden in different property handles
Here 0x02 means readable 0x08 means writable 0x0a means both read and write EXTENDED PROPERTIES (0x80), NOTIFY (0x10), WRITE (0x8), and READ (0x2)

# FLAG 20:
It was the md5sum of the authors twitter profile id

# GATTTOOL BASICS:

ATT is a wire application protocol, while GATT dictates how ATT is employed in service composition.
ATT- attribute - three elements- 16 bit handle uuid value of fixed length

Everything starts with "gatttool -b <MAC_ADDRESS> -I" to get the interactive mode.

You read values using --char-read and specify the handle address with -a.
You write values using --char-write and specify the handle address with -a.

To find the Manufacturer Name of a certain device "hcitool lescan" to do a scan of BLE devices and to find the address of your device.

# BLE BASICS:

## BLE stack:
### APPLICATION:
It uses the APIs (Program telling the device what to do) provided by the host to send and receive information.
Manages the Profiles.Takes raw hexadecimal values received from the ATT table and converts them into something meaningful

### HOST:
this contains the GATT,GAP,ATT,SMP
GATT-
ATT- Atribute protocol:
Is it like rule for communication

GAP- Generic access profile:
It explains how the device tells its presence, and defines server and client.
Server is the ble device it advertise and waits for client to connect
Client is the laptop it scans and intiates the connection

SMP-


GATT Server (usually the Peripheral) holds the data. The GATT Client (usually the Central) wants to access it. But before any conversation can happen, the client must first figure out what topics the server can discuss.
## Client-Initiated Operations
Read
Write
Write without Response
## Server-Initiated Operations
Notify
Indicate

Characteristics are fragments of a larger Attribute Table.

Handles are the unique keys used to jump to a specific row.

UUIDs define the "meaning" of the data in that row.

Hierarchy in ATT protocol

1. profile 
2. services
3. characteristics
4. attributes 

## Profile 
"Profile" is the complete collection of all Services and Characteristics on the device.

## services
A Service is a logical grouping of related data. It doesn't hold data itself; it acts as a container or a folder.It starts with a special attribute called the Service Declaration Followed by Characteristic Definition Attributes

Services --> service declaration --> Character definition --> character definition's value --> characteristic property, characteristic value, characteristic UUID, characteristic descriptors. 

## Characteristics
A Characteristic is the actual piece of data you want to interact with, that is group of attributes. It is stored as a table. It lives inside a Service. Every characteristic has three distinct parts:

Declaration: The metadata (What is this? Can I read it? Can I write to it?).
Value: The actual data - ATTRIBUTE
Descriptors: Optional extra information

## Attributes
1. handle - Address - 2 bytes
2. UUID - Kind of data 
3. Permission
4. Value

## UUID - Universally Unique Identifier
Long: 128-bit UUID (Vendor Specific): custom UUID you generate for your own proprietary services and characteristics.
Shore: 16-bit UUID (Bluetooth SIG Defined) standardized numbers for common functions. All official services and characteristics use these.
