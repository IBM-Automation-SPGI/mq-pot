# Lab 3 - MQ Uniform Cluster on CP4I
[Return to main lab page](../index.md)

Featuring:

 * Creating a Uniform Cluster
 * Application Rebalancing
 * Application Rebalancing & Queue Manager Outage
 * Metrics
 * Using CCDT Queue Manager Groups

## Introduction

This lab introduces MQ Uniform Cluster and Application Rebalancing as of MQ 9.3 code level. The lab can be run on any IBM Cloud environment running RedHat OpenShift 4.12 or above with IBM Cloud Pak for Integration (CP4I) 2023.2.1 or above.

In this lab, you will:

* Create a Uniform Cluster quickly using yaml configuration files consisting of three identical queue managers
* Run re-connectable sample applications to a queue manager within the Uniform Cluster to show automatic rebalancing of the apps to other queue managers in the Uniform Cluster

	![](./images/image00.png)

* Stop then restart a queue manager with connected apps to show automatic application rebalancing to remaining running queue managers in the Uniform Cluster
* Report resource usage metrics for the applications, introduced in MQ 9.1.5.
* (Bonus Track) Connect an application to a Queue Manager Group instead of a queue manager

## Pre-reqs

You should have already downloaded the artifacts for this lab in the lab Environment Setup. If you are doing this lab out of order return to [Environment Setup](../envsetup/mq_cp4i_pot_envsetup.md) to perform the download. Then continue from here.

### Important points to note

The lab guide assumes you are using the VDI VM (VNC Desktop) from IBM Assets. If you are using another platform, you can download the necessary artifacts from the github repo. The instructor will provide directions.

**IMPORTANT** "The screen shots were taken on a test cluster and many will not match what you see when running the lab. Particularly URL values will be different depending on the cluster where CP4I is running. Projects (Namespaces) may also vary. It is important to follow the directions, not the pictures."


### Further information

