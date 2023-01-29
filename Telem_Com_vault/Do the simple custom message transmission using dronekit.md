[related stack question](https://stackoverflow.com/questions/30306799/droneapi-and-custom-mavlink-message) : a little old. I think you should test the `Vehicle.send_mavlink` for sending custom messages!
[`send_mavlink API reference`](https://dronekit-python.readthedocs.io/en/latest/automodule.html) : search send_mavlink in this page.
[how to send a mavlink messages using `send_mavlink`](https://dronekit-python.readthedocs.io/en/latest/guide/copter/guided_mode.html#guided-mode-how-to-send-commands)
[`message_factory` explanation](https://dronekit-python.readthedocs.io/en/latest/automodule.html#dronekit.Vehicle.message_factory)
[Useful documents for listening on messages in dronekit](https://dronekit-python.readthedocs.io/en/latest/guide/mavlink_messages.html) I think this can help us in receiving data!

__Note:__ As I understood dronekit is more than pymavlink. its connection handler also automatically handles the listening to the Heartbeat message and many other messages like location and ... . it uses the mavutil.mavlink_connection inside but As I understood from its source code, it listens to the heartbeats as a ground control station!(I think these!)
So in RPI side you need to send heartbeat periodically with supported [MAV_TYPE](https://mavlink.io/en/messages/common.html#MAV_TYPE)! and use the pymavlink in there not the dronekit. this is a library for writing applications that communicate with autopilots( like mobile app connection to pixhawk and standard dialects like ardupilotmega.xml) and I think if you want to write mavlink code on drone or RPI you should/must use the pymavlink itself.
==For Now I think the pymavlink is enough and even better for funnywing custom connection between the RPI and my GCS application, but in future I will do this TODO:==
- [ ] Search on How we should send custom message between RPI and custom GCS application using dronekit? or How we can use custom dialects with dronekit(dronekit python)?
The Code we've developed till now are:
- these codes are in the GCS side:
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
© Copyright 2015-2016, 3D Robotics.

my_vehicle.py:

Custom Vehicle subclass to add IMU data.
"""

from dronekit import Vehicle


class HelloWorld(object):
    def __init__(self, sender_id=None, sender_name=None, sequence=None):
        """
        HelloWorld object constructor.
        """
        self.sender_id = sender_id
        self.sender_name = sender_name
        self.sequence = sequence

    def __str__(self):
        """
        String representation used to print the HelloWorld object. 
        """
        return "HelloWorld: sender_id={},sender_name={},sequence={}}".format(self.sender_id, self.sender_name, self.sequence)


class MyVehicle(Vehicle):
    def __init__(self, *args):
        super(MyVehicle, self).__init__(*args)

        # Create an Vehicle.raw_imu object with initial values set to None.
        self._hello_world = HelloWorld()

        # Create a message listener using the decorator.
        @self.on_message('HELLO_WORLD')
        def listener(self, name, message):
            """
            The listener is called for messages that contain the string specified in the decorator,
            passing the vehicle, message name, and the message.

            The listener writes the message to the (newly attached) ``vehicle.hello_world`` object 
            and notifies observers.
            """
            self._hello_world.sender_id = message.sender_id
            self._hello_world.sender_name = message.sender_name
            self._hello_world.sequence = message.sequence

            # Notify all observers of new message (with new value)
            #   Note that argument `cache=False` by default so listeners
            #   are updated with every new message
            self.notify_attribute_listeners('hello_world', self._hello_world)

    @property
    def hello_world(self):
        return self._hello_world

```

```python
#!/usr/bin/env python3


from dronekit import connect
from my_vehicle import MyVehicle  # Our custom vehicle class
import rospy
import time
import threading
# import dronekit
import os
from pymavlink import mavutil


def hello_world_callback(self, attr_name, value):
    # attr_name == 'raw_imu'
    # value == vehicle.raw_imu
    print(value)


# os.environ["MAVLINK20"] = "1"
# os.environ["MAVLINK_DIALECT"] = "funnywing"


rf_connection = connect(
    "/dev/ttyUSB0", wait_ready=False, baud=57600, vehicle_class=MyVehicle)

rf_connection.add_attribute_listener('hello_world', hello_world_callback)


def writer_worker():
    wc = 0

    while not rospy.is_shutdown():
        rospy.loginfo("GCS writer thread writes!")
        msg = rf_connection.message_factory.hello_world_encode(
            mavutil.mavlink.SENDER_ID_GCS, b"GCS", wc)
        rf_connection.send_mavlink(msg)
        # rf_connection.mav.hello_world_send(
        #     mavutil.mavlink.SENDER_ID_GCS, b"GCS", wc)
        wc += 1
        time.sleep(0.5)

    return


def reader_worker():
    while not rospy.is_shutdown():
        msg = rf_connection.hello_world
        print(
            f"GCS reader thread reads: (SENDER_ID: {msg.sender_id}, SENDER_NAME: {msg.sender_name}, SEQUENCE: {msg.sequence})")
        # msg = rf_connection.last_heartbeat
        # print(f"{msg}")
    return


if __name__ == "__main__":
    rospy.init_node("gcs_rf_node")

    writer_thread = threading.Thread(target=writer_worker)
    reader_thread = threading.Thread(target=reader_worker)

    writer_thread.start()
    reader_thread.start()

    rf_connection.close()

```

- This code is in the RPI side:
```python
#!/usr/bin/env python3


import rospy
import time
import threading
import os
from pymavlink import mavutil


# setting mavlink version to mavlink20
os.environ["MAVLINK20"] = "1"
rf_connection = mavutil.mavlink_connection("/dev/ttyUSB0", baud=57600, dialect="funnywing")

def writer_worker():
    wc = 0

    while not rospy.is_shutdown():
        rospy.loginfo("RPI writer thread writes!")
        rf_connection.mav.heartbeat_send(mavutil.mavlink.MAV_TYPE_FIXED_WING, mavutil.mavlink.MAV_AUTOPILOT_INVALID, 0, 0, 0)
        rf_connection.mav.hello_world_send(mavutil.mavlink.SENDER_ID_RPI, b"RPI", wc)
        wc += 1
        time.sleep(0.5)
    return

def reader_worker():
    while not rospy.is_shutdown():
        # receiving any kind of data
        msg = rf_connection.recv_match(blocking=True)
        if not msg:
            return
        if msg.get_type() == "BAD_DATA":
            if mavutil.all_printable(msg.data):
                print(f"Bad Data from GCS: {msg.data}")
        if msg.get_type() == "HELLO_WORLD":
            # Valid message
            print(f"RPI reader thread reads: (SENDER_ID: {msg.sender_id}, SENDER_NAME: {msg.sender_name}, SEQUENCE: {msg.sequence})")
        else:
            print("heart beat!")
    return


if __name__ == "__main__":
    rospy.init_node("rpi_rf_node", anonymous=True)

    writer_thread = threading.Thread(target=writer_worker)
    reader_thread = threading.Thread(target=reader_worker)

    writer_thread.start()
    reader_thread.start()

    # rf_connection.mav.heartbeat_send(mavutil.mavlink.MAV_TYPE_ONBOARD_CONTROLLER, mavutil.mavlink.MAV_AUTOPILOT_INVALID, 0, 0, 0)

```

#### some explanation about how dronekit streams data and also for streams for custom data(I think not actually custom, just the ones from the ardupilot messages sets that are not officially or currently handled automatically)
as I understood it inherits from the `vehicle` class and in there creates listener and property(with the on_message and property decorators) for the supported message in the message set(i think!). Also for that message defines a class to hold the data. finally in the main code it adds an attribute listener for the object of custom vehicle class which calls a callback function on receiving that message type. finally I think we can get the data with `vehicle.property` like we get the position and ... .

some useful links for this subject are:
- [connect and send_mavlink functions](https://dronekit-python.readthedocs.io/en/latest/automodule.html) 
- [message factory](https://dronekit-python.readthedocs.io/en/latest/automodule.html#dronekit.Vehicle.message_factory)
- [sending messages with message factory](https://dronekit-python.readthedocs.io/en/latest/guide/copter/guided_mode.html#guided-mode-how-to-send-commands)
- [dronekit mavlink message listener](https://dronekit-python.readthedocs.io/en/latest/guide/mavlink_messages.html)
- [example for adding the message listeners](https://dronekit-python.readthedocs.io/en/latest/examples/create_attribute.html#example-create-attribute)
- [source code of the dronekit where you can find many automatic mavlink message handlers like hearbeat](https://github.com/dronekit/dronekit-python/blob/34d54eb6a9d465f071d7f07ede9fb539815e35b3/dronekit/__init__.py#L1606)

In future I will work on using the dronekit for transmission of custom mavlink messages and dialects and for Now let's do the job! and I will ...

==Let's do the job ....!==


==__A little note:__== One thing that I've mentioned earlier is the fact that dronekit handles somethings automatically.
For example consider the case of `heartbeat` message. As I understood this message is a message which is used by standard/convention. I say that because using `pymavlink` I've sent some messages without sending/receiving the hearbeat message, So I can conclude that it is not a technical/software/hardware/... related necessity for communicating, But It can be a measure/notification to monitor the or to see if the connection is alive(I think and I mean by alive that the data stream continously transmit between systems). Actually I think the previous guys came into the idea that they need something showing the systems which their connections are alive and data transmits. Somewhere in the documentation I found a sentence saying that we should/must transmit heartbeat messages in the main loop or thread of the system to notify the other systems that the system works well and transmits data or its connection is alive, because if there was a problem in the system this thread could be stucked or ... (I think Not necesserily shows the failures because  maybe the minor threads have some problems and the main works well, But because the main tasks of the system are done by the main thread, so problem in the main thread, probabily implies a major problem!).
Based on what I've said the autopilots transmit heartbeat with periode of I think 1 Hz and dronekit connection using `connect` function handles this automatically(I think as a GCS app). So If we use the pymavlink I think we should do this by ourselves. ==Anyway this could be a nice thing to have in your communication system and you can implement it when your setup becomes more compelete. For now let's do the job!==
