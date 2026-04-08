Ladder logic:

purpose: take input process and give output 

instead of plc we had relay logic, this bridges the gap.

left-input and logic +ve rail
right-output -ve rail

flow of electricity 
contact flow - allows power like switch 
types of contact:
open contact - standard
closed contact 
positive edge - react when from flase to true
negative edge - react when from true to false

types of coil
coil - standard
negated coil closed
set coil when true it set something high but lose nothing
reset coil when false it sets something but get power it wont do anything
these both are both part of same thing


AND
OR
XOR
FUNCTION BLOCK
timer example 
https://realpython.com/python-sockets/#multi-connection-client-and-server
# Traffic Light
<img width="1920" height="920" alt="image" src="https://github.com/user-attachments/assets/79c7804c-6587-4cc1-b1bb-01243cdf2f1d" />
[https://app.plcsimulator.online/](https://app.plcsimulator.online/)

# SOCKET PROGRAMMING:
## Creating a socket object:
1. connecting two nodes of a network. one socket listens while other advertises
```
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```
AF_NET is the address family, i.e tells the system how to find the destination, types:
1.AF_INET (IPv4)-32-bit addresses-192.168.1.1
2.AF_INET6 (IPv6)-newer version of IP addresses-2001:0db8:85a3...
3. AF_UNIX (Unix Domain Sockets)-Used for communication between two processes on the same machine.
SOCK_STREAM is the socket type,i.e tells the system how to send the data, which is specifically for the TCP protocol, Reliable, ordered, two-way, connection-based.

## Preparing for server to connect:
```server_address = ('localhost', 12345)```
This says the connection will be listent from local host(the computer) and 12345 will be the port that the client had to use to communicate with the server, and is the port the server listen to and lock on.
```sock.bind(server_address)```




# TCP SERVER
```
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_address = ('localhost', 12345) #port is in between 1 to 65535 
sock.bind(server_address) #bind expects a tuple the values depend on the type o>
sock.listen()

print('Server started. Waiting for a connection...')
connection, client_address = sock.accept() #complete connection in server side
with connection:
    print('Connection from:', client_address)
    while True:
        data = connection.recv(1600)
        if data:
            text = data.decode('utf-8')
            print(f'Received: "{text}"')

            response = text + ' - this was the message received'
            connection.sendall(response.encode('utf-8'))
        else:
            break
print('Closing server')
sock.close()
```
# TCP CLIENT
```
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_address = ('localhost', 12345) #port is in between 1 to 65535 

print('Client started. Waiting for a connection...')
sock.connect(client_address) #complete connection in server side
message = "Hello World"
print('Sending: "%s"' % message)
sock.sendall(message.encode('utf-8'))

data = sock.recv(1600)
print('Received from server: "%s"' % data.decode('utf-8'))

print('Closing client')
sock.close()


# This line pauses the code until something happens on ANY socket in 'inputs'
readable, writable, exceptional = select.select(inputs, [], [])
```
# MULTIPLE CLIENT MODEL:

There are three methods: either threads, or use asynchronous input output, other is .select()
Which helps in finding multiple input output completion of more than one socket at the same time. 
Three sets of select()
Read set: check for incoming data
Write set: check if ready to receive data
Exception set: for error
# MULTIPLE TCPSERVER:
```
import socket
import threading

IP=socket.gethostbyname(socket.gethostname())
PORT=1107
ADDRESS=(IP,PORT)



def handle_client(conn,addr):
    print(f"New connection made:{addr}")
    
    while True:
        msg=conn.recv(1600).decode("utf-8")
        if msg=="Disconnect":
            break
        print(f"{addr}: {msg}")
        msg=msg+"Message recieved"
        conn.send(msg.encode("utf-8"))
    conn.close()

def main():
    server=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(ADDRESS) #takes ip address and port in a tuple
    server.listen()
    
    while True:
        conn,addr=server.accept()
        thread = threading.Thread(target=handle_client,args=(conn,addr)) # For >
        thread.start()
        print(f"Active connection: {threading.activeCount()-1}") # we do minus >


if __name__=="__main__":
    main()
```

# TCP CLIENT MULTIPLE
```
import socket
IP=socket.gethostbyname(socket.gethostname())

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_address = (IP, 1107) #port is in between 1 to 65535 
print('Client started. Waiting for a connection...') 
sock.connect(client_address) #complete connection in server side


while True:
        print("Type Disconnect if you wanna end the connection...")
        message = input("Write your message: ")
        sock.send(message.encode('utf-8'))
        if message == "Disconnect":
                break
        print(f"Sending: {message}")
        data = sock.recv(1600)
        print('Received from server: "%s"' % data.decode('utf-8'))

print('Closing client')
sock.close()

```

# TCP MODBUS SERVER
```
import socket
from pymodbus.server import StartTcpServer
from pymodbus.datastore import ModbusSlaveContext, ModbusServerContext
from pymodbus.datastore import ModbusSequentialDataBlock  

IP=socket.gethostbyname(socket.gethostname())
PORT=1107
ADDRESS=(IP,PORT)

store=ModbusSlaveContext(zero_mode=True)
context=ModbusServerContext(slave=store,single=True)

print(f"New connection is made in {IP}:{PORT}")

StartTcpServer(context,ADDRESS)
```
# Modbus server
```
#databank method is only for the server modserv.py
#!/bin/python

from pyModbusTCP.server import ModbusServer, DataBank
from time import sleep
# Create an instance of ModbusServer
server = ModbusServer("127.0.0.1", 1109, no_block=True)
# the no_block is for the server.start() statement

try:
    print("Start server...")
    server.start() #the script stops here to listen to w connections if set_block=False
    print("Server is online")
    state = [0] # initial value of the register is 0, later when value is changed, this value is updated
# state is a list since the get_words() return as list
    while True:
        if state != DataBank.get_words(1):
            state = DataBank.get_words(1)
            print(f"New Value Received: {state[0]}")
        sleep(0.5) #required to stop the system from overheating

except:
    print("Shutdown server ...")
    server.stop()
    print("Server is offline")
# single register: set_words(0,[55]) 
# multiple register: set_words(10,[1,2,3]) writes on next next register, that is on 10 and 11
# clear registers: set_words(0,[0]*10)
# register numbers start from 0 to 100
# the input integer that we can write is 16 bit value i.e 65,535 in hex 0xffff
# try expect is to close the connection smoothly, except block is triggered with keyboard interrupt
```

# TCP MODBUS CLIENT
```
from pyModbusTCP.client import ModbusClient
client= ModbusClient(host="127.0.0.1",port=1109,unit_id=1, auto_open=True)
regs = client.read_holding_registers(0, 2)
regs1 = client.read_holding_registers(10,2)

if regs:
    print(regs)
else:
    print("read error")
if client.write_multiple_registers(10, [44,55]):
    print("write ok")
else:
    print("write error")
if regs1:
        print(regs1)
else:
        print("read error")
```




UDP: https://dl.acm.org/doi/pdf/10.1145/1409908.1409911
