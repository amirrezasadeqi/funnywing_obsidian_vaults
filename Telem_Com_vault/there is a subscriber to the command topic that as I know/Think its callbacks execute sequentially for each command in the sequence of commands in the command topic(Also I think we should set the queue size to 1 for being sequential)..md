### Some Notes about the queue size in ROS publisher and subscribers
As I understood each ROS node has some buffers that we actually use them when publishing and subscribing. For example consider the case of publication. When you publish a message, indeed you add that message into the buffer in your node and ROS(I think roscore or something embedded in the ROS framework) takes the messages from the buffer and sends them throught the ROS network. Also in the subscription case, ROS receives the messages and puts them into a buffer(for the node) and handles the callback invokation for the messages in that buffer(I think callbacks are being handled by the rospy.spin() which actually creates a thread inside as I think and know).
So if you publish with a rate larger than ROS sending the data in the ROS network, those pulished data will be accumulated in the buffer or the __queue__(FIFO). If the rate is too high such that the queue becomes full and ROS is busy in sending the data, publishing new data leads to the deletion of the old data in the queue. So the queue size must be set big enough to not drop important data.
In the case of subscription, if the callbacks take too much time(usually the callback are being called sequentially not concurrently and this will be considered in this note), the buffer will be filled by ROS and receiving new data will delete the oldest data in the queue.
Some important links about this subject are:
- [Reason to set queue size of ROS publisher or subscriver to a large value](https://stackoverflow.com/a/60554760/10243689)
- [Is it sufficient to set ROS publisher buffer to 1 and Subscriber buffer to 1000 and still not loose any messages](https://stackoverflow.com/a/68473712/10243689)
- [How do Publisher/Subscriber Message Queues Work?](https://answers.ros.org/question/243855/how-do-publishersubscriber-message-queues-work/)
- [roscppOverviewPublishers and Subscribers](http://wiki.ros.org/roscpp/Overview/Publishers%20and%20Subscribers)
- [Choosing a good queue_size](http://wiki.ros.org/rospy/Overview/Publishers%20and%20Subscribers#Choosing_a_good_queue_size)

### Are ROS callbacks(specially subscription callbacks) executing concurrently?
Actually in most common cases that a node subscribes to a topic towhich another nodes is publishing, the answer is __YES__. Indeed in the cases that the data published by multiple processes, As I understood for each process, a new thread will be created and the answer would be __NO__.
In the case of funnywing, I think the callbacks will be ran sequentially, because until now I think the commands are publishing from a single node or process.
Below is a good Q&A abouth this question:
- [Are Rospy subscriber callbacks executed sequentially for a single topic?](https://robotics.stackexchange.com/questions/20069/are-rospy-subscriber-callbacks-executed-sequentially-for-a-single-topic)
and [this answer](https://robotics.stackexchange.com/a/21891/27131) is the nicest one!
Also there are some cases that data publishing in batches or in burst and it is related to somethings about Naggle algorithm and `tcp_nodelay` option in rospy publisher class objects and I did not see this problem yet. You can find more information about this in below link:
- [rospy.Publisher initialization](http://wiki.ros.org/rospy/Overview/Publishers%20and%20Subscribers)

So for now consider the callbacks are being invoked sequentially and make the funnywing as fast as possible!
let's do the job!
