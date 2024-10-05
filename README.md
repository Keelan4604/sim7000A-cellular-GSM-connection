# sim7000A-cellular-GSM-connection
Explain first up until 8/11



Im now set up and receiving messages from both my phone and from hologram to the sim stored on the sim. Im in roaming mode and not connected to home network but i dont think it matters. The sim was automatically in PDU mode when i recieved the messages and because of baud rate problems i was unable to decode the messages so i swithced over to SMS format which i should have been on in the first place but then i think because im trying to read PDU in SMS format it wont show any other info. Im going to switch over to SMS, send a new message and try to read it in text format, hopefully itll work.

Did not work, will update soon.

Update from a few days later. (8/15) I got it to work it was that setting the CNMP to 38 was disabling GSM for CREG which is needed in my area for SMS. On top of that i was on AT&T which I believe doesn't support SMS in my area either. Setting CNMP to 2 for auto and staying on T-Mobile did the trick. However, I will eventually need to figure out how to get this to work when I am auto connecting to random networks when I am traveling. Another problem I had was confusing CNMP with CMNB.





SETUP AT COMMANDS:

AT+CMGF=1 ; SMS mode

AT+CPMS="SM","SM","SM" ; show message storage

AT+CMGD=1,4 ; delete all messages

AT+CMGR= ; read message in space

AT+COPS=0 ; automatic selection

AT+CSQ ; network strength



AT+CGDCONT=1,"IP","hologram"

AT+CNMP=0 vs 38????

AT+COPS=1,2,"310410"



AT&F

AT+COPS=0

AT+CNMP=0

AT+CGDCONT=1,"IP","hologram"

AT+CPMS="SM","SM","SM"



setup before working

AT+CREG?

AT+COPS=1,2,"310410"

AT+CSQ

AT+CNMP? - should be 38

AT+CNMP=2

AT+CMNB=1

AT+COPS=1,2,"310410"

AT+CFUN=1

AT+CMNB=1

AT+CNMP=38

AT+CNMP? - should be 38- dont think so

AT+CMNB? - should be 1

AT+CIPSTATUS

AT+CSTT="hologram"

AT+CIICR

AT+CIFSR

AT+CIPSTART="TCP","cloudsocket.hologram.io",9999

AT+CCID - 8935711550`00608094f

AT+COPS? - +COPS: 1,2,"310410",7

AT+CSQ - +CSQ: 21,99

AT+CGREG? 0,5

AT+CMGL

AT+COPS? - 1,2,"310410",7

AT+COPS=0

AT+CGREG? 0,5

AT+CREG? 0,2

AT+CPMS="SM","SM","SM" no messages yet

at+cops? - +COPS: 0,2,"310410",7

AT+CREG? 0,3

AT+COPS? - +COPS: 0,2,"310410",7

AT+CSCA? - +CSCA: "+3933588d0000087",14e

AT+COPS=0
AT&F

AT+COPS=0 no response

AT+CNMP=0 no response

AT+CREG? 0,3

AT+CgREG? 0,3

AT+CGDCONT=1,"IP","hologram" 

AT+CPMS="SM","SM","SM" - +CPMS: 3,20,3,20,3,20 



8/13/24



AT+CNMP=38 ; LTE only no GSM

AT+CNMP=13 ; GSM only no LTE

AT+COPS=1,2,"310410"



AT+CNMP=2 ; auto

AT+CMNB=1 ; RAT pref to CATM1 for using LTE

AT+CMGF=1 ; text mode

AT+CGDCONT? ; APN/PDP query

AT+COPS=1,2,"310260" ; T-Mobile CREG 0,5 CNMP:13 CMNB:1 ; +COPS: 1,2,"310260",7

AT+CPMS="SM","SM","SM" ; +CPMS: 4,20,4,20,4,20

AT+CIPSTATUS ; STATE: PDP DEACT

AT+CMGR= ; read SMS



8/15/24

It's mostly working now, definitely staying connected to the network which is good, just doing more basic but tedious coding which is responsible for automatically detecting, receiving, and processing these messages. Doing this from scratch yields a lot more problems than one would imagine like getting a correct message count, storing it to integers vs strings, pulling message content from different length phone numbers, automatically detecting messages through storing previous message count integers and processing, etc.



To Do:





be able to connect to networks while traveling



receiving messages from longer phone number



increasing character return max amount from SIM7000A



8/22/24

Last I remember I did get the texts working and the code working. Obviously when I went to put it into the actual truck it stopped working and I haven't had time to work on it. Once I had the cellular configured correctly and reliably connecting on boot it was just a bunch of coding to get the SoftwareSerial to communicate properly and deliver the information from the sim7000a to the Arduino. I had the most problems getting long strings to reliably deliver but once I got it good enough I could extract the text and send it through a series of functions that would operate different pins based on the text. Not sure why it stopped working when I plugged it in. Some things I had to figure out were auto checking for messages, extracting information from messages based on message count and number. To check for messages we run a function that checks the number of messages stored on the sim, converts this into an integer which is compared to the integer which was gathered last time we ran the function. Each time we run the function it ran it over and over until it got the same answer twice in a row to ensure that it is the true number because sometimes SoftwareSerial likes to make things up. I also used this for gathering the text string itself. Once we have the new number of stored messages, we read the new message, store it to a string, then process it, running specific commands based on what the message was.

