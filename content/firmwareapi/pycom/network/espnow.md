---
title: "ESPNOW"
aliases:
    - firmwareapi/pycom/network/espnow.html
    - firmwareapi/pycom/network/espnow.md
    - chapter/firmwareapi/pycom/network/espnow
---
This module implements interface to ESP-NOW protocol.

## Usage Example

```python

from network import WLAN
from network import ESPNOW
import binascii
import time

# The callback to be registered when a message has been sent to a Peer
def espnow_tx(result):
    # "result" is the parameter in form of 2 element long tuple
    # "peer" is the Peer which the message has been sent to
    # "sent" is a boolean showing whether the message could be sent
	peer, sent = result
	mac = peer.addr()
	if(sent == False):
		print("Sending message to %s failed!" % binascii.hexlify(mac))
	else:
		print("Message sent to: %s" % (binascii.hexlify(mac)))

# The callback to be registered when a message has been received
def espnow_rx(result):
	# "result" is the parameter in form of 3 element long tuple
    # "mac" is the MAC address of the sender
    # "peer" is the Peer which the message has been received from. If message has been received from a not registered Peer this parameter is None
    # "msg" is the payload from the received message
    mac, peer, msg = result
	if(peer is None):
		print("Message received from an unknown Peer, registering...")
		peer = ESPNOW.add_peer(mac)
	print("Message received from %s with content: %s" % (binascii.hexlify(mac), msg))
	peer.send("Sending back an answer")

# The ESPNOW module needs that WLAN is initialized
w = WLAN()
# Initialize the ESPNOW module
ESPNOW.init()
print("ESP-NOW version: %s" % ESPNOW.version())

# Register the callback which will be called on TX
ESPNOW.on_send(espnow_tx)
# Register the callback which will be called on RX
ESPNOW.on_recv(espnow_rx)
# Configure the Primary Master Key which is used to encrypt the Local Master Key
ESPNOW.pmk('0123456789abcdef')

# Add a dedicated Peer with MAC address: 11:22:33:44:55:66
p1 = ESPNOW.add_peer("112233445566")
# Add a dedicated Peer with MAC address: 66:55:44:33:22:11
p2 = ESPNOW.add_peer("665544332211")
# Set the Local Master Key of Peer p1
p1.lmk(b'0123456789123456')
# Do not set LMK for p2, traffic is not encrypted

# Get the number of registered Peers and how many is encrypted
count = ESPNOW.peer_count()
print("Number of Peers: %s, encrypted: %s" % (count[0], count[1]))

# Sending some messages to the Peers individually
i = 0
while i < 10:
    # The Peer p1 will only receive this message if the same LMK is configured for the Peer object created for this device on the other device also
    p1.send("%s" % (i))
    # Message to p2 is not encrypted, on the p2 device encryption for this current device's Peer object should not be configured
    p2.send("%s" % (i))
    i = i + 1
    time.sleep(1)

# Sending 1 message to all Peers which are registered
ESPNOW.send(None, "Hello all Peers!")

# Remove 1 Peer
ESPNOW.del_peer(p1)

# Deinit the module
ESPNOW.deinit()

```

## Initialization

#### ESPNOW.init()

Initialize the ESP-NOW module.

## Methods:

#### ESPNOW.deinit()

Deinitialize the ESP-NOW module.

#### ESPNOW.pmk(pmk)

Configures the Primary Master Key which is used to encrypt the Local Master Key.

* `pmk` is the Primary Master Key to be configured.

The PMK specified by `pmk` is accepted in format of Byte array with length 16.

#### ESPNOW.send(addr, msg)

Sends a message `msg` to a remote device with MAC address `addr`.

* `addr` is the MAC address of the remote device. If `None` is passed then the message is sent to all registered Peers.
* `msg` is the message to send.

#### ESPNOW.on_recv(cb)

Registers the `cb` callback which will be called when a new message has been received.

* `cb` is the callback function to be called, it expects 1 parameter which is a tuple with 3 elements:
    * Element 0: is the MAC address of the sender.
    * Element 1: is the Peer which the message has been received from. If message has been received from a not registered Peer this parameter is None.
    * Element 2: is the payload from the received message.

#### ESPNOW.on_send(cb)

Registers the `cb` callback which will be called when a new message has been sent.

* `cb` is the callback function to be called, it expects 1 parameter which is a tuple with 2 elements:
    * Element 0: is the Peer which the message has been sent to.
    * Element 1: is a boolean showing whether the message could be sent.

#### ESPNOW.peer_count()

Returns with the number of registered Peers in a form of tuple with 2 elements:
* Element 0: is the number of all the registered Peers.
* Element 1: is the number of encrypted Peers.

#### ESPNOW.version()

Returns with the curently used version of the ESP-NOW protocol.

#### ESPNOW.add_peer(addr, lmk=None)

Creates a new Peer object and registers it into ESP-NOW module.
* `addr` is the MAC address of the Peer to be created. MAC address is accepted in either String format or as a Byte array.
* `lmk` is the Local Master Key to be used when communicating with the Peer. By default it is None which means the communication will not be encrypted.

The LMK specified by `lmk` is accepted in format of Byte array with length 16.

This function returns with an `ESPNOW_Peer` object.

#### ESPNOW.del_peer(Peer)

Destroys the Peer object with type `ESPNOW_Peer`.

## Class ESPNOW_Peer

The ESPNOW_Peer class represents a Peer in the scope of the ESP-NOW module. A new resource can only be created with the `ESPNOW.add_peer` function.

#### Class methods

#### ESPNOW_Peer.addr(addr=None)

Returns with the MAC address of the Peer.

#### ESPNOW_Peer.lmk(lmk=None)

Configures or returns the Local Master Key of the Peer.
* `lmk` is the new LMK to be set to this Peer.

If `lmk` is not used then this functions returns the LMK of the Peer.
The LMK specified by `lmk` is accepted in format of Byte array with length 16.

#### ESPNOW_Peer.send(msg)

Sends a message to the Peer.
* `msg` is the message to send.