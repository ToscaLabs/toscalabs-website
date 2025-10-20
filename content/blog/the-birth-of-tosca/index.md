+++
template = "post.html"

title = "The birth of `tosca`"
description = """
A customizable IoT framework for developing firmware and the
associated controllers.
"""
date = 2026-01-05
tags = ["tosca", "toscalabs", "secure-by-design", "rust"]
+++

The `tosca` framework has finally come to light! 🎉🎉

Originally designed as an IoT architecture to make device command risks
transparent to users, `tosca` has, over the past two years, evolved into a
**customizable**, **self-contained** framework.

By **customizable**, we mean that the framework can easily adapt to any existent
firmware without changing the internal implementation of device commands.  
The framework APIs only expose commands to external software, optionally
hiding some parameters or adding metadata.  
To achieve this, an **_HTTP_** route must be defined for each command.  
During firmware startup, `tosca` aggregates all route information into a
single file representing the device description, which is then made available on
the network through an **_HTTP_** server.  
To interact with `tosca`-compliant devices, a software application must
integrate the `tosca` controller component  .
This component first discovers devices on the network, then retrieves their
description files through a **_REST_** request and maps the data into its
internal structures.  
Eventually, developers can use the controller APIs to configure device command
parameters and then send these commands as **_REST_** requests.  
A controller also provides APIs to allow or block requests to devices based on
privacy rules defined from the risks associated with each command.

By **self-contained**, we mean that the framework operates as an independent
system, minimizing reliance on external resources. One of its key features is
the ability to establish communication between the firmware and its controller
using only built-in functionalities.

# Main Features

* **Designed for IoT applications**: from home automation to industrial
processes, including devices for social purposes, such as those assisting
people with disabilities and older adults.
* **Firmware APIs**: for building firmware components, including device
descriptors, discovery protocols, and server configurations.
* **Controller APIs**: for discovering devices on a network, executing their
commands, and managing their hazards information.
* **Command Hazards**: labels attached to device commands to alert users
of potential _safety_, _privacy_, and _financial_ hazards during execution.
* **Safe programming with Rust**: memory and thread safety guarantees,
preventing many classes of bugs at compile time.

# Still Missing Features

- **Secure communication**: ensure confidentiality, integrity, and
authentication between firmware devices and controllers.
- **Bluetooth stack integration**: enable wireless connectivity and
communication with devices.
- **Accurate energy and performance metrics**: provide primitives for precise
on-device monitoring and performance analysis.
- **Support for additional microcontroller architectures**: expand hardware
compatibility and improve portability.
- **Command scheduling system**: allow controllers to execute device commands
at specific times or according to predefined schedules.
- **Over-the-Air (OTA) updates**: enable reliable remote firmware upgrades.

## Technical aspects

- **Code Refactoring**: remove duplicated code, clarify the intent of some
APIs, and eliminate ambiguous logic between devices and controllers.
- **Reduced heap allocations on firmware devices**: improve memory efficiency
for resource-constrained environments
- **Performance Optimizations**: reduce the number of instructions in critical
methods, leverage more efficient hardware instructions, and minimize reliance
on external crates whenever possible.

# Framework Structure

The framework is centered around the `tosca` interface, which serves as a bridge
between the two different sides of the framework.
The first side is the **Firmware Side**, which provides APIs for developing
firmware and drivers for sensors.
The second side is the **Controller Side**, which provides APIs for interacting
with devices built using the `tosca` framework.

The [`tosca`](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca)
interface is the core component of the framework. It is used by both the
firmware and controller side APIs to provide a uniform set of definitions.

It can:

* **Create and manage REST routes** that issue commands from a controller to a
device. Each route can specify parameters to configure the corresponding
device command and returns a _**single**_ response type.

  * `Ok`: indicates that the command was successfully executed on the device.
  * `Serial`: returns additional data related to device operation.
  * `Info`: provides metadata and other details about the device.
  * `Stream` (optional feature): delivers chunks of multimedia data as
  byte streams.
The sending of responses is implemented differently in each supported
firmware-side crate, while their reception in the controller remains consistent.
* **Provide a description of a device**, including its internal data,
available methods, and information about its economic and energy consumption.
* **Associate hazards with a route.** As mentioned earlier, hazards
are categorized into three types:

  * _**Safety**_ hazards refer to potential risks to human life.
  * _**Financial**_ hazards concern the economic impact of the device and
  its commands.
  * _**Privacy**_ hazards relate to issues involving the handling and management of
  sensitive data.

