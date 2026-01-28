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

# Traffic Light
<img width="1920" height="920" alt="image" src="https://github.com/user-attachments/assets/79c7804c-6587-4cc1-b1bb-01243cdf2f1d" />
[https://app.plcsimulator.online/](https://app.plcsimulator.online/)


# TCP SERVER
```
import socket

# Create a TCP/IP socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# AF_INET is the IPv4 that is
#SOCK_STREAM is to specify socket type
# Bind the socket to the port
server_address = ('localhost', 10000)
print('starting up on %s port %s' % server_address)
sock.bind(server_address)

# Listen for incoming connections
sock.listen(1)

while True:
    # Wait for a connection
    print('waiting for a connection')
    connection, client_address = sock.accept()

    try:
        print('connection from', client_address)

        # Receive the data in small chunks and retransmit it
        while True:
            data = connection.recv(1600)
            print('received "%s"' % data)
            if data:
                print('sending data back to the client')
                msg = str(data)
                msg = msg + ' plus extra from the server'
                msg = bytes(msg, 'utf-8')
                connection.sendall(msg)
            else:
                print('no more data from', client_address)
                connection.close()
                break

    finally:
        # Clean up the connection
        connection.close()
```

