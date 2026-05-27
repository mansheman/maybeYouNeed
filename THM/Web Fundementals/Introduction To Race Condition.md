2025-07-16 20:17

Status:

Tags: [[THM Web Fundementals]] [[tryhackme]]
###### Prerequisites: [[Web Proxies]]
# Introduction To Race Condition

Let’s say we are tasked with testing the security of an online shopping web application. Many questions pop up. Can we reuse a single $10 gift card to pay for a $100 item? Can we apply the same discount to our shopping cart multiple times? The answer is _maybe_! If the system is susceptible to a race condition vulnerability, we can do all this and more.

This room introduces the race conditions vulnerability. A race condition is a situation in computer programs where the timing of events influences the behaviour and outcome of the program. It typically happens when a variable gets accessed and modified by multiple threads. Due to a lack of proper lock mechanisms and synchronization between the different threads, an attacker might abuse the system and apply a discount multiple times or make money transactions beyond their balance.

## Learning Objectives

After completing this room, you will learn about the following:

- Race conditions vulnerability
- Using Burp Suite Repeater to exploit race conditions

Along the way, you will also learn about:

- Threads and multi-threading
- State diagrams

## Learning Prerequisites

To follow this room, we recommend familiarity with the HTTP protocol, web applications, and Burp Suite. The following rooms and modules are recommended to fill any knowledge gaps.

- [How the Web Works](https://tryhackme.com/module/how-the-web-works)
- [Packets and Frames](https://tryhackme.com/room/packetsframes)
- [Burp Suite: The Basics](https://tryhackme.com/r/room/burpsuitebasics)