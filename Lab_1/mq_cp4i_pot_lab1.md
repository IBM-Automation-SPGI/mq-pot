# Lab 1 - Creating an MQ Instance Using the Platform Navigator

[Return to lab index page](../index.md)

Starting with this lab, each attendee will be assigned an ID number (01 - 30) by the instructor if running a PoT.

## Getting Started with MQ on Cloud Pak for Integration

These instructions document how to setup MQ within Cloud Pak for Integration which is accessible from within the OpenShift Cluster. The instructions have been created using a a Red Hat OpenShift environment deployed on bare metal servers on IBM Cloud however the process should be similar on other environments.

### Important points

The lab guide assumes you are using the RHEL Virtual Desktop Image (VDI) VM from the IBM Technology Zone. If you are using another platform, you can download the necessary artifacts from the github repo. The instructor will provide directions.

If running as part of a PoT, you will only see your project (namespace). The name will be of the form student + your student number. For instance if your student number is 10, your namespace will be student10. So each attendee has a unique namespace and will only be authorized to see that namespace. You will also find the Platform navigator instance, you SHALL NOT EDIT NOR DELETE this instance.

Important - "You will see other projects such as cp4i. Since this cluster will be shared with other PoTs those have been predefined. They are not to be used for this PoT and can be ignored. You will only use your assigned namespace and at times cp4i."

Tip - "The screen shots were taken on a test cluster and many will not match what you see when running the lab. Particularly URL values will be different depending on the cluster where CP4I is running. Projects (Namespaces) may also vary. It is important to follow the directions, not the pictures."

## Deploying IBM MQ for internal consumers

1. Open a browser and go to the *CP4I Navigator*  using the URL provided by instructors.

1. If prompted with the potential security risk, click *Advanced*.

	![](./images/image204.png)

1. Scroll down and click the *Accept the Risk and Continue* button.

	![](./images/image205.png)

	If prompted again, respond as you did above.

1. Finally you are routed to the *Log in to IBM Cloud Pak* page. Enter the user name and password that was provided to you to log on.

	![](./images/image206.png)

1. The *Cloud Pak Home* page appears. This is also referred to as the *Platform Navigator*. Here you can see the various runtimes available in CP4I. Click *Create an Instance*.

	![](./images/image207.png)

1. A number of tiles are displayed. Click the *Queue Manager* tile, then click *Next*.

	![](./images/image209.png)

1. You will use the option called **Quick start** which will deploy an MQ container with 0.5 cpu and 1 GB of memory. Click the *Quick start* tile then click *Next*.

	![](./images/image210.png)

1. On the *Create queue manager* configuration page, enter "mq" plus your student number as a prefix to the Name *quickstart-cp4i* (for instance mqxx-quickstart-cp4i where xx is student number) and use the drop-down under *Namespace* to select your assigned target namespace(for instance studentXX). Click the License acceptance checkbox to turn it on.

	![](./images/image211.png)

1. Scroll down to *Queue Manager* section. Using the drop-down under *Type of availability* select **SingleInstance**. Under *Type of volume* leave the default **ephemeral**. You have a choice between ephemeral or persistent-claim. Ephemeral means that the queue manager does not maintain state so does not need persistent-storage.

	![](./images/image104.png)


1.	On the left side bar, click the button under *Advanced settings* to expose more settings. You may need to scroll to find the *Queue Manager* section again. Under *Advanced: Name* field you see the default **QUICKSTART**. Replace this with your queue manager name using your student ID and qs, ie **mqxxqs** (Where xx is your student number).

	Now click *Create*.

	![](./images/image212.png)

1. If all entries are valid, you will receive a notification of success in green. Any errors result in a notification in red. Status remains *Pending* while the queue manager is being provisioned. If you click *Pending* you may receive a *Conditions* pop-up.

	![](./images/image8a.png)

	Close the pop-up.

	![](./images/image8a1.png)

1. Click refresh button until you see you MQ instance to be **Ready**. You can use the filter feature  to limit the showed instances to those that are on your namespace.

	![](./images/image220.png)


<a name="mqconsole"></a>
### Start MQ Console

1. Once your queue manager shows a *Ready* status, you can now click the hyperlink to open the MQ console for your queue manager. The runtime instance shows *mqxx-quickstart-cp4i* (where xx is your student number), but that is not your queue manager name. You named it **mqxxqs**.

	![](./images/image223.png)

	If you receive a potential security risk warning. Click *Advanced* then *Accept the risk* as you did before.

	The MQ Console looks nothing like MQ Explorer. It doesn't even look like earlier versions of MQ Console. Feel free to poke around (or click around) and explore the various tiles and side bar menus. When you are ready, continue to the next step - creating a queue.

