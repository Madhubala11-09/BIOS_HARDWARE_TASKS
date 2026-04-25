
# wireshark setup 

To capture the Modbus TCP packets, I opened Wireshark and selected the loopback interface (lo), since the communication between the client and server occurs locally on 127.0.0.1 (local host). Initially, capturing on the “any” interface resulted in capturing all the traffic that my computer was handling in that moment, so i tried loopback interface to isolate only the relevant packets. After starting the capture, I configured Wireshark to recognize traffic on port 1212 as Modbus TCP by adding this port in the Modbus/TCP dissector settings (Analyze then Decode As / Protocol Preferences). Then, I used filters such as modbus.func_code == 3, modbus.func_code == 6, and modbus.func_code == 16 to view specific operations like reading and writing registers. Once the client performed actions (read/write), the corresponding Modbus request and response packets appeared in Wireshark. Finally, I used the following filters for filtering:
1. modbus.func_code == 3 to track holding register reads.
2.  modbus.func_code == 6 to track single register writes
3. modbus.func_code == 4 to track input register reads.
4. modbus.func_code == 16 to track multiple register writes

# writing single register alone

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/75b33ac4-e518-4883-b511-c51bccebe0f9" />

# writing multiple register

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5199ea19-a34c-487c-9119-8834bf2a6979" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d165e0f2-54ce-4da6-9146-35e5547a575c" />

# reading single, and multiple registers
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ca9ed4e1-4f16-4480-8106-685a13913304" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d02ad5c6-0e19-4899-a663-3a883d01bd5a" />


# server for modbus code
```
#!/bin/python
from pyModbusTCP.server import ModbusServer, DataBank
from time import sleep

server = ModbusServer("127.0.0.1", 1212, no_block=True)

try:
    print("Starting server...")
    server.start()
    print("Server is online")
    last_state=server.data_bank.get_holding_registers(0,100)
    while True:
        current_state = server.data_bank.get_holding_registers(0, 100)
        if current_state != last_state:
            for i in range(100):
                if current_state[i] != last_state[i]:
                    print(f"Change in register {i}: {last_state[i]}-->{current_state[i]}")
            last_state = current_state

        sleep(0.5) 

except Exception as e:
    print(f"Error occurred: {e}")
    print("Shutdown server ...")
    server.stop()
    print("Server is offline")

```
## code explanation:
we use databank to read and write from register, here in the server code server.data_bank.set_holding_registers() is to write the value of holding register, whereas server.data_bank.get_holding_registers() is to read the value of holding register. Here in the code, in the while loop, I read all the 100 register, and update last_value before the user enters data, and after the user enters data I read all 100 registers, to update it as current_state, so we can find if there is a change in registers value by comparing last and current state and print the value of register. 



# client for modbus code
```
from pyModbusTCP.client import ModbusClient

client = ModbusClient(host="127.0.0.1", port=1212, unit_id=1, auto_open=True)

while True:
    print("\n--- Modbus Menu ---")
    print("1. Read Single Register")
    print("2. Read Multiple Registers")
    print("3. Write Single Register")
    print("4. Write Multiple Registers")
    print("5. Exit")
    choice = input("Select an option (1-5): ")

    if choice == '5':
        break

    try:
        # READ SINGLE
        if choice == '1':
            reg_num = int(input("Enter Register Number (0-99): "))
            result = client.read_holding_registers(reg_num, 1)
            if result:
                print(f" Register {reg_num} Value: {result[0]}")
            else:
                print(" Read Error.")

        # READ MULTIPLE
        elif choice == '2':
            start_reg = int(input("Enter Starting Register (0-99): "))
            count = int(input("How many registers to read?: "))
            result = client.read_holding_registers(start_reg, count)
            if result:
                print(f"Values: {result}")
            else:
                print(" Read Error.")

        # WRITE SINGLE
        elif choice == '3':
            reg_num = int(input("Enter Register Number (0-99): "))
            reg_val = int(input("Enter Value (0-65535): "))
            if client.write_single_register(reg_num, reg_val):
                print(" Write OK!")
            else:
                print(" Write Error.")

        # WRITE MULTIPLE
        elif choice == '4':
            start_reg = int(input("Enter Starting Register (0-99): "))
            val_string = input("Enter values separated by commas: ")
            val_list = [int(v) for v in val_string.split(',')]
            if client.write_multiple_registers(start_reg, val_list):
                print(f" Write OK!")
            else:
                print(" Write Error.")
        
        else:
            print("Invalid selection.")

    except ValueError:
        print("Error: Please enter valid numbers.")

client.close()
```
## Code explanation: 
Here in this code we use a menu driven way, to let user decide on to read or write, and proceed accordingly. In each option we used the following to execute the functions:

1. client.read_holding_registers() to read register, this requests the server to read, here the first value in the bracket will be the register number and second value will be how many register to read so this can be used to read single and multiple registers.


2. client.write_single_register() to write single register, here first value is the register number and second value is the value to write in the register.


3. client.write_multiple_registers() this is to write multiple, here first value is the starting value of register from which we have to write, and the second value is a list, this list contains the value to be written in the registers, so from the starting value registers are written for how many numbers are there in the list.


# Difficulties I faced:

1. no_block=True, here intially I gave no_block=False, so my code didnt work, after learning more, I realised that this line is to tell the computer to wait and check for connnections in the background, while continuing with the tasks, after server.start() command, if we give false, the computer keeps listening to clients, and never proceed to other lines.


2. using get_words instead of  get_holding_register, this caused and error I used it this way ( DataBank.get_words) this was the basic syntax is got for Data bank in google, but while running the code it kept showing error, the error itself asked to use set_holding_registers() so I used (data_bank.get_holding_registers).



3. I had trouble with the server, execution of while loop, it kept showing varies errors, and kept running infinetly without waiting for the client to respond, so after refering few other examples of modbus tcp server I figured they used sleep so I added that and the problem was rectified.


4. Then in wireshark as I mentioned above I had trouble with capturing packets, that is with lo and any, since I had hard time with the permissions required to use lo to capture the packets.


5. Then I had trouble with choosing the port, because when I used port 502, the desginated port for modbus(tcp) I got port busy error, even after killing all the running operations, so I used another random port 1212, but this port didn't get recognised as modbus protocol, so I had to add this port as default modbus port to use modbus protocol function code stuff, I had hard time figuring this out.

# what is happening:

The communication follows a simple request–response flow. When the client performs an operation (such as reading or writing a register), it sends a Modbus TCP request packet containing the function code, register address, and data. This packet is captured in Wireshark on the loopback interface. The server receives the request, processes it using the DataBank (either reading or updating register values), and then sends a response packet back to the client confirming the operation or returning the requested data. Wireshark captures both the request and response packets.
