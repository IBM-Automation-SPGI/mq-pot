# Overview - MQ on CP4I PoT

This guide and associated standard Virtual Desktop Instance (VDI) help you get started with IBM MQ on IBM Cloud Pak for Integration (CP4I). This is not a primer for MQ basics. The intent is to show how to create and operate MQ queue managers on IBM Cloud Pak for Integration running on a Red Hat OpenShift cluster.

You will learn how to create a simple queue manager, an MQ cluster using the latest MQ feature Uniform Clusters and a highly available queue manager using the *NativeHA* technology.

For the MQ on CP4I PoT, multiple users will create each of the above sharing one CP4I on a single Red Hat OpenShift cluster.

## Names used in lab

| student number | QM prefix | Lab 1 QM Name|  Lab 3 QM Names|  Lab 5 QM Name |
|:--------------:|:---------:|:------------:|:-------------:|:-------------:|
| 01             | mq01      | mq01qs       | mq01a, b, c   | mq01ha        |
| 02             | mq02      | mq02qs       | mq02a, b, c   | mq02ha        |
| 03             | mq03      | mq03qs       | mq03a, b, c   | mq03ha        |
| 04             | mq04      | mq04qs       | mq04a, b, c   | mq04ha        |
| 05             | mq05      | mq05qs       | mq05a, b, c   | mq05ha        |
| xx...          | mqxx...   | mqxxqs...    | mqxxa, b, c...| mqxxha...     |

## Setup instructions

### PoT Student instructions

The PoT coordinator will provide to you your student number and links to a VDI Linux desktop to be used for accessing the CP4I cluster and executing the lab instructions.

Once you have the required information navigate to the environment setup:
[Environment Setup](../envsetup/mq_cp4i_pot_envsetup.md)

## End of PoT overview and setup

You are now ready to setup for the labs.   

[Environment Setup](../envsetup/mq_cp4i_pot_envsetup.md)

[Return to main lab page](../index.md)
