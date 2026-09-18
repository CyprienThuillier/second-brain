The TCP/IP model is very similar to the OSI model. It consists of 4 layers :

![[tcpip_layers.png]]

We can compare with the OSI model : 

![[tcpip_osi_layers_cmp.png]]

When you attempt to make a connection, your computer first sends a special request to the remote server indicating that it wants to initialise a connection. This request contains something called a *SYN* (Short For Synchronise) bit, which essentially makes first contact in starting the connection process. The server will then respond with a packet containing the SYN it, as well another "acknowledgement" bit, called *ACK*. Finally, your computer will send a packet that contains the ACK bit by itself, confirming that the connection has been setup successfully.

![[tcpip_working.png]]

