# IBM MQ Workshop

## Introduction
You can use IBM MQ to enable applications to communicate at different times and in many diverse computing environments.

What is IBM MQ?

   - IBM MQ is messaging for applications. It sends messages across networks of diverse components. Your application connects to IBM MQ to send or receive a message. IBM MQ handles the different processors, operating systems, subsystems, and communication protocols it encounters in transferring the message. If a connection or a processor is temporarily unavailable, IBM MQ queues the message and forwards it when the connection is back online.

   - An application developer has a choice of programming interfaces, and programming languages to connect to IBM MQ.

   - IBM MQ is messaging and queuing middleware, with point-to-point,publish/subscribe, and file transfer modes of operation. Applications can publish messages to many subscribers over multicast.

## Lab Abstracts

|  Subject                       | Description                                                                                         |
|--------------------------------|-----------------------------------------------------------------|
| [EnvSetup](Msg-Pre-lab/mqsetup/mq_setup_steps.md) | Environment Setup - Download lab artifacts   |
|--------------------------------|-----------------------------------------------------------------|
| [Lab 1](Lab_1/mq_cp4i_pot_lab1.md)       | Creating MQ Instance Using Platform Navigator                   |
|--------------------------------|-----------------------------------------------------------------|
| [Lab 2](Lab_2new/Readme.md)       | Native HA Queue Manager on Cloud Platforms             |
|--------------------------------|-----------------------------------------------------------------|
| [Lab 3](Lab_3new/Readme.md)       | Uniform Cluster on CP4I and Application Load Balancing                      |
