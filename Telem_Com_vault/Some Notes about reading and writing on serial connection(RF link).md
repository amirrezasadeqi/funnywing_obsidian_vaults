### Some Notes about reading and writing on serial connection(RF link)
I have tested a simple case in which we run on each of the GCS PC and RPI a node to interface RF module and in each of these nodes we have two threads, one for writing to port and the other for reading from that. The transferring messages were simple greetings(I mean short hello world messages). In this setup I had not any issues about reading and writing from/to serial port at the same time(But there may be in future and I don't know at the moment! we should go forward to see what happens).
The code for RPI is as bellow:
```python
#!/usr/bin/env python3

import rospy
import serial
import threading
import time


port = serial.Serial("/dev/ttyUSB0", baudrate=9600, timeout=1.0)


def writer_worker():

    if not port.isOpen():
        port.open()
	# port.reset_output_buffer()
	time.sleep(1.0)
	port.flush()

    while not rospy.is_shutdown():
        rospy.loginfo("RPI writer thread writes!")
        port.write(b"Hello from RPI writer thread!\r\n")
		# rospy.loginfo("writer log!")
        time.sleep(0.5)

    return


def reader_worker():

    if not port.isOpen():
        port.open()

	# port.reset_input_buffer()
	time.sleep(1.0)
	port.flushInput()
	
    while not rospy.is_shutdown():
        data = port.readline().decode('utf-8')
        rospy.loginfo(f"RPI reader thread reads: {data}")
		# time.sleep(0.25)
		# rospy.loginfo("reader log!")
		# time.sleep(1.0)

    return


if __name__ == "__main__":
    rospy.init_node("rpi_rf_node")
    writer_thread = threading.Thread(target=writer_worker)
    reader_thread = threading.Thread(target=reader_worker)

    writer_thread.start()
    reader_thread.start()

```
The code for GCS PC is as below:
```python
#!/usr/bin/env python

import rospy
import serial
import threading
import time


port = serial.Serial("/dev/ttyUSB0", baudrate=9600, timeout=1.0)


def writer_worker():

    if not port.isOpen():
        port.open()

    while not rospy.is_shutdown():
        rospy.loginfo("GCS writer thread writes!")
        port.write(b"Hello from GCS writer thread!\r\n")
        time.sleep(0.5)

    return


def reader_worker():

    if not port.isOpen():
        port.open()

    while not rospy.is_shutdown():
        data = port.readline().decode('utf-8')
        rospy.loginfo(f"GCS reader thread reads: {data}")

    return


if __name__ == "__main__":
    rospy.init_node("gcs_rf_node")
    writer_thread = threading.Thread(target=writer_worker)
    reader_thread = threading.Thread(target=reader_worker)

    writer_thread.start()
    reader_thread.start()

```

__Note__ that if we don't add the `time.sleep(0.5)` after writing to buffers in `writer_worker`, we would have some wierd behaviors like:
- Even if we don't send data from GCS, we get some not complete and corrupted greetings by running the node in RPI by `reader_thread`.
- Also the data transferring is corrupted data, for example I think some messages were incomplete or some characters were missed.
so this 0.5 seconde sleep is needed after writing to port(If you remember, I think also in 9XTend modules if we set the duration of the loop for sending packets below 0.5 seconds, the corresponding recieved data in the other side was corrupted and wierd!). Note that I don't tested yet the effect of those `flush` functions and those `reset_<in/out>put_buffer` functions. So this is a TODO:
- [ ] Check the effect the `flush` and `reset_<in/out>put_buffer` functions!

So by this simple example I think we are ready for implementing the RF link interface node for the funnywing project. __Note__ that these experiments are conducted using the old air module RF telemeteries. So you should also test them with funnywing 9XTend modules.


###  Which type are my serial connections? full or half duplex?
Based on the previouse simple example in which we have two threads writing and reading to/from the serial port we can conclude that our connections are full-duplex, but remember threading does not mean running processes in parallel and actually because of fast processor and fast switiching between processes, it feels that we are running processes in parallel. So there is a probability that the connection is half-duplex and since the switching between threads are so fast and furthermore the TX and RX lines are seperated(which causes the input and output bytes don't mix to each other and create wierd messages.), it seems to us like the full-duplex connection. Anyway, at the moment I have not more time to spend on examining this fact exactly, But in future I will ... and ask from and so on and ...
So in the future ...
So now let's go and make our funnywing project!

#### Some notes and observations about serial connection buffer
I have a weird problem with reading and writing from/to RF link serial ports, specially on RPI side. Consider the flow of the data stream in our scenario to further explain the problem. When the GCS RF modules sends data, the RPI RF module receives that data and because of the firmware running on the module, it resends the data to the USB-TTL converter module. Sequentially, the converter module sends data to the USB port of the raspberry pi(again because of the module firmware). Finally, When the RPI receives the data from its USB port, the linux kernel/drivers put the data to the input buffer of that specific serial port(/dev/ttyUSB0). I think(most likely based on my actual/physical experiments) this flow is running/repeating, even when RPIs does not open the serial port or reads/writes the data using python serial library functions.
As a result, when we run the RPI code after the GCS, the data can be accumulated on the RPI input buffer for the serial port. So after running on RPI we can get the old data.
To solve this issue we could use the reset_<input/output>_buffer function of the python serial library, but those functions don't work in a consistent way. I randomly observe situations in which, when I run the RPI node, it logs the data from the input buffer with a fast pace and weirdly the buffer contents don't empty and remain constant(I have tested it using in_Waiting property of python serial library.)
This can happen because of hardware problems or bad air module and now I have no more time to figure it out. So For now let's do the job as fast as possible but in future ...
Some useful links about this problem can be as below:
	- [data flow in setup](https://elinux.org/Serial_port_programming)
	- [when to flush pyserial buffers](https://stackoverflow.com/questions/52706998/when-to-flush-pyserial-buffers)
	- [pyserial when should I use flush](https://stackoverflow.com/questions/52706998/when-to-flush-pyserial-buffers)
	- [pyserial buffer won't flush](https://stackoverflow.com/questions/7266558/pyserial-buffer-wont-flush)
	- [reset_input_buffer does not appear to successfully clear the buffer](https://github.com/pyserial/pyserial/issues/344)

Now Let's do the job as fast as possible!

