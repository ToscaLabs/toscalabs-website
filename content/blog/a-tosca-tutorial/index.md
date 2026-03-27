+++
template = "post.html"

title = "A `tosca` tutorial"
description = """
A step-by-step tutorial for implementing a simple smart light firmware for an
`ESP32-C3` microcontroller, along with its corresponding
controller, starting from the basics. The tutorial is intended for readers who
are already familiar with the `Rust` programming language.
"""
date = 2026-03-15
tags = ["tosca", "tutorial", "smart-light"]
+++

_This blog post is written in English because `tosca` APIs and their
documentation are in English._

We aim to implement a complete IoT system using version `0.1.1` of the `tosca`
framework. This also offers a way to demonstrate the most commonly used APIs of
the framework and show how they can be combined.

As mentioned in a previous [post](..), the `tosca` framework provides APIs for
building both firmware devices and their corresponding controllers through
different crates.

At its current early stage of framework development, the `tosca-esp32c3` crate
supports only **light** devices, so **unknown devices** cannot yet be defined.
Meanwhile, `tosca-os` supports both **light** and **unknown** device types.

With version `0.2`, we plan to align API definitions across all supported
hardware architectures to provide a consistent and uniform interface.

In this step-by-step tutorial, we will use the `tosca-esp32c3` crate to build a
simple smart light firmware with just two commands: **on** and **off**.
We chose this crate to demonstrate an example developed for a real, concrete
embedded device, such as the `ESP32-C3`. We will then use the `tosca-controller`
crate to issue these commands.

Let’s start by implementing the firmware!

## Firmware

## Controller

# Conclusions

In this tutorial, we have seen how to implement a complete IoT system from
scratch. The APIs are still evolving and may undergo changes in the near future
to simplify some complex logic, making them more flexible through traits,
allowing the creation of custom devices.

If you have feedback to improve this tutorial, the APIs, or anything else,
feel free to open an issue in the
[`tosca`](https://github.com/ToscaLabs/tosca/issues) repository.

Thank you in advance!