## Firmware Side

The firmware side provides the APIs required to develop a `tosca`-compliant
firmware device. These APIs are split into two library crates according to the
target hardware architecture:

* [tosca-os](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca-os)
* [tosca-esp32c3](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca-esp32c3)

The APIs are largely equivalent across the two crates, and both integrate the
`tosca` crate to maintain a common interface between firmware and controller
components.

The `tosca-os` library crate is designed for firmware running
on operating systems.

It offers APIs to:

* Define routes for commands, including support for `Stream` responses,
  which can be enabled via a feature.
* Encapsulate commands in functions invoked by the server when a route
  is called.
* Configure the **_mDNS-SD_** discovery service.
* Initialize and run an **_HTTP_** server.

Using these APIs, we have implemented firmware for a
[smart light](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca-os/examples/light)
and an
[IP camera](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca-os/examples/ip-camera).

Instead, the `tosca-esp32c3` library crate is designed for building firmware
that runs on **_ESP32-C3_** microcontrollers.

It provides APIs to:

* Connect a device to a **_Wi-Fi_** access point.
* Build and configure the network stack.
* Define routes for commands. `Stream` responses **_are_** not supported.
* Encapsulate commands in functions invoked by the server when a route
  is called.
* Configure the **_mDNS-SD_** discovery service.
* Declare events for specific route tasks.
* Initialize and run an **_HTTP_** server.

To showcase the capabilities of these APIs, we developed several
[light firmware](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca-esp32c3/examples)
examples that progressively increase in complexity.

Finally, we developed a library crate that provides architecture-agnostic
implementations for sensors without dedicated drivers:
[`tosca-drivers`](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca-drivers).
While more complex sensor drivers can be imported directly into
firmware, the goal of this library is to provide simple, reusable modules that
can be shared across all `tosca` architectures.

## Controller Side

The controller side provides the APIs needed to build software capable of
interacting with all `tosca`-compliant firmware devices on the same network.
To use these APIs, a controller application must simply include the
[`tosca-controller`](https://github.com/ToscaLabs/tosca/tree/v0.1.1/crates/tosca-controller)
library crate as a dependency.

The core functionalities of this crate include:

* **Discovering devices**: automatically discovering all `tosca`-compliant
firmware devices available on the network.
* **Issuing device commands**: from a device’s description, constructing the
corresponding _REST_ requests and sending them over the network to invoke
the device's commands.
* **Enforcing security and privacy policies**: define policies to allow or block
requests for commands that have determined hazards.
* **Subscribing to device events broker**: intercepting events published by
a device to its broker. These events represent the internal state of the
device. The controller subscribes to the broker using the IP address or URL
specified in the relevant field of the device description.

# Conclusions & Future Plans

`tosca` represents a new approach to connecting firmware devices with a
controller. It leverages cutting-edge technologies to build components that
are reliable and less error-prone.
One of the framework’s main features is the provision of clear and easily
understandable APIs that guide developers in building the crates of their
IoT systems. These APIs are designed to resemble natural language as closely as
possible, making them intuitive and easy to use.

As future work, we plan to release the `0.2` version of the framework with the
following improvements:

* **Contract-based `tosca` crate**: extend the `tosca` library crate so that it
serves as a shared contract between a firmware binary crate and a controller
binary crate. In this way, developers do not need to modify the
framework to change hard-coded values or introduce additional interfaces.
Instead, they can rely on a minimal shared interface.
* **Support for dynamic device behavior**: devices may join or
leave the network at any time. The `tosca-controller` should be able to
reliably detect these changes and provide mechanisms to register and manage
the behaviors of connected devices.
* **Secure communication**: implement secure communication between firmware
devices and controllers. Currently, no security mechanisms exist, and all
exchanged messages are transmitted in plaintext, making the infrastructure
vulnerable.
* **API unification across device crates**: uniform the APIs of the `tosca-os`
and `tosca-esp32c3` crates. Currently, some features are implemented only for
specific targets. For example, the event manager exists in `tosca-esp32c3` but
not in `tosca-os`.

# Acknowledgments

Along the journey to the first release of `tosca`, I have not been alone.

I would like to sincerely thank:

* [@alexle0nte](https://github.com/alexle0nte): for contributing to the
development of numerous features over the past year and for his
invaluable advice.
* [@giusbianco](https://github.com/giusbianco): for his continuous support and
for helping me in shaping the framework’s design during the early stages of
development.

Thank you, guys!
