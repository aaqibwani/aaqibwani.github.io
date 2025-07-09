---
title: Capture a Network Trace in Windows without installing Wireshark
nav_order: 12
---
# Capture a Network Trace in Windows without installing Wireshark
You can capture a network trace in Windows without installing Wireshark by using the built-in `netsh` command. Here's how you can do it:

1. **Open an Elevated Command Prompt**:
   - Press `Win + X` and select "Command Prompt (Admin)" or "Windows PowerShell (Admin)".

2. **Start the Network Trace**:
   - Run the following command:
     ```sh
     netsh trace start persistent=yes capture=yes filemode=circular report=disabled tracefile=C:\temp\nettrace.etl
     ```
     Make sure the `C:\temp` directory exists or choose another location.

3. **Reproduce the Issue**:
   - Perform the actions that you want to capture in the network trace.

4. **Stop the Network Trace**:
   - Run the following command to stop the trace:
     ```sh
     netsh trace stop
     ```

5. **Convert the etl Trace to pcapng**:
   - The trace will be saved as an `.etl` file in the specified location. You can convert this file to a format readable by Wireshark using `Etl2Pcapng` https://github.com/microsoft/etl2pcapng 
   -  Open Command Prompt and navigate to the directory where etl2pcapng is installed
   -  Run 
```
etl2pcapng.exe nettrace.etl nettrace.pcapng
```

6. **View the Trace**
   -  Open WireShark
   -  Go to File > Open and select the newly converted .pcapng file
 
 
{: .important }
> This method works on Windows 7/2008 R2 and above.
