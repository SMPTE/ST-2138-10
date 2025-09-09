# ST 2138-10  Catena — Model

## General

_This repository is *public*._

Please consult [CONTRIBUTING.md](./CONTRIBUTING.md), [CONFIDENTIALITY.md](./CONFIDENTIALITY.md), [LICENSE.md](./LICENSE.md) and
[PATENTS.md](./PATENTS.md) for important notices.

Your feedback is welcome at _link to GitHub issue tracker_ or at _TC chair email address_.

## Public Committee Draft (PCD) Notice

The following elements are made available for a public review period ending no earlier than 2026-03-02, and no later than 2026-09-01:

* <a href="https://github.com/SMPTE/ST-2138-10/blob/main/34CS-PCD-ST-2138-10-Catena-Model-2025-09-08.pdf">SMPTE PCD ST 2138-10 Catena — Model</a>
* <a href="https://github.com/SMPTE/st2138-a.git">https://github.com/SMPTE/st2138-a.git</a>
* <a href="https://github.com/SMPTE/st2138-a/blob/main/interface/schemata/device.yaml">https://github.com/SMPTE/st2138-a/blob/main/interface/schemata/device.yaml</a>

## Details

### Overview
SMPTE ST 2138-10 _Catena — Model_ specifies a standardization of communication methods between (micro)services and full products designed for hybrid cloud and on-premises solutions, with the goal of making it easy to secure, connect and control a multi-vendor ecosystem of media processing services and microservices, no matter where they are in a true plug-and-play model.

### The Hybrid Environment Reality
Today's environments typically encompass a combination of on-premises and cloud environments. Solutions span a variety of platforms, and must interoperate, regardless of where they are hosted.

### The Need for More Seamless Multi-Vendor Integration
The number of devices, products, and services from a wide array of vendors can be daunting, particularly with the move to microservices. Media companies need technologies to simply work together.

### Security as a Priority
The need to secure environments and the connections between products and services in a standardized manner is a must. ST 2138-10 defines a self-describing device model that incorporates an access control mechanism designed to scale across multiple devices from multiple vendors.

## Additional Elements
Interface Definitions based on the model shown in this document are companion elements to this document. These can be found in the public repository covering all parts of the 2138 suite of documents at the following URL: <a href="https://github.com/SMPTE/st2138-a.git">https://github.com/SMPTE/st2138-a.git</a>

Navigate to the /interface/proto folder for the interface definitions in Protobuf 3.0. 

The schemata can be found at /interface/schemata/device.yaml, but are published at the following URL: <a href="https://github.com/SMPTE/st2138-a/blob/main/interface/schemata/device.yaml">https://github.com/SMPTE/st2138-a/blob/main/interface/schemata/device.yaml</a>, which is also the $id tag for the schemata. 

The contents of st2138-a.git are normative.
The software version corresponding to this document revision is tagged as v1.0.0-pcd. 
