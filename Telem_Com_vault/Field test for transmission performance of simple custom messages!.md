You should send a simple custom mavlink message to the RPI and take the response back. Also consider some form of error assersion in message transmission to be able to compare the implemented methods for messaging.

### Do you observe bad data transmission using your simple codes?
- __YES__: Use a more complex transmission mechanisms, like mavlink microservices. To do this you need to use MAVSDK or DroneKit which have implemented a subset of standard microservices.
- __NO__: Use simple serialization to text for messaging, like `json` module.

==TODO==
- [ ] Prepare your laptop for field test!
