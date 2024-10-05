# SIM7000A Cellular GSM Connection

## Overview and Initial Setup (Up to 8/11)

I'm now set up to receive SMS messages on my **SIM7000A** module, both from my phone and from the **Hologram** IoT platform. These messages are stored directly on the SIM card. Initially, I was in **roaming mode** and not connected to a specific home network, but this did not seem to affect the ability to receive messages.

One early issue was the **PDU (Protocol Data Unit) mode**. By default, incoming messages were received in **PDU format**, which is essentially a binary format for SMS messages. Due to a mismatch in the **baud rate** and lack of compatibility, decoding these PDU messages was challenging. I then switched over to **SMS text mode** by setting `AT+CMGF=1`, which allows messages to be processed as plain text.

However, since the SIM had already received messages in PDU format, trying to read them in SMS text mode produced no meaningful information. The next step was to send a new SMS while in **SMS mode** and read it in the correct format. This is the expected behavior, as mixing PDU and SMS modes causes issues in message interpretation.

## Troubleshooting and Progression (8/15)

The new approach of switching to SMS mode and reading a new message did not initially work. After extensive debugging and testing AT commands, I discovered two main issues:
1. **Network Mode Problem**: Setting `AT+CNMP=38` was disabling **GSM** connectivity and forcing the device to use **LTE-only mode**. This setting prevented the module from properly registering on the network via `CREG` in my area, which was required for SMS transmission.
2. **Carrier Compatibility**: My SIM was connected to **AT&T**, which didn't support SMS in my region. I switched `CNMP` to `2` for auto-mode and connected to **T-Mobile**, which resolved the SMS reception problem.

These changes allowed the module to correctly receive and send SMS messages. However, for future use while traveling, I will need to figure out a way to automatically connect to various networks as I move across different regions. Another hurdle was confusing **CNMP** (Cellular Network Mode Preference) with **CMNB** (Cellular Network Bandwidth), which caused me to test incorrect configurations.

## AT Command Setup and Testing

Here are the AT commands used to configure the **SIM7000A** module for receiving SMS messages and testing connectivity:

```plaintext
AT+CMGF=1                        ; Set SMS mode to text
AT+CPMS="SM","SM","SM"           ; Select and display message storage (SIM storage)
AT+CMGD=1,4                      ; Delete all stored messages on the SIM
AT+CMGR=                         ; Read a specific SMS message
AT+COPS=0                        ; Automatic network selection
AT+CSQ                           ; Check signal strength
AT+CGDCONT=1,"IP","hologram"     ; Set up PDP context for Hologram APN
AT+CNMP=2                        ; Set network mode to auto (supports GSM and LTE)
AT+CMNB=1                        ; Set RAT (Radio Access Technology) preference to CAT-M1 for LTE
```

## Diagnosing Network Issues

Initially, my SIM was connected to **AT&T** using the following command:
```plaintext
AT+COPS=1,2,"310410"             ; Force manual selection to AT&T's MCC/MNC (310410)
```
After some `AT+CREG?` and `AT+CGREG?` checks, it was clear that registration was failing. Signal strength (`AT+CSQ`) was also inconsistent. Switching to **T-Mobile** resolved these issues:
```plaintext
AT+COPS=1,2,"310260"             ; Force manual selection to T-Mobile's MCC/MNC (310260)
```
Setting `CNMP=2` allowed the module to use both GSM and LTE as available, and `CMNB=1` was set to prefer LTE for higher speed connections.

## Further Development and Coding

With the SIM7000A now receiving messages properly, I began developing the Arduino code to automate **message detection**, **reception**, and **processing**. This involved multiple steps:
- **Counting Stored Messages**: Using AT commands to retrieve the count of stored messages on the SIM and convert this count to an integer for easy comparison.
- **Extracting SMS Content**: Parsing the content of incoming messages, which varies in length depending on the phone number and message.
- **Ensuring Reliable Data**: Since **SoftwareSerial** (used for Arduino communication with the SIM7000A) can occasionally produce errors or return false readings, I implemented checks to confirm consistent results across multiple reads.

The goal was to enable the Arduino to interpret SMS commands and operate different pins accordingly (e.g., turning on lights or controlling motors based on the message content).

## Key Coding Challenges

The most tedious aspect of this project was accurately processing the SMS data:
- **Correct Message Count**: The function checking message count had to be robust enough to handle variations in network conditions and false positives from the `SoftwareSerial` buffer.
- **String vs. Integer Storage**: Properly storing the message content as a string and comparing it to integer message counts was necessary for consistent processing.
- **Detecting New Messages**: By running a loop to constantly compare the current message count with the previously stored count, the code could identify when a new message arrived.

## AT Commands for Diagnostics and Troubleshooting

I used various AT commands throughout this process to troubleshoot and ensure stable connections:
```plaintext
AT+CREG?                          ; Check network registration
AT+CSQ                            ; Query signal quality
AT+CGDCONT?                       ; Query current APN and PDP context settings
AT+COPS?                          ; Check current network operator
AT+CIPSTATUS                      ; Check the current network status (e.g., PDP activation state)
AT+CIPSTART="TCP","cloudsocket.hologram.io",9999 ; Open a TCP connection to Hologram
AT+CPMS="SM","SM","SM"            ; Query message storage
AT+CMGL                           ; List all SMS messages
AT+CMGR=                          ; Read a specific SMS message by index
AT+CCID                           ; Display SIM card identifier
```

## Final Status and Future Work (8/22)

After extensive debugging, the system was finally able to reliably receive, parse, and respond to SMS messages. However, when integrating the module into the actual truck setup, the functionality broke, requiring further investigation.

Some key problems that I faced include:
- **Reliable Cellular Connection on Boot**: Ensuring that the SIM7000A would connect to the network automatically and reliably upon power-up was a persistent issue.
- **Long String Handling**: SMS messages with long strings caused buffer overflow problems, which I had to mitigate by optimizing the code and parsing the data effectively.
- **Automated Message Processing**: Implementing a loop to check for new messages, process the data, and execute commands based on message content.

### Current To-Do List:
- **Travel Connectivity**: Improve the ability to automatically connect to different networks while traveling.
- **Handling Longer Phone Numbers**: Refine message parsing to handle various phone number lengths without errors.
- **Increase Character Limit**: Optimize the SIM7000A and Arduino to handle longer messages without losing data.

In summary, the main challenges have been resolved, but some finer points of integration and network reliability still need work, particularly in ensuring that the system operates smoothly in its intended final setup.