1. 	Click on the *Manage* menu, then click *Queues* tab, and finally the *Create* button.

	![](./images/image224.png)

1. A number of tiles are displayed for the different queue types. Behind each tile are the properties for that particular queue type. Select the tile for a *Local* queue and click the *Next* button.

	![](./images/image225.png)

1. Only the required basic options are displayed with the required field *Queue name*. If you need to alter or just want to see all available options, you can click the *Custom create* tab. For now, just enter the name for the test queue **app1** and click *Create*.

	![](./images/image226.png)

1. An MQ channel needs to be defined for communication into MQ. Click the *Manage* option.

	![](./images/image301.png)

1. Select the *Applications* tab, click *App channels*, then click *Create +*.

	![](./images/image302.png)

1. Select the *Server Connection Channel* tile. Read the definition, then click *Next*.

1. To keep it simple enter the name of your queue manager (mqXXqs where XX is the student number provided by the instructor) so that the channel name matches the queue manager name. *Custom create* tab lets you provide detailed properties. Click *Create*.

	![](./images/image229.png)

1. The channel appears in the list.

	![](./images/image303.png)

## Test MQ

MQ has been deployed within the Cloud Pak for Integration to other containers deployed within the same Cluster. This deployment is NOT accessible externally. Depending on your scenario you can connect ACE / API Connect / Event Streams, etc to MQ using the deployed service. This acts as an entry point into MQ within the Kubernetes Cluster. Assuming you followed the above instructions within the deployment the hostname will be of the form mqXXqs-cp4i-ibm-mq. To verify the installation we will use an MQ client sample within the deployment.

1. Return to the OCP Console tab. Make sure you are in the *studentxx* namespace (Where xx is studen number), by clicking *Projects*, typing **student** in the filter field to find *studentxx* and clicking its hyperlink.

	![](./images/image237.png)

1. Once in the *studentxx* namespace, click the drop-down for *Workloads* then select *Pods*. Specify as Filter Name **mq** and you will see the one pod for your MQ instance. You will see that the *Status* is **Running** and there are one containers running in this pod.

	![](./images/image238.png)

1. Click the hyperlink for the pod. Now you will see quite a bit of details about the pod. Explore the details where you find graphs for memory and cpu usage, filesystem and network (Tab **metrics**). Come back to **Details** tab and scroll down and you will find the containers and volumes. From this panel you can drill down into any of these. But for now, you need to run a test. Click the *Terminal* tab which will automatically log you into the Queue Manager container.

	![](./images/image239.png)

1. By default MQ is secure and will block all communication without explicit configuration. Run the following commands to disable channel authentication (only for demo purposes).Don't forget to replace XX with your student ID.

	```
	runmqsc mqXXqs
	ALTER QMGR CHLAUTH(DISABLED) CONNAUTH(' ')
	REFRESH SECURITY TYPE(CONNAUTH)
	end
	```

1. Run the following commands to send a message to the **app1** queue. Don't forget to replace XX with your student ID.

	```
	export MQSERVER='mqXXqs/TCP/mqXX-quickstart-cp4i-ibm-mq(1414)'
	```

	The format of this environment variable is :
	*channel name / TCP / host name (port)*

	The host name is actually the network service for your queue manager instance.

	```
	/opt/mqm/samp/bin/amqsputc app1 mqXXqs
	```

	This command is putting a message on queue **app1** on queue manager **mqXXqs**.
	Type a message such as:'

	```
	sending my first test message to qm mqXXqs queue app1
	```

	Hit enter again to end the program.

	![](./images/image240.png)

1. Return to the MQ Console and navigate back to the queue manager view by clicking on **Manage**, and select again the **Queues** tab.

	![](./images/image241.png)

1. Notice the **app1** queue has a queue depth of 1 with a maximum queue depth of 50,000. Select the *app1* queue.

	![](./images/image305.png)

1. Click the hyperlink for queue **app1**. Here you see the messages on the queue. You will recognize your message under *Application data* along with application name and time stamp.

	![](./images/image243.png)


Congratulations! on completing Lab 1.

## Cleanup (Optional)

1. Return to the Platform Navigator. Find your instance, click the elipsis on the right and select **Delete**. This will delete the queue manager pods and all related artifacts. This will help reduce load on the cluster as you continue the rest of the labs. This queue manager will not be needed again.

	![](./images/image244.png)

1. You will be required to enter the name of your instance. Then the *Delete* button becomes active. You must now click the *Delete* button.

	![](./images/image245.png)

1. Your instance is now gone and you will receive a success pop-up.

	![](./images/image246.png)

[Continue to Lab 2](../Lab_2new/Readme.md)

[Return to lab index page](../index.md)
