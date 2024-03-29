---
marp: false
theme: default
title: A modern data acquisition proposal
description: A presentation on a pipeline to gather, store and vizualize arbitrary data using foxglove and mcap files
---

# <!--fit--> A new approach to data acquisition

By: Ben Hall

---
## overview
- what is the purpose of data acq?
    - live data viewing
    - "run to run" data recording
    - long-term data storage and ease of access to previous recordings
    - car performance over time

---
### from this:
![Alt text](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image-4.png)

---
### to this:
![Alt text](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image-5.png)

---
### foxglove studio used by Chalmers team in Formula Student
![Alt text](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image-6.png)

---
### foxglove studio

![gif](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/extension.gif)

---
![Alt text](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image-7.png)

---
### foxglove studio features
- foxglove studio can graph, show live video feed, and even show our GPS position on a google maps overlay.
- can run soley in a browser with no downloads required
- can re-play mcap data
- mcap files (run to run data reccording)
    - mcap file = smartly encoded binary file and is distinctly NOT a database

---
### how do we get there?
- misconception: foxglove studio != ROS required
- two main problems:
    - raw serialized CAN data ⮕ ? ⮕ foxglove live view 
    - raw serialized CAN bus data ⮕ ? ⮕ mcap file 
---
## on CAN and DBC files
- CAN is the largest source of data on the car
    - most of the boards connect over CAN
    
    - CAN = serialized protocol
    ![CAN data](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image.png)

---
## on CAN and DBC files
- The standard way of handling this process is through DBC files.
    - [introduction to dbc files](https://www.csselectronics.com/pages/can-dbc-file-database-intro)
    - DBC files can be used by numerous tools for analysis and automatic deserialization of CAN traffic via libraries like `cantools` or any of the other tools [lists here](https://github.com/iDoka/awesome-canbus?tab=readme-ov-file#converters-and-parsers)

    ![bg right 90%](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image-2.png)
---

## SYM; a better DBC file
- an alternative to DBC file that conveys the same information (and can easily be convert to DBC) is a SYM file
- generated by PCAN editor 6
![bg right 95%](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image-3.png)
- far more human readable than dbc files

---

### getting to our goal
- raw CAN bus bits ⮕ DBC parsing lib ⮕ raw parsed data ⮕ ? ⮕ foxglove live view 
- raw CAN bus bits ⮕ DBC parsing lib ⮕ raw parsed data ⮕ ? ⮕ mcap file 

---
### protobuf
- solves the same problem as DBC files do for the low level but is designed to be used with internet based protocols or streams
- based around `.proto` files and these files are used with a native tool called `protoc` for creating serialization and deserialization libraries for most any language
- nix integration and automation
    - Tim Gallion's [nix wrapper](https://github.com/notalltim/nix-proto) for `protoc` for automated creation of these libraries.


---
### how does protobuf connect?
raw parsed data packs a generated protobuf message
![bg right 90%](https://raw.githubusercontent.com/RCMast3r/data_acq_presentation/master/image-8.png)


---
### how does protobuf connect?

mcap files and the foxglove websocket protocol natively support protobuf messages

simple:
- mcap python library guide for [writing protobuf msgs to an mcap file](https://mcap.dev/guides/python/protobuf#writing)

mcap:
```python
mcap_writer.write_message(..., message=ExampleMsg_pb2.ExampleMsg(msg="Hello!", count=69))
```
foxglove studio webserver:
```python
# code that sends the ExampleMsg protobuf message over the foxglove websocket
await server.send_message(
    chan_id,
    time.time_ns(),
    ExampleMsg_pb2.ExampleMsg(msg="Hello!", count=420).SerializeToString(),
)
```

---
### reaching our goal

raw CAN bus ⮕ DBC parsing lib ⮕ raw parsed data ⮕ binary pb msg ⮕ foxglove 
raw CAN bus ⮕ DBC parsing lib ⮕ raw parsed data ⮕ pb msg class ⮕ mcap file 

 _(pb = protobuf)_

---

![bg center 80%](https://media.tenor.com/rcZMAz3r-fQAAAAM/borat-very.gif)

---
### what is the status of things?
- work started back in ~ mid December 2023 on this project
- main repositories:
    - https://github.com/hytech-racing/HT_CAN (CAN description)
    - https://github.com/RCMast3r/data_acq
    - https://github.com/RCMast3r/hytech_nixos (raspberry pi OS)
- feature milestones reached:
    - [x] low-level automatic CAN library generation
    - [x] nixos first boot on TCU and ssh connection established
    - [x] live view foxglove data on entire CAN bus of other car
- what is left to do?
    - [ ] relational database that stores meta data about per-run mcap files
    - [ ] local service for handling offload of mcap files from car
    - [ ] finalizing raspberry pi nixos image (~90% there)

