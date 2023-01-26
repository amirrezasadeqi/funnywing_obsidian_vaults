Something really important suddenly came to my mind:
Consider the RPI case. It sends the sensors data and command responses back. we have two option:
- add all of the sending commands into a queue and sequentially send them back. This case may cause the data become out of date, especially in the case that we have too much sensor data!
- sending data concurrently, like using threads! this can cause the transmitting data become a mixture of bytes which are outputs of different threads(because of the nature of concurrency). This can be solved using `locks` that for example lock the port for writing a compelete message! maybe the libraries like dronekit and pymavlink use this method for sending data!
__NOTE__: in each of these cases the transmission is not just writing the messages on serial and you should handle the problems which I have explained above!


