### Protocols Techniques for Reliable Data Transfer

This framework has been designed to enable practical experiments and obtaining hands-on experience with implementation of transport layer protocols for reliable data transfer (RDT). The naming scheme used for the packages and the classes follows the naming scheme of the RDT1-4 protocols in the networking book.

To undertake the following exercises it is assumed that you have read pages 234-246 in the networking book. You may also want to review the lecture notes from the transport layer lectures, in particular the parts that cover the RDT implementation framework.

##### Exercise 1 - Cloning the RDT Framework

Start by importing the RDT framework and associated testing project into your IDE (Eclipse):

https://github.com/selabhvl/dat110-rdt

https://github.com/selabhvl/dat110-rdt-testing

and make sure that the classes and the tests compile.

The project is organised into the following packages

- `no.hvl.dat110.application` implements an application-level sender process that sends messages to a receiver applicatiob-level process using the underlying transport protocol implementation.

- `no.hvl.dat110.common` contains some common classes for logging used in the implementation of the framework. It also contains the `Stopable.java` which implements a stopable thread-abstraction which is central to the implementation of the transport protocol entities.

- `no.hvl.dat110.network` implements a simulated network that can connect a sender and a receiver transport entity by means of two channels (one in each direction). By changing the type of the channels (the network model) we can simulate reliable and unreliable networks and experiments with different transport protocol implementations.

- `no.hvl.dat110.network.models` contains classes implementing various network/channel models, including a fully reliable channel, a channel in which DATA segments may be corrupted, a channel in which both DATA and ACK/NAK segments may get corrupted, a lossy channel and a channel in which overtaking of segments is possible.

- `no.hvl.dat110.transport` contains the base classes for implementing transport protocols for reliable data transfer of segments and defines the basic primitives of `rdt_send`, `deliver_data`, `udt_send`, and `rdt_recv`.

- `no.hvl.dat110.rdt1` contains classes implementing the RDT1 protocol.

- `no.hvl.dat110.rdt2` contains classes implementing the RDT2 protocol.

- `no.hvl.dat110.rdt21` contains classes implementing the RDT21 protocol.

- `no.hvl.dat110.rdt3` contains classes implementing the RDT3 protocol.

- `no.hvl.dat110.rdt4` contains classes that can serve as a basis for implementing the RDT4 protocol.

The `rdt-testing project` implements JUnit-tests for running and testing the transport protocols. The basic correctness criteria is that the receiver must receive all data sent from the sender and in correct order.

##### Exercise 2 - Basic experiments

Perform the following experiments

1. Run the main-method in `Main.java` where a sender process sends three messages to the receiver. Try to understand and interpret the output generated in the console and where it is generated from.

2. Run the `TestRDT1.java` which tests the rdt1.0 transport protocol over a perfect channel.

3. Run the `TestRDT2ReliableChannel.java` which tests the rdt2.0 transport protocol over a perfect channel.

4. Run the `TestRDT2DataBitErrors.java` which test the rdt2.0 transport protocol over a channel with bit-errors on DATA segments.

#### Exercise 2 - Handling Corrupt ACK/NAK segments

The `RDT2DataBitErrors.java` class implements a network model that at random sets negates the checksum in the transmitted segment thereby simulating a transmission errors. This in turn cause the `doProcess()` method in `TransportReceiverRDT2.java` to detect that the received segment has a checksum errors and therefore send a NAK segment.

Methods for calculating and checking checksums can be found in the `Segment.java` class.

As a checksum of `0` is the correct checksum for ACK and NAK segments (see Segment-constructors), then this effectively mean that only DATA segments can have transmission errors.

##### Exercise 2.1

Modify the implementation of the `SegmentRDT2.java` class such that the correct checksum or ACK/NAK is set to 1. This means that also ACK/NAK may have checksum errors.

##### Exercise 2.2

Augment the implementation of the sender side in the rdt2.0 transport protocol in `TransportSenderRDT2.java` such that the sender checks for any transmission errors on ACK and NAK segments.

What could/should the sender do in case a corrupt ACK / NAK segment is received?

Implement your proposed solution and use the `TestRDT2DataAckNakBitErrors.java` to test your solution. You can modify the probability of transmission errors by adjusting the value `CORRUPTPB` in `RDT21DataAckNakBitErrors.java` network model.

#### Exercise 3 - RDT 2.2 Implementation

The  `no.hvl.dat110.transport.rdt21` package  implements the RDT 2.1 transport protocol for reliable data transfer over a channel with bit errors as described on pages 237-239 in the networking book. The implementation of the sender and receiver are in the classes `TransportSenderRDT21.java` and `TransportReceiverRDT21.java`.

The transport receiver of RDT2.1 uses a NAK (negative acknowledgement) to signal to the sender that a corrupt data segment has been received. As described on page 242 in the networking book, then it is also possible to replace the use of NAKs with the use of an ACK (acknowledgement) in combination with a sequence number.

Modify the RDT 2.1 implementation (i.e., the classes `TransportSenderRDT21.java` and `TransportReceiverRDT21.java`) to become an RDT2.2 implementation as described on page 242 in the networking book.

The test `TestRDT21DataAckNakBitErrors.java` can be used to test your protocol implementation.

#### Exercise 4 - Reliable Transport and Overtaking

The  `no.hvl.dat110.transport.rdt3` package implements the RDT 3.0 transport protocol for reliable data transfer over a channel with bit errors and loss of segments as described on pages 242-245 in the networking book.

##### Exercise 4.1: Study the RDT 3.0 Implementation

The implementation of the sender in `TransportSenderRDT3.java` uses a timer to implement timeouts, and thereby recover from possible loss of data and acknowledgements. Study the implementation of the transport sender and transport receiver in order to understand how it implements retransmission and the state machines shown in Figures 3.15 and 3.16.

The test in `TestRDT3LossyBitErrors.java` can be used to run and test the implementation.  

##### Exercise 4.2: Implementation of the RDT 4.0 Protocol

The RDT3.0 protocol cannot recover from overtaking of segments, i.e., that segments may arrive in a different order from which they were sent.

The `no.hvl.dat110.transport.rdt4` package contains templates for implementing a RDT 4.0 protocol that can recover from overtaking of segments. The protocol is to replace the alternating bit sequence number of RDT3.0 with an increasing sequence number and operates as follows:

- The sender keeps sending the same data segment (having the same sequence number) until an acknowledgement with a sequence number which is one higher is received. This should include retransmission similar to RDT 3.0. This means that the sender interprets a sequence number received in an acknowledge which is one higher that its current sequence number as indication that the receiver has received the data segment with the current sequence number.

- The receiver keeps an internal sequence number indicating the data segment expected next. If a data segment with the expected sequence number is received, then the internal sequence number is incremented and a corresponding acknowledgement segment is sent back to the sender. If the receiver receives a data segment with the wrong (non-expected) sequence number, then the receiver replies with an acknowledgement segment containing the sequence number of the data segment expected next.

The test in `TestRDT4DelayLossyBitErrors.java` can be used to test and run the protocol implementation.

##### Exercise 5: Variant of the RDT4.0 Protocol

Modify the implementation of the receiver side of the RDT4.0 protocol in Exercise 4.2 such that an acknowledgement is only sent if the data segment with the expected sequence number is received. Is the protocol still correct?
