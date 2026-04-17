
# wireshark setup 

use lo(loopback) i intially used any since lo needed few permission settings, but realised any captures all packets that my computer is communicating with anyway i can filter it but it was lot of data, after changing permissions i used lo to capture the data, before that i added 1212 port also in modbus/tcp port default port so that it recognises the captured datd using modbus protocol, then i applied the filter modbus.func_code==6 for single register writing

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

# Ensure port matches client (1109)
server = ModbusServer("127.0.0.1", 1212, no_block=True)

try:
    print("Starting server...")
    server.start()
    print("Server is online")
    
    # Initialize the baseline for all 100 registers
    # Use get_words, NOT get_holding_registers
    last_state=server.data_bank.get_holding_registers(0,100)
    while True:
        # Get the current snapshot
        current_state = server.data_bank.get_holding_registers(0, 100)

        # Check if anything changed in the whole bank
        if current_state != last_state:
            for i in range(100):
                if current_state[i] != last_state[i]:
                    print(f"Change in register {i}: {last_state[i]}-->{current_state[i]}")


            # Update the baseline
            last_state = current_state

        sleep(0.5) # MUST be inside the while loop

except Exception as e:
    print(f"Error occurred: {e}")
    print("Shutdown server ...")
    server.stop()
    print("Server is offline")

```

# client for modbus code
```
from pyModbusTCP.client import ModbusClient

# Ensure port matches your server
client = ModbusClient(host="127.0.0.1", port=502, unit_id=1, auto_open=True)

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
                print(f">>> Register {reg_num} Value: {result[0]}")
            else:
                print(">>> Read Error.")

        # READ MULTIPLE
        elif choice == '2':
            start_reg = int(input("Enter Starting Register (0-99): "))
            count = int(input("How many registers to read?: "))
            result = client.read_holding_registers(start_reg, count)
            if result:
                print(f">>> Values: {result}")
            else:
                print(">>> Read Error.")

        # WRITE SINGLE
        elif choice == '3':
            reg_num = int(input("Enter Register Number (0-99): "))
            reg_val = int(input("Enter Value (0-65535): "))
            if client.write_single_register(reg_num, reg_val):
                print(">>> Write OK!")
            else:
                print(">>> Write Error.")

        # WRITE MULTIPLE
        elif choice == '4':
            start_reg = int(input("Enter Starting Register (0-99): "))
            val_string = input("Enter values separated by commas: ")
            val_list = [int(v.strip()) for v in val_string.split(',')]
            if client.write_multiple_registers(start_reg, val_list):
                print(f">>> Write OK!")
            else:
                print(">>> Write Error.")
        
        else:
            print("Invalid selection.")

    except ValueError:
        print("Error: Please enter valid numbers.")

client.close()
```
