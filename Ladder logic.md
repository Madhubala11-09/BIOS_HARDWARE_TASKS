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