* IBM MQ Knowledge Center
* “Building scalable fault tolerant systems with IBM MQ 9.1.2 CD” article by David Ware, STSM, Chief Architect, IBM MQ:
[David Ware article](http://ibm.biz/MQ-UniCluster)
* “Active/active IBM MQ with Uniform Clusters” video by David Ware:
[YouTube Demo Video](https://www.youtube.com/watch?v=LWELgaEDGs0&feature=youtu.be)

## Review MQ Instances
1. Login into the RHEL desktop VM (VNC). Use credentials and url provided by the instructor.

	![](./images/image302c.png)

1. Open a Firefox web browser by double-clicking the icon on the desktop.

	![](./images/image302a.png)

1. Navigate to the URL for the OCP console and login using credentials provided by the instructor.
	![](./images/image304.png)

1. Then add another browser tab by clicking the "+" sign and opening the *CP4I Navigator* URL for the platform navigator provided by instructors. Select "Enterprise LDAP" as a login mechanism and use same credentials as in one step back.

	![](./images/image306.png)

1. Once the navigator opens, click the hamburger menu in the top left corner of the window, click the drop for *Administration* and select **Integration Instances**. You can click the *Show less* hyperlink to conserve screen space.

	![](./images/image306b.png)

1. If you ran the *MQ Cleanup* step in prior labs there should be none of your queue managers running. However there may be other's instances running. Also you may see Integration Design, Integraion Dashboard, Kafka Clusters, etc..  instances.

	![](./images/image308.png)

	If you have any remaining *mqxx* (xx = your student ID), delete them now (only MQ instances).



## Configure the cluster using yaml

1. Open a new terminal window by double-clicking the icon on the desktop or reuse existing one.

	![](./images/image1a.png)

1. Navigate to the */MQonCP4I/unicluster/* directory using the following commnand:

	```
	cd ~/MQonCP4I/unicluster
	```

1. There are two subdirectories, *deploy* and *test*. Change to the *deploy* directory.

	```
	cd deploy
	ls
	```

	![](./images/image309.png)

	*unicluster.yaml_template* contains the yaml code to define a cluster. *uni-install.sh* is a shell script which contains environment variables using your student ID and copies *unicluster.yaml_template* to *unicluster.yaml* and runs the openshift command to apply the definitions. *uni-cleanup.sh* is another shell script containing environment variables with your student ID and commands to delete your queue managers and related artifacts when you are finished.

1. Enter the following command to display the permissions for the files:

	```
	ls -al
	```

	![](./images/image301.png)

1. Make *uni-addqmgr.sh*, *uni-installl.sh* and *uni-cleanup.sh* executable with the following commands.

	```
	chmod +x *.sh
	```

	![](./images/image302.png)

2. Open an editor to review the *uni-install.sh* script with the following command:

	```
	gedit uni-install.sh
	```

	![](./images/image206.png)

	This terminal is running *gedit* and cannot be used for other commands while you are editing files.

	Note: To show line numbers in *gedit* click the drop-down in bottom right corner and select **Display line numbers**.

	![](./images/image302b.png)

3. Review the *uni-install.sh* shell script. The export commands define environment variables using your student ID to uniquely define your queue managers and related objects. This will help you identify and filter based on your student ID.

4. Change TARGET_NAMESPACE value to the project/target namespace provided by the instructor (for instance wedgeXX). There will be three queue managers, *mqxxa*, *mqxxb*, and *mqxxc* in your cluster *UNICLUSxx*.

	![](./images/image207.png)

5. One of the environment variables is *SC* for Storage Class. Change **managed-nfs-storage** value to **ocs-storagecluster-cephfs**. Notice: change first occurent of SC. Second one does it have an # (Comment).

	![](./images/image207b.png)

6. Review **VERSION** and **LICENSE** values. The instructor will provide you new values if changes are necessary.

	![](./images/image209b.png)

7. Enter your student ID in the *n* field, then click *Save*.

	![](./images/image209.png)

8. Click *Open* and select *uni-cleanup.sh*. Change the *n* field again as in you did for *uni-install.sh*

9.  Click *Open* and select *unicluster.yaml_template*.

	Note: If the file name does not appear in the drop-down list, click *Other* and navigate to the correct directory to find it.

	![](./images/image210.png)

10. Important: No changes are required for *unicluster.yaml_template*. Just only take a look on it. The template has definitions for all three queue managers. Your environment variables will be substituted throughout the file. If you execute a find for "$" you can easily locate the substitutions.

	Each queue manager has two *ConfigMap* stanzas, one *QueueManager* stanza, and one *Route* stanza. One *ConfigMap* is the mqsc commands for the queue manager - **uniform-cluster-mqsc-x** and one for the cluster ini file - **uniform-cluster-ini-x**.

	The queue managers share the same secret - lines 1 - 9.
	Queue manager **mqxxa** is defined on lines 11 - 116.
	Queue manager **mqxxb** is defined on lines 118 - 223.
	Queue manager **mqxxc** is defined on lines 225 - 325.
	![](./images/image211.png)

	Pay particular attention to the mqsc commands which define the cluster repository queue managers and the cluster channels.

11. Review the config map.

12. Open another terminal window and navigate to */home/student/MQonCP4I/unicluster/deploy*.

13. Go to your Firefox instance and select Openshift Console tab. If you are logged off, please log in again. Click on your username on the top right menu of the OpenShift Console, then click on *Copy Login Command*. If Openshift ask you again for user and password, please specify them

	![](./images/image00b.png)

14. Click *Display Token*, copy the token and run it on your terminal.

	![](./images/image0a.png)

	Run the following command to navigate to the **TARGET_NAMESPACE** project provided by the instructor to you:

	```
	oc project TARGET_NAMESPACE
	```

15. Enter the following command to create your uniform cluster.

	```
	./uni-install.sh
	```

	![](./images/image310.png)
16. Return to the *Platform Navigator* web browser page. In *Integration instances* click the *Refresh* button.

17. The queue managers will be in a *Pending* state for a couple of minutes while they are provisioned.

	![](./images/image311.png)
18. On the *OpenShift Console* you can watch the pods as they create containers. Check that you are in your assigned **TARGET_NAMESPACE**  Click *Workloads* then select *Pods*. You will see a pod for each queue manager and you will see states of *Pending*, *Container creating*, and then *Running*.

	![](./images/image312.png)

	![](./images/image313.png)

19. After a few minutes the queue managers will then show *Ready* on the *Platform Navigator* and the pods will show *Running* on the *OpenShift Console*.

	![](./images/image315.png)

	![](./images/image314.png)

20. Your cluster is also now completely configured. Check this from the *MQ Console* of one of thee queue managers (on the *Platform Navigator*). Click the hyperlink for **mqxxa** (Where xx is your stutent number)

	![](./images/image20.png)

21. You may be presented with a **"Error 502 - Bad Gateway"**, if so, wait a couple of minutes while the Queue Manager is being created. Then reload the page.

22. You may be presented with a warning pop-up. Click *Advanced*, then scroll down and click *Accept the Risk and Continue*.

	![](./images/image21.png)

23. In the *MQ Console* click *Manage mqxxa*. Of course your queue managers are different, xx being replaced by your student ID.

	![](./images/image22.png)

24. You will see the two local queues **APPQ** and **APPQ2** which were defined by the mqsc *ConfigMap* defined in the yaml template. The other queue managers also have the queues by that name.

	![](./images/image400.png)

	Click **View configuration**.

25. Click **Listeners**. The listener *SYSTEM.LISTENER.TCP.1* is running.

	![](./images/image401.png)

	Go back to *Manage* and click *Appplications* -> *App channels*.

26. *App channels* are actually *SVRCONN* channels. You will have two defined within the yaml file. **MQXXCHLA** and **TO_UNICLUSXX** (where xx is your student id). These will be used during testing in the next section.

	![](./images/image402.png)

	Click *MQ network* -> *Queue manager channels*.

27. Here you find your cluster channels. If you looked closely at the yaml template, you'll remember that your *mqxxa* and *mqxxb* are the primary repositories for your cluster *UNICLUSxx*. While looking at *mqxxa* you see a cluster receiver channel **TO_UNICLUS_MQ00A** and two cluster sender channels **TO_UNICLUS_MQ00B** and **TO_UNICLUS_MQ00C**. They should be *Running*. Remember that xx is your Student Id.

	![](./images/image404.png)

	Click *View Configuration* in the top right corner.

28. The queue manager properties are displayed. Click *Cluster* to see the cluster properties where you see your cluster name - UNICLUSxx.

	![](./images/image27.png)

29. You can check the other queue manager's console to verify that they are all configured the same. You should have verified that when reviewing the yaml template.

	You are all set, time for testing.

## Test uniform cluster

### Perform health-check on Uniform Cluster
Before proceeding, we need to check the cluster is up and running.

1. Open another terminal window and start MQ Explorer with the following command.

	```
	MQExplorer
	```
1. Starting from mqxxa, you must add your queue managers to the MQ Explorer. Right click in **Queue Managers**  and select **Add Remote Queue Manager**.

	![](./images/image214a.png)

2. Specify <span style="color:red">**mqXXa**</span> as queue manager name (remember that xx is your student id).Click **Next** Button.

	![](./images/image214b.png)

3. Specify:

	- Hostname to mq**XX**a-ibm-mq-qm-**TARGET_NAMESPACE**.apps.**CLUSTER_NAME**.cloud.techzone.ibm.com. Change **XX** to your student id, change **TARGET_NAMESPACE** to the one provided by the instructor and change **CLUSTER_NAME** to the one provided by the instructor. For instance if you are student **00**, your target_namespace is **student00** and your cluster name is **662fba8be89099001e8c335f** then your hostname will be **mq00a-ibm-mq-qm-student00.apps.662fba8be89099001e8c335f.cloud.techzone.ibm.com**). You can find this hostname in Openshift Console, select Networking and Routes. Filter by mq and select route for **mqxxa-ibm.mq-qm**.
	- Port number to **443**
	- Server-connection channel to **MQXXCHLA** where XX is your student id.

	![](./images/image214c.png)

	Click **Next Button**

4. Click **Next Button** on this step

	![](./images/image214d.png)

5. Click **Next Button** on this step

	![](./images/image214e.png)

6. Click the checkbox for Enable SSL key repositories.

	![](./images/image214f.png)

7. Click Browse for *Trusted Certificate Store* and navigate to "/home/student/MQonCP4I/tls" (file you downloaded for the workshop) and select **MQExplorer.jks**. Then click **Open**.

	![](./images/image139.png)

8. Click **Enter password** button and enter *password* for Password. Click **OK** then **Next**.

9. Click **Enable SSL Options** and select as SSL CipherSpec **ECDHE_RSA_AES_128_CBC_SHA256**

	![](./images/image214h.png)

	Click **Finish** button

10. Queue manager **mqxxa** should appear in the MQExplorer.

	![](./images/image214i.png)

11. Repeat the process for **mqxxb** and **mqxxc**.

   - For **mqxxb** use
     - Hostname to mq**XX**b-ibm-mq-qm-**TARGET_NAMESPACE**.apps.**CLUSTER_NAME**.cloud.techzone.ibm.com (change **XX** to your student id, change **TARGET_NAMESPACE** and **CLUSTER_NAME** to the ones provided to you by the instructor. For instance if you are student 00, ,your target namespace is student00 and your cluster name is 662fba8be89099001e8c335f then your hostname will be **mq00b-ibm-mq-qm-student00.apps.662fba8be89099001e8c335f.cloud.techzone.ibm.com**).
     - Port to **443**.
     - Server-Connection Channel to **MQXXCHLB** (Where XX is your student id).
     - Keep other parameters like for **mqxxa**.
   - For **mqxxc** use
     - Hostname to mq**XX**c-ibm-mq-qm-**TARGET_NAMESPACE**.apps.**CLUSTER_NAME**.cloud.techzone.ibm.com (change **XX** to your student id, change **TARGET_NAMESPACE** and **CLUSTER_NAME** to the ones provided to you by the instructor. For instance if you are student 00, ,your target namespace is student00 and your cluster_name is 662fba8be89099001e8c335f then your hostname will be **mq00c-ibm-mq-qm-student00.apps.662fba8be89099001e8c335f.cloud.techzone.ibm.com**).
     - Port to **443**.
     - Server-Connection Channel to **MQXXCHLC** (Where XX is your student id).
     - Keep other parameters like for **mqxxa**.

12. MQ Explorer should look like following image

	![](./images/image214.png)

13. Expand *Queue Manager Clusters* to confirm that queue managers **mqxxa** and **mqxxb** have full repositories, while **mqxxc** has a partial repository.

	![](./images/image215.png)

14. You observed the cluster channels in the MQ Console, but you can verify them in MQ Explorer also. Since *mqxxc* is a partial repository, it has two cluster sender channels, one to each full repository.

	![](./images/image31.png)

15. Check that Cluster Sender and Receiver Channels for each queue manager are running and if not, start them.

	![](./images/image30.png)

	**Note**: the Server Connection Channels will be inactive – do not attempt to start these.

16. Right-click you cluster name and select *Tests* > *Run Default Tests*.

	![](./images/image32.png)

17. Check that there are no errors or warnings resulting in the *MQ Explorer - Test Results*.

	![](./images/image33.png)

## Launch getting applications

In this section we shall launch 6 instances of an application connected to the same queue manager.

The Client Channel Definition Table (CCDT) determines the channel definitions and authentication information used by client applications to connect to a queue manager.
We shall be using a CCDT in JSON format.

1. Open a terminal window and navigate to */home/student/MQonCP4I/unicluster/test*. Copy the command snippet so you don't have to type the whole thing (you will need it in other terminals).

	```
	cd /home/student/MQonCP4I/unicluster/test
	```

1. Edit **ccdt.json** with the following command:

	```
	gedit ccdt.json
	```

	![](./images/image218.png)
1. Before you make any changes review the file observing:

	* the channel name matches the server connection channel on the queue manager
	* the host is in the format that you used in MQ Explorer but a different route
	* the port is the listener port for the queue manager
	* queue manager name
	* cipherspec

	![](./images/image34.png)

1. You only need to change the channel name, host, and queue manager name for each **channel** defined in this file. You must use same parameters as defined in the MQ Explorer. For MQ00CHLA change it to MQXXCHLA where XX is your student id, host value to mq**XX**a-ibm-mq-qm-**TARGET_NAMESPACE**.apps.**CLUSTER_NAME**.cloud.techzone.ibm.com and queue manager to **mqxxa** in all cases xx is the student id.

1. Do the same for MQ00CHLB change it to MQXXCHLB where XX is your student id, host value to mq**XX**b-ibm-mq-qm-**TARGET_NAMESPACE**.apps.**CLUSTER_NAME**.cloud.techzone.ibm.com and queue manager to **mqxxb** (in all cases xx is the student id)

1. Do the same for MQ00CHLC change it to MQXXCHLC where XX is your student id, host value to mq**XX**c-ibm-mq-qm-**TARGET_NAMESPACE**.apps.**CLUSTER_NAME**.cloud.techzone.ibm.com and queue manager to **mqxxc** (in all cases xx is the student id)

	![](./images/image38.png)

	Click *Save*.

1. In the editor, click the Open drop-down and select *.../unicluster/test/getMessage.sh*. You may need to click *Other documents* if it doesn't appear in the list.

	![](./images/image39.png)

1. Again, review this file before making any changes observing:

	* MQCHLLIB sets the folder containing the JSON CCDT file
	* MQCHLTAB sets the name of the JSON CCDT file
	* MQCCDTURL sets the address of the JSON CCDT file
	* MQSSLKEYR sets the location of the key
	* MQAPPLNAME gives a name to the application for displays
	* The shell will run the sample program *amqsghac* getting messages from queue *APPQ* on your queue manager

	Change the "00" in QMpre and QMname to your student ID, then click *Save*. Please keep QMname value as **mqxxa** where xx is your student id.

	![](./images/image219.png)

1. Open a new terminal window and enter following commands:

    ```
    cd /home/student/MQonCP4I/unicluster/test/
    ```

    ```
    chmod +x *.sh
    ```

    ```
    ./getMessage.sh
    ```

    ![](./images/image220.png)

    Execute command "./getMessage" in 5 new terminals so you will have six terminals running the program.

1. Notice that each time you open a new terminal and run the shell again the programs start to rebalance and reconnect.

	![](./images/image42.png)

1. Return to the OpenShift Console browser tab. Make sure you are in your **TARGET_NAMESPACE** project. Open *Wokrload > Pods*, use the filter to search for you queue managers, then click the hyperlink *mqxxa-ibm-mq-0* pod.

	![](./images/image44.png)

1. Click *Terminal*. This opens a terminal window in the container running inside that pod.

	![](./images/image45.png)
1. There is a new MQSC command, *DISPLAY APSTATUS*, which we shall now use to display the status of an application across all queue managers in a cluster.

	Start *runmqsc* with following command(remember to change 00 to your student id):
	```
	runmqsc mq00a
	```

	Run the display *APSTATUS* command:

	```
	DISPLAY APSTATUS(MY.GETTER.APP) TYPE(APPL)
	```

	*COUNT* is the number of instances of the specified application name currently running on this queue manager, while *MOVCOUNT* is the number of instances of the specified application name running on the queue manager which could be moved to another queue manager if required. You started the getMessage.sh six times.

	![](./images/image46.png)

	Click *Expand* to make the window larger. Repeat the command changing the *TYPE* to **QMGR**.

	```
	DISPLAY APSTATUS(MY.GETTER.APP) TYPE(QMGR)
	```

	At first, all instances of the application will be running on mqxxa and none on the other two queue managers. However, by the time you run this command, the instances will probably be shared across all queue managers as shown below.

	This display shows how the applications have been rebalanced. Notice that each queue manager in the cluster is now running two of the client applications making a total of six rebalanced.

	![](./images/image47.png)

	Click *Collapse* to return the window to normal size. Enter *end* to stop runmqsc.

	```
	end
	```

1. Some of the application instances will show reconnection events as the workload is rebalanced to queue managers *mqxxb* and *mqxxc*.

	![](./images/image48.png)

## Launch putting application

We shall launch another sample which will put messages to each queue manager in the cluster. The running samples should then pick up these messages and display them. In this lab, we are using one putting application to send messages to all getting applications using cluster workload balancing. You could set up the same scenario with one or more putting applications per queue manager and application rebalancing would work in the same way that you’ve seen for getting applications.

1. Open a new terminal window and navigate to */home/student/MQonCP4I/unicluster/test*. Open an edit session for *sendMessage.sh*. Review the export commands observing:

	* MQCHLLIB (sets the folder of the JSON CCDT file
	* MQCHLTAB sets the name of the JSON CCDT file
	* MQSSLKEYR sets the location of the key
	* MQAPPLNAME gives a name to the application for displays
	* The shell will run the sample program *amqsphac* putting messages to queue *APPQ* on your queue manager

	Change *00* to your student ID, then click *Save*.

	![](./images/image223.png)
1. We shall be using the sample amqsphac in this scenario. In the terminal window, enter the following command:

	```
	./sendMessage.sh
	```

	![](./images/image224.png)

1. You should now see the generated messages split across the getting application sessions that are running. Each window will contain a subset, like this:

	![](./images/image52.png)

	Note: the messages may not be evenly distributed across the getting applications instances. Some instances may not receive any message. The delivery of messages from the sendMessage app is slow enough that the “first” getter has already processed its message and has come back for more before the next message appears on the queue. MQ follows a model of LIFO when dispatching across multiple getters, i.e. it’ll give the next message to the last getter to come and ask for a message. This is actually more efficient when dispatching high volumes of data. However, it has the side effect of this perception that one getter is not able to receive messages, whereas they would if the put rate increases above what a single getter can process. This is tuneable. More info at: https://www.ibm.com/support/pages/node/6572739

## Queue Manager maintenance

In this scenario, imagine a queue manager needs to be stopped for maintenance purposes. We shall demonstrate how doing this will cause the applications running on that queue manager to "move" and run on the remaining active queue managers in the Uniform Cluster. Once the maintenance is complete, the queue manager will be re-enabled.

When a queue manager is ended, the applications on that queue manager are usually lost. However, if the optional parameter -r is used, the applications will attempt to reconnect to a different queue manager.

1. Return to the OpenShift Console tab in the web browser. You should still be in the **TARGET_NAMESPACE** Project. Click the drop-down for *Workloads* and select *Stateful Sets*. Filter on your **c** queue manager. Click the hyperlink for your the *Stateful Set*.

	![](./images/image54.png)

1. Click *YAML* which will open an editor for the *Stateful Set*. Scroll to the *spec* stanza at line 377. Change *replicas* to **0**. This will remove the active container in effect stopping the queue manager.

	![](./images/image225.png)

	Click *Save*.

1. Click *Save* again on the *Managed resource* pop-up.

	![](./images/image227.png)

1. Click *Pods* in the side-bar and notice that the pod for *mqxxc* has been terminated.

	![](./images/image56.png)

1. In the application windows, you'll notice that the application connected to *mq00c* are now trying to reconnect.

	![](./images/image57.png)

1. After a while, an application imbalance will be detected, and affected applications will be reconnected to the other available queue managers. To see this happening, re-run the MQSC command DISPLAY APSTATUS on any active queue manager in the cluster. After a minute or two you should see all application instances now running on mq00a and mq00b (where 00 is your student id):

	```
	runmqsc mq00a
	DISPLAY APSTATUS(MY.GETTER.APP) TYPE(QMGR)
	```

	![](./images/image60.png)

1. Once you are happy that the applications have balanced out equally across the other queue managers, re-start the stopped queue manager by changing the *spec > replicas* back to one in *Stateful Set*, **mq00c-ibm-mq**.

	![](./images/image226.png)

1. In the application windows, you will notice that the application connected to *mq00c* has now reconnected.

	![](./images/image59.png)

1. Display *apstatus* again and you'll see that the applications are again rebalanced.

	![](./images/image61.png)

## Metrics (new in 9.1.5)

The amqsrua sample application provides a way to consume MQ monitoring publications and display performance data published by queue managers. This data can include information about the CPU, memory, and disk usage. MQ v9.1.5 adds the ability to allow you to monitor usage statistics for each application you specify by adding the STATAPP class to the amqsrua command. You can use this information to help you understand how your applications are being moved between queue managers and to identify any anomalies.

The data is published every 10 seconds and is reported while the command runs.

Statistics available are:

* Instance count: number of instances of the specified application name currently running on this queue manager. See also COUNT from MQSC APSTATUS that we saw earlier.
* Movable instance count: number of instances of the specified application name running on this queue manager which could be moved to another queue manager if required. See also MOVCOUNT from MQSC APSTATUS that we saw earlier.
* Instance shortfall count: how far of the mean instance count for the uniform cluster that this queue manager’s instance count is. This will be 0 if queue manager is not part of a uniform cluster.
* Instances started: number of new instances of the specified application name that have started on this queue manager in the last monitoring period (these may have previously moved from other queue managers or be completely new instances).
* Initiated outbound Instance moves: number of movable instances of the specified application that have been requested to move to another queue manager in the last monitoring period. This will be 0 if the queue manager is not part of a uniform cluster.
* Completed outbound instance moves: number of instances of the specified application that have ended following a request to move to another queue manager. This number includes those that are actioning the requested move, or that are ending for any other reason after being requested to move (note that it does not mean that the instances have successfully started on another queue manager). This will be 0 if the queue manager is not part of a uniform cluster.
* Instances ended during reconnect: number of instances of the specified application that have ended while in the middle of reconnecting to this queue manager (whether as a result of a move request from another queue manager, or as part of an HA fail over).
* Instances ended: number of instances of the specified application that have ended in the last monitoring period. This includes instances that have moved, and those that have failed during reconnection processing.

1. In the browser tab for OpenShift Console stop *runmqsc* for *mqxxa* by entering *ctrl-C*. Then run the *amqsrua* command as follows, i.e. with a class of STATAPP, a type of INSTANCE, and object of your getting application name. Change *xx* to your student ID.

	```
	/opt/mqm/samp/bin/amqsrua -m mqxxa -c STATAPP -t INSTANCE -o MY.GETTER.APP
	```

	(Note: you can omit the class, type and object parameters and enter them when prompted instead).

	Initial stats are displayed and then updated every 10 seconds to show activity in the previous interval. You should see an Instance Count and Movable Instance Count of 2 as shown below. You may see different numbers for the other stats in the first interval, but these should be 0 in subsequent intervals.

	![](./images/image62.png)

	Refer to the description of these stats at the start of this section.
	Keep this command running.

1. Open a new browser tab and paste the Openshift cluster URL has been provided to you.  Change to the **TARGET_NAMESPACE**  project if not already there.

	![](./images/image229.png)

1. In this browser tab, expand *Workloads*, select *Stateful Sets*, then click the hyperlink for *mqxxc-ibm-mq*.

	![](./images/image228.png)

1. As you did previously, stop *mqxxc* by editing the *YAML* changing *spec > replcas* to zero. Click *Save*.

	![](./images/image230.png)

	Click *Save* on the *Managed resource* pop-up.
1. Refer back to the browser tab where you are running the *amqsrua* session. When the next update is shown, the following should have changed:

	**Instance Count & Movable Instance Count**
	there are now 3 instances of the application running on this queue manager;

	**Initiated & Completed Outbound Instance Moves, Instances ended**
	temporarily equal to 1 during the first interval as an instance is moved from *mqxxc* to this queue manager.

	![](./images/image65.png)

1. In the other console, restart mqxxc by editing the *YAML* changing *spec > replicas* back to one and clicking *Save*.
	![](./images/image67.png)

1. Again, refer back to the **amqsrua** session. When the next update is shown, the stats will have changed again. There are now 2 instances running on this queue manager, one having been moved (back) to *mqxxc*. As this happens, the numbers of moved and ended instances are again temporarily equal to 1.

	![](./images/image68.png)

1. Stop the **amqsrua** session when you are ready, using *ctrl-C*.

1. Stop the putting application with *ctrl-C*. And also stop all six of the getting application with *ctrl-C*. You can leave the terminal windows open as you will need them in the next secion.

## Bonus track

Next part of the lab is optional. Try to finish it if there is enough time left.

## Using CCDT Queue Manager Groups

So far we have connected our getting applications to *mqxxa* directly, and relied on the Uniform Cluster to rebalance them across the other queue managers over a period of time. There are 2 disadvantages to connecting in this way:

* When the applications initially connect, they all start out connected to *mqxxa* and there is a delay in the Uniform Cluster balancing them across the other queue managers
* If *mqxxa* is stopped unexpectedly or for maintenance, any applications connected to it will try to reconnect to *mqxxa* and fail. They will not attempt to connect to the other queue managers in the cluster. This will also be true if applications connected to other queue managers try to reconnect after an outage.

In this section, we shall see that by using Queue Manager Groups within our CCDT file we can decouple application instances from a particular queue manager and take advantage of the built-in load balancing capabilities available with CCDTs.

For a fuller description of the issues highlighted here, see step 5 of the following article:

[Walkthrough Uniform Cluster](https://community.ibm.com/community/user/integration/viewdocument/walkthrough-auto-application-rebal?CommunityKey=183ec850-4947-49c8-9a2e-8e7c7fc46c64&tab=librarydocuments)

### Stop queue manager - application refers to queue manager directly

Now that you know how to stop and start queue managers in CP4i, you will not receive the detailed instructions as previously.

1. Stop *mqxxa* by scaling the replicas in its *Stateful Set* to zero.

	![](./images/image231.png)

	Click *Save* on the *Managed resource* pop-up.

1. This time the getting application instances connected to *mqxxa* will continually try to reconnect to the stopped queue manager:

	![](./images/image70.png)

1. Now run the following MQSC command on any active queue manager in the cluster:

	```
	runmqsc mq00b
	DISPLAY APSTATUS(MY.GETTER.APP) TYPE(APPL)
	```
	After a while, there should be fewer than the 6 application instances that were originally present.

	![](./images/image71.png)

	If it still shows *COUNT(6)* then run the command again with *type(QMGR)*.

	```
	DISPLAY APSTATUS(MY.GETTER.APP) TYPE(QMGR)
	```

	![](./images/image232.png)

	You can see that there are four, two each on mq00b and mq00c, but none on mq00a.

1. Now restart *mqxxa* by scaling the replicas back up to one.

	![](./images/image233.png)

### Stop queue manager – application refers to CCDT Queue Manager Group

1. In the classroom environment, an updated CCDT file has been created for you to use: */home/student/MQonCP4I/unicluster/test/ccdt5.json*.

	Open this file in the editor. Change the **host** and **name** for every entry (MQ00CHLA to MQXXCHLA where XX is your student id, MQ00CHLB to MQXXCHLB where XX is your student id, host value to mqXXa-ibm-mq-qm-TARGET_NAMESPACE.apps.CLUSTER_NAME.cloud.techzone.ibm.com, etc...) values for each queue manager as you did in the *ccdt.json* file. There are 8 hosts parameters to change. As well as containing the original set of direct references to the queue managers, it gives a queue manager group definition with a route to all queue managers using the name **ANY_QM**.

1. Scroll down the file and note two new attributes:

	![](./images/image234.png)

	These are defined under *connectionManagement*:

	* **clientWeight**: a priority list for each client. The default value is zero. A client with a higher clientWeight will be picked over a client with a smaller value.
	* **affinity**: setting the affinity to “none” will build up an ordered list of group connections to attempt to try in a random order, for any clients on a particular named host.

1. Now let’s put the updated CCDT to the test. First, stop the 6 running getting application instances that you started earlier by entering *ctrl-C* in each terminal. You may leave the termninal running.

	![](./images/image74.png)

1. Please note: the supplied updated CCDT5 file was originally created for a scenario with an additional queue manager called *mqxxd*. For completeness, we shall create that missing queue manager now.

1. In your editor session (gedit), click *Open* > *Other documents* and navigate to
 */home/student/MQonCP4I/unicluster/deploy*, then select *uniaddqmgr.yaml_template* and click *Open*.

	![](./images/image75.png)

	Do not change anything in this file. Review it observing that it will create *qmxxd*, configmaps for *mqxxd*, and a route for *mqxxd*. It will use the same secret as the other three queue managers.

	![](./images/image76.png)

1. Open another file in the editor:

	*/home/student/MQonCP4I/unicluster/deploy/uni-addqmgr.sh*.

	Review the export commands observing:

	* MQCHLLIB sets the folder of the JSON CCDT file
	* MQCHLTAB sets the name of the JSON CCDT file
	* MQSSLKEYR sets the location of the key
  Change **TARGET_NAMESPACE** to the one provided by the instructor.

	As you did previously, replace n with your student ID.

	![](./images/image77.png)

	One of the environment variables is *SC* for Storage Class. Change it to use **ocs-storagecluster-cephfs** as *SC*.Please review **VERSION** and **LICENSE** parameters and change them if necessary. Instructor will provide you the values. Click **Save**.

	![](./images/image235.png)

1. In one of the terminal windows navigate to */home/student/MQonCP4I/unicluster/deploy/*.

	Enter the following command to create the new queue manager:

	```
	./uni-addqmgr.sh
	```

	![](./images/image78.png)

	Like *mqxxc*, it will have a partial repository.

	Note: You need to add *mqxxd* to MQExplorer to see it in the cluster display.

1. In the editor, open  */home/student/MQonCP4I/unicluster/test/sClient.sh*. Change *00* to your student ID. Change the path for *MQCHLTAB* and *MQCCDTURL* to **/home/student/MQonCP4I/unicluster/test/ccdt5.json**. Click *Save*.

	![](./images/image236.png)

	*ccdt5.json* includes *mqxxd* and entries for the queue manager group *ANY_QM*. The script will connect to an available queue manager and run the getting application *amqsghac*.

1. Open and make the same edits in *rClient.sh*.

	![](./images/image237.png)

1. In the editor, open file */home/student/MQonCP4I/unicluster/test/showConns.sh*. Make the necessary changes: 00 to your student ID and the paths for MQCHLTAB and MQCCDTURL to **/home/student/MQonCP4I/unicluster/test/ccdt5.json**.

   ![](./images/image238.png)

	Script *sClient.sh* will start the getting application *amqsghac* using *ccdt5.json* and will continue to run in that terminal. Script *rClient.sh* however, will start six more client applications running getting application *amqsghac* in the background. The main difference being that displays for those six clients will all be displayed in that single terminal.

1. Before you start the getting applications, you will want to start a script which displays the queue managers and the number of applications connected to it. Open four new terminal windows. In each one enter the following command where "xx" is equal to your student ID and "z" is equal to "a", "b", "c", or "d". Move to **test** directory if you are not there.

	```
	./showConns.sh mqxxz
	```

	![](./images/image239.png)

1. Sign and position those four windows so you can see the diplays:

	![](./images/image83.png)

1. Now you are ready to start the getting applications. Please open 6 new terminals. In each one of your open terminal windows, run the following command:

	```
	./sClient.sh
	```

	![](./images/image240.png)

	This time you are running the application with the queue manager group ANY_QM, prefixed with * which tells the client to connect to any queue manager in the ANY_QM group.

	Again, you will need to run this command 6 times.

1. Observe the behavior as the queue managers rebalance the connections. Watch the windows runnning the *showConns.sh* scripts as well as the windows where the getting applications are running. The application instances will now attempt to connect to any of the queue managers defined in the queue manager group, and with the client weight and affinity options defined above, we should see each application instance connect to one of the queue managers in the Queue Manager Group and Uniform Cluster.

	Eventually, the applications are evenly distributed across the queue managers.

	![](./images/image85.png)

	You can confirm this by watching the windows runnning the *showConns.sh* script or running the MQSC command DISPLAY APSTATUS on any active queue manager in the cluster.

1. In a command window, start the putting application by entering the following command:

	```
	./sendMessage.sh
	```

	![](./images/image86.png)

1. While *showConns.sh* displays show even distribution, you can also observe the application windows to see that the applications are getting an even distribution of messages.

	![](./images/image87.png)

1. Now end *mqxxa* to force the applications to be rebalanced:

	![](./images/image90.png)

	Rather than the applications getting stuck in a reconnect loop trying to connect to *mqxxa* as we saw using the previous version of the CCDT file, the applications now tied to the queue manager group ANY_QM will go through each of the definitions of ANY_QM and when able to successfully connect to one of the underlying queue managers, will do so. You should see this reported in a subset of the application instances:

	![](./images/image91.png)

1. Run the MQSC command DISPLAY APSTATUS on any active queue manager in the cluster (try it on mqxxd stateful set this time since it was added later). After a while, there should once more be 6 connections in total, as there were before *mqxxa* was shut down. Notice there are 6 total, but none on *mqxxa*

	```
	runmqsc
	```

	```
	DISPLAY APSTATUS(MY.GETTER.APP) TYPE(APPL)
	```

	```
	DISPLAY APSTATUS(MY.GETTER.APP) TYPE(QMGR)
	```

	![](./images/image92.png)

1. Restart *mqxxa*.

	![](./images/image93.png)

1. Open one more terminal window and run the following command:

	```
	./rClient.sh
	```
	Six more clients are started and you can see the messages that window is receiving.

	![](./images/image242.png)

1. Check the *showConns.sh* windows and you will see the applications evenly distributed again now totaling twelve.

	![](./images/image89.png)

## Congratulations

You have completed this lab Uniform Clusters and Application Rebalancing.


## Cleanup

1. Close all the applications and terminal windows.

2. Deleting the queue manager in the Platform Navigator does not delete PVCs. You should have edited *uni-cleanup.sh* for your student ID at the beginning of the file. If you didn't, please open */home/student/MQonCP4I/unicluster/deploy* in gedit now and make sure your student ID is at the beguining of the file.

	![](./images/image243.png)

3. In a terminal window, move to **deploy** directory and run the following command:

	```
	./uni-cleanup.sh
	```

	![](./images/image245.png)


[Return to lab index page](../index.md)
