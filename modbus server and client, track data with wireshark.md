
# wireshark setup 
use lo(loopback) i intially used any since lo needed few permission settings, but realised any captures all packets that my computer is communicating with anyway i can filter it but it was lot of data, after changing permissions i used lo to capture the data, before that i added 1212 port also in modbus/tcp port default port so that it recognises the captured datd using modbus protocol, then i applied the filter modbus.func_code==6 for single register writing
# writing single register alone
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/75b33ac4-e518-4883-b511-c51bccebe0f9" />
# writing multiple register
# reading single register
# reading multiple register

# server for modbus code

# client for modbus code
