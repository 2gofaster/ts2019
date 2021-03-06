.. _xplay:

------------------------
Prism Pro: X-Play
------------------------

*The estimated time to complete this lab is 60 minutes.*

.. raw:: html

  <iframe width="640" height="360" src="https://www.youtube.com/embed/7Uh-5XbDk_Y?rel=0&amp;showinfo=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Overview
++++++++

Prism Pro is a product designed to make our customer IT operations smarter and automated through machine intelligence and automation. Today, there is no IT operations management (ITOM) solution that is specifically designed for the data center built around HCI. The modern data center is dynamic, scalable, and highly performant. Traditional ITOM, performance monitoring, and IT OPS tools are built for the static infrastructure. When being used in the modern data center, these tools can overwhelm IT admins with complexity and noisy signals. This decreases operational productivity and can reduce the ROI from adopting HCI.

Prism Pro takes a unique approach that maximizes the operational efficiency of a Nutanix based data center. Prism Pro uses purpose-built machine learning (X-Fit) to extract insights from massive amounts of operational data to guide capacity forecasting and planning, VM right-sizing, and anomaly detection - increasing productivity and eliminating waste.

In 5.11, Prism Pro introduces an automation mechanism (X-Play) that enables customers to automate operations and respond to signals generated by X-Fit.

X-Play is designed to address the #1 pain point when customers deal with automation, the fear of amplified impact because of complexity of the automation.
Unlike solutions such as Calm, which focus on application lifecycle automation, X-Play’s goal is to automate infrastructure tasks that admins face daily.
To eliminate the fear and give the control back to the admin, X-Play takes the codeless approach, a model proven by companies such as IFTTT and Zapier, making it versatile and easy to adopt.

The power of X-Fit and X-Play allows the customer to easily leverage machine data produced by their infrastructure, and operate it efficiently, confidently, and intelligently.

**In this lab you will create multiple different X-Play alert policies and playbooks to become familiar with the functionality and ease of use offered by this Prism Pro feature.**

Lab Setup
+++++++++

This lab requires applications provisioned as part of the :ref:`linux_tools_vm`.

If you have not yet deployed this VM, see the linked steps before proceeding with the lab.

Enabling X-Play
...............

X-Play is available as a hidden, Tech Preview in AOS 5.10 and will GA in AOS 5.11. Follow the steps below to enable X-Play on AOS 5.10:

#. Open **Google Chrome** and select **View > Developer > Developer Tools**.

   .. figure:: images/xplay_42.png

#. Click **Application**, and expand **Local Storage** under **Storage** on the right pane.

#. Add the following entry by double-clicking:

   - **nutanix_ui_release**  - 5.11

   .. figure:: images/xplay_43.png

#. Close **Developer Tools**.

#. Browse to your Prism Central IP and refresh the page.

Automatically Add Memory to a VM When A Constraint is Detected
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

How often have you been on-call, and got that alert or service ticket for a VM that had high memory or CPU usage?
Chances are a lot, and generally during dinner, while you are out with family, or sleeping.

What if you could use X-Play in Prism Pro to automatically take care of this for you when Prism Pro detected the constraint?

Good news, you can. Let's walk through how to set that up.

Run Stress Test
...............

Lets add some load by initiating a stress test.

#. Login to the *Initials*\ **-Linux-ToolsVM** via ssh or Console session.

   - **Username** - root
   - **password** - nutanix/4u

#. Execute the following to generate memory load:

   .. code-block:: bash

     stress -m 4 --vm-bytes 500M -t 40m &

   .. note::

     It will take approximately 5 minutes for **stress** to generate the memory load to cause the alert.

Create Alert Policy
...................

#. In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > VMs**, and click *Initials*\ **-Linux-ToolsVM**.

#. Select **Metrics > Memory Usage**.

   .. figure:: images/xplay_01.png

#. Click **Alert Settings**

   .. figure:: images/xplay_02.png

#. In the **Create Alert Policy** window, fill out the following fields:

   - **Entity Type** - VM
   - **Entity (Line 1)** - One VM
   - **Entity (Line 2)** - *Initials*\ **-Linux-ToolsVM**
   - **Metric** - Memory Usage
   - **Impact Type** - Performance
   - **Policy Name** - *Initials* - VM Memory Constrained
   - **Description** - Optional
   - **Auto Resolve Alerts** - Checked
   - **Enable Policy** - **Unchecked**
   - **Trigger alert if conditions persist for** - 0 Minutes

   - **Behavioral Anomaly**
       - **Every time there is an anomaly, alert** - Checked / Warning

   - **Static Threshold**
       - **Alert Critical if** - Checked / >= 60

   .. figure:: images/xplay_03.png

#. Click **Save**.

   .. note::

     Customers can choose out-of-the-box alert policies (shown below) to detect the memory and cpu constraint by Prism Pro X-Fit.

     .. figure:: images/xplay_04.png

Create Playbook
...............

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

   .. figure:: images/xplay_05.png

#. Click **Create Playbook**.

#. Select :fa:`bell` **Alert** as Trigger, and click **Select**.

   .. figure:: images/xplay_06.png

   .. note::

     When X-Play is GA in 5.11, we will also support a new trigger type “Manual” which allows you associate a playbook to VMs, Hosts, and Clusters and trigger it manually.

     .. figure:: images/xplay_07.png

#. Start typing to search for “VM Memory Constrained” in **Alert Policy**, and select *Initials* - **VM Memory Constrained**.

   .. figure:: images/xplay_08.png

#. Click **Add Action**, and select the :fa:`camera` **VM Snapshot** action.

   .. figure:: images/xplay_09.png

#. Select **Source Entity** from the parameters.

   Source entity refers to the entity that triggered the alert.

   .. figure:: images/xplay_10.png

   - **Target VM** - {{trigger[0].source_entity_info}}
   - **Time To Live**  - 1 day(s)

   .. figure:: images/xplay_11.png

#. Click **Add Action**, and select the :fa:`memory` **VM Hot Add Memory** action.

#. Select **Source Entity** from the parameters.

   - **Target VM** - {{trigger[0].source_entity_info}}
   - **Add Absolute Memory** - 1 GiB
   - **Absolute Maximum** -  20 GiB

   .. figure:: images/xplay_12.png

#. Click **Add Action**, and select the :fa:`envelope` **Email** action.

   .. note::

     Try creating your own alert messages using available parameters.

   - **Recipient** - YourEmail@nutanix.com
   - **Subject** - Playbook {{playbook.playbook_name}} addressed alert {{trigger[0].alert_entity_info.name}}
   - **Message** - Prism Pro X-FIT detected  {{trigger[0].alert_entity_info.name}} in {{trigger[0].source_entity_info.name}}.  Prism Pro X-Play has run the playbook of "{{playbook.playbook_name}}". As a result, Prism Pro increased 1GB memory in {{trigger[0].source_entity_info.name}}.

   .. figure:: images/xplay_13.png

#. Click **Add Action**, and select the **Acknowledge Alert** action.

#. Select **Alert** from the parameters.

   .. figure:: images/xplay_14.png

   - **Target Alert**  - {{trigger[0].alert_entity_info}}

#. Click **Save & Close**, and fill out the following fields:

   - **Name**  - *Initials* - Auto Remove Memory Constraint
   - **Description** - Optional
   - **Status**  - Enabled

   .. figure:: images/xplay_15.png

#. Click **Save**.

Cause Memory Constraint
.......................

#. In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > VMs**, and click *Initials*\ **-Linux-ToolsVM**.

#. Take note of your *Initials*\ **-Linux-ToolsVM** VM's memory capacity (should be 2 GiB).

#. Click **Alerts**, Select **Alert Policy** from **Configure** drop-down menu.

   .. figure:: images/xplay_16.png

#. Select *Initials* - **VM Memory Constrained**, and **Enable** the policy.

   .. figure:: images/xplay_17.png

#. Open a console session or SSH into Prism Central, and run the **paintrigger.py** script:

   - **Username** - nutanix
   - **password** - nutanix/4u

   .. code-block:: bash

     python paintrigger.py

   .. note::

     This script resolves all current alerts an forces an NCC check to run immediately, triggering the alert. As previously mentioned, in 5.11 you will have the ability to manually trigger the alert through Prism Central.

   After 2-5 minutes you should receive an email from Prism.

#. Verify the email reflects the subject and body relevant to the parameters configured earlier in the lab.

#. Verify that the memory capacity on your *Initials*\ **-Linux-ToolsVM** VM has increased.

Review the Playbook Play
........................

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

#. Select your *Initials* - **Auto Remove Memory Constraint**, and **Disable** it.

#. Click **Plays**.

   You should see that a Play has just completed.

#. Click the Play, and examine the details.

   .. figure:: images/xplay_18.png

Reset VM Memory
...............

#. Change your *Initials*\ **-Linux-ToolsVM** memory back to 2GB.

Reduce CPU Capacity for a VM During a Maintenance Window
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

X-Fit in Prism Pro utilizes Machine Learning to continually analyze the environment. This is helpful to detect resource constraints, such as our memory constraint in the last lab, or inefficiencies such as VMs with too many vCPUs or too much memory.

In this exercise we will create a playbook to take care of over-provisioned CPU.

Create Alert Policy
...................

#. In **Prism Central** > select :fa:`bars` **> Activity > Alerts**, and select **Alert Policy** from **Configure** drop-down menu.

#. Click **+ New Alert Policy**.

   .. figure:: images/xplay_19.png

#. In the **Create Alert Policy** window, fill out the following fields:

   - **Entity Type** - VM
   - **Entity (Line 1)** - One VM
   - **Entity (Line 2)** - *Initials*\ **-Linux-ToolsVM**
   - **Metric** - CPU Usage
   - **Impact Type** - Performance
   - **Policy Name** - *Initials* - VM CPU Overprovisioned
   - **Description** - Optional
   - **Auto Resolve Alerts** - Checked
   - **Enable Policy** - **Unchecked**
   - **Trigger alert if conditions persist for** - 0 Minutes

   - **Static Threshold**
       - **Alert Critical if** - Checked / <= 30

   .. figure:: images/xplay_20.png

#. Click **Save**.

Create Playbook
...............

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

#. Click **Create Playbook**.

#. Select :fa:`bell` **Alert** as Trigger, and click **Select**.

#. Start typing to search for “VM CPU Overprovisioned” in **Alert Policy**, and select *Initials* - **VM CPU Overprovisioned**.

#. Click **Add Action**, and select the :fa:`power-off` **Power Off VM** action.

#. Select **Source Entity** from the parameters.

   - **Target VM** - {{trigger[0].source_entity_info}}
   - **Type of Power Off Action**  - Power Off

#. Click **Add Action**, and select the **VM Reduce CPU** action.

#. Select **Source Entity** from the parameters.

   - **Target VM** - {{trigger[0].source_entity_info}}
   - **vCPUs to Remove**  - 1
   - **Minimum Number of vCPUs**  - 1
   - **Cores per vCPU to Remove**  - Leave Blank
   - **Minimum Number of Cores per vCPU**  - Leave Blank

     .. figure:: images/xplay_21.png

#. Click **Add Action**, and select the :fa:`power-off` **Power On VM** action.

#. Select **Source Entity** from the parameters.

   - **Target VM** - {{trigger[0].source_entity_info}}

#. Click **Add Action**, and select the :fa:`envelope` **Email** action.

   In many Environments, a production VM can not be powered off to alter the VM configuration. X-Play provides a way for the administrator to specify the time window where the actions can be executed.

#. Click **Restrict**.

   .. figure:: images/xplay_22.png

#. Configure the start time for ~5 minutes after your current time.

   .. figure:: images/xplay_23.png

#. Click **Set Restriction**.

   The **Restrict** label will change to **Restriction Set**. If you hover the mouse, you will see the schedule you just set.

   .. note::

     The steps above illustrate the way you can achieve this in 5.10 EA. In GA there will be three action types that will replace and enhance the **Restrict**:

      **Wait for Some Time**

     .. figure:: images/xplay_24.png

     **Wait until Day of Month**

     .. figure:: images/xplay_25.png

     **Wait until Day of Week**

     .. figure:: images/xplay_26.png

     These action types can be used the same as any other regular action type in any part of the Playbook. These restrictions can help support both planned maintenance windows and human approval process for playbook execution.

#. Click **Save & Close**, and fill out the following fields:

   - **Name**  - *Initials* - Reduce VM CPU
   - **Description** - Optional
   - **Status**  - Enabled

#. Click **Save**.

Cause CPU Over-Provision
........................

#. In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > VMs**, and click *Initials*\ **-Linux-ToolsVM**.

#. Take note of your *Initials*\ **-Linux-ToolsVM** VM's CPU Cores (should be 2).

#. Click **Alerts**, select **Alert Policy** from **Configure** drop-down menu.

#. Select *Initials* - **VM CPU Overprovisioned**, and **Enable** the policy.

#. Open a console session or SSH into Prism Central, and run the **paintrigger.py** script:

   - **Username** - nutanix
   - **password** - nutanix/4u

   .. code-block:: bash

     python paintrigger.py

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

#. Select your *Initials* - **Reduce VM CPU -**, and click **Plays**.

   You should see that there is a Play with your initials in **Scheduled** status.

#. Wait for 1-2 minutes past the previously configured start time and you should receive an email from Prism.

#. Verify the email reflects the subject and body relevant to the parameters configured earlier in the lab.

#. Verify that the CPU cores on your *Initials*\ **-Linux-ToolsVM** VM have been reduced.

   This indicates that the trigger happened and the rest of the Play is waiting for the window to execute. You can select this Play and abort it from the **Action** menu.

Review the Playbook Play
........................

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

#. Select your *Initials* - **Reduce VM CPU**, and **Disable** it.

#. Click **Plays**.

   You should see that the Play has just completed.

#. Click the Play, and examine the details.

Things to do Next
+++++++++++++++++

As you can see, X-Play paired with X-Fit is very powerful. You can go to **Action Gallery** page and familiarize yourself with all the out-of-the-box Actions to see all the possible things you can do.

#. In **Prism Central** > select :fa:`bars` **> Operations > Actions Gallery**.

   .. figure:: images/xplay_27.png

Use X-Play with Other Nutanix Products
++++++++++++++++++++++++++++++++++++++

Let's see how we can use X-Play with other Nutanix products by creating a Playbook to automatically quarantine a bully VM.

#. Login to the *Initials*\ **-Linux-ToolsVM** via ssh or Console session:

   - **Username** - root
   - **password** - nutanix/4u

#. Make sure NODE_PATH has the global nodejs module directory by running the following command to set it:

   .. code-block:: bash

     export NODE_PATH=/usr/lib/node_modules

#. Within *Initials*\ **-Linux-ToolsVM**, download the :download:`processapi.js <processapi.js>` file:

   .. code-block:: bash

     curl -L https://s3.amazonaws.com/get-ahv-images/processapi.js -o processapi.js

#. Modify the PC IP address and username/password in the script.

   .. code-block:: bash

     sed -i 's/127.0.0.1/<*your PC IP*>/g' processapi.js

     sed -i 's/pc user/admin/g' processapi.js

     sed -i 's/pc password/<*your PC password*>/g' processapi.js

#. Start the nodejs server

   .. code-block:: bash

     node processapi.js&

#. Run the stress command to simulate the IO load

   .. code-block:: bash

     stress -d 2

#. Keep ``stress`` running until you complete this exercise.

Create Alert Policy
...................

#. In **Prism Central** > select :fa:`bars` **> Activity > Alerts**, and Select **Alert Policy** from **Configure** drop-down menu.

#. Click **+ New Alert Policy**.

#. In the **Create Alert Policy** window, fill out the following fields:

   - **Entity Type** - VM
   - **Entity (Line 1)** - One VM
   - **Entity (Line 2)** - *Initials*\ **-Linux-ToolsVM**
   - **Metric** - Controller IO Bandwidth
   - **Impact Type** - Performance
   - **Policy Name** - *Initials* - Bully VM
   - **Description** - Optional
   - **Auto Resolve Alerts** - Checked
   - **Enable Policy** - **Unchecked**
   - **Trigger alert if conditions persist for** - 0 Minutes

   - **Behavioral Anomaly**
       - **Every time there is an anomaly, alert** - Checked / Warning

   - **Static Threshold**
       - **Alert Critical if** - Checked / >= 250

   .. figure:: images/xplay_28.png

#. Click **Save**.

   .. note::

     Customers can choose out-of-the-box alert policies (shown below) to detect the bully VM with X-Fit.

Create Custom REST API Action
.............................

#. In **Prism Central** > select :fa:`bars` **> Operations > Actions Gallery**.

#. Select **REST API** action, and then select **Clone** from the **Action** dropdown.

   .. figure:: images/xplay_29.png

#. Fill in the following fields:

   - **Name**  - *Initials* - Quarantine a VM
   - **Description** - Quarantine a VM using Flow API
   - **Method**  - PUT
   - **URL** - https://*<your PC IP>*:9440/api/nutanix/v3/vms/{{trigger[0].source_entity_info.uuid}}
   - **Request Headers** - Content-Type: application/json

   .. figure:: images/xplay_30.png

#. Click **Copy**.

Create Playbook
...............

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

#. Click **Create Playbook**.

#. Select :fa:`bell` **Alert** as Trigger, and click **Select**.

#. Start typing to search for “Bully VM” in **Alert Policy**, and select *Initials* - **Bully VM**.

#. Click **Add Action**, and select the :fa:`terminal` **REST API** action.

   - **Method**  - GET
   - **URL** - http://<IP of *Initial*-Linux-toolsVM>:3000/vm/{{trigger[0].source_entity_info.uuid}}

   .. note::

     There is a known issue in 5.10 where you have to click the “GET” in the drop list once even though “GET” is shown as the default value.

#. Click **Add Action**, and select the :fa:`terminal` *Initials* - **Quarantine a VM** action.

   .. note::

     There is a known issue in 5.10 where the title of this action still shows as “REST API”. In 5.11 GA, you will see the title as you specified earlier.

#. Click **Parameters** and select **Response Body** into the request body field.

   .. figure:: images/xplay_31.png

#. Specify the **Username** and **Password** for **Prism Central**.

#. Click **Add Action**, and select the **Acknowledge Alert** action.

#. Select **Alert** from the parameters.

   - **Target Alert**  - {{trigger[0].alert_entity_info}}

#. Click **Save & Close**, and fill out the following fields:

   - **Name**  - *Initials* - Auto Quarantine A Bully VM
   - **Description** - Optional
   - **Status**  - Enabled

#. Click **Save**.

Cause Bully VM Condition
........................

#. In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > VMs**, and click *Initials*\ **-Linux-ToolsVM**.

#. Click **Categories**, and make sure it is not currently quarantined and associated with any categories.

#. In **Prism Central** > select :fa:`bars` **> Activity > Alerts**, and select **Alert Policy** from **Configure** drop-down menu.

   Select *Initials* - **Bully VM**, and **Enable** the policy.

   Open a console session or SSH into Prism Central, and run the **paintrigger.py** script:

   - **Username** - nutanix
   - **password** - nutanix/4u

   .. code-block:: bash

     python paintrigger.py

#. After 1-2 minutes check *Initials*\ **-Linux-ToolsVM**, you should now see the VM is quarantined.

Cleanup Bully VM Condition
..........................

#. Un-quarantine your *Initials*\ **-Linux-ToolsVM**.

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

#. Click the *Initials* - **Auto Quarantine A Bully VM** playbook, and click the **Disable** button.

#. Click the **Play** tab, you should see that a Play has just completed.

#. If the terminal session is broken (due to the quarantine), log in to *Initial*-**Linux-ToolsVM** to kill the node and stress processes.

(Optional) Endless Possibilities Using APIs
+++++++++++++++++++++++++++++++++++++++++++

This exercise will show how you can easily include 3rd party tools into X-Play. Using `IFTTT <https://ifttt.com>`_ you can easily send a Slack message when an alert is detected. This same functionality could be extended to SMS alerts, ServiceNow, or any other 3rd party tools.

#. Before we set up IFTTT, ensure your *Initial*-**Linux-ToolsVM** has 2GB of memory assigned.

#. Log in to the *Initials*\ **-Linux-ToolsVM** via ssh or Console session.

#. Run ``stress`` again to generate memory pressure:

   .. code-block:: bash

     stress -m 4 --vm-bytes 500M

   .. note::

     It will take roughly 5min for Stress to generate the memory load to cause the alert.

Setup IFTTT
...........

#. Register for a free account at https://ifttt.com/.

#. Log in and search for **Webhooks**.

#. Click on **Services > Webhooks**.

   .. figure:: images/xplay_32.png

#. Click **Connect**.

   .. figure:: images/xplay_33.png

#. Click the **Settings** button at the top right.

   .. figure:: images/xplay_34.png

#. Copy the URL shown in the **Settings** (e.g. https://maker.ifttt.com/use/xxxxxyyyyzzz).

#. Paste that URL into a new browser tab, and go to the page. The page that opens will show your unique Webhook address (e.g. \https://maker.ifttt.com/trigger/{event}/with/key/xxxxxyyyzzz).

   Take note of the address, as this is what we will be targeting in the X-Play REST API action later.

   Now you can create your own applet that will be triggered when it is called from X-Play.

#. In a new browser tab, open https://ifttt.com/my_applets.

#. Click **New Applet**.

   .. figure:: images/xplay_35.png

#. Click **+this**.

   This is where you will set up the Webhook URL that X-Play can trigger.

   .. figure:: images/xplay_36.png

#. Search and click **Webhooks**.

   .. figure:: images/xplay_37.png

#. Click **Receive a web request**.

#. Fill your **event** name. This name will be part of the Webhook URL from earlier in the exercise:

   For example, if the event name is **xplay**, the Webhook URL you will use in X-Play will be something like this:

   \https://maker.ifttt.com/trigger/xplay/with/key/xxxxxyyyzzz

   .. figure:: images/xplay_38.png

#. Click **Create Trigger**.

   You can now create the **+that** to decide what you are going to do in this applet.

   You can use your imagination here. There are over 600 IFTTT services from which you can choose. For example, you can call your cell phone, send you an calendar event, send a text message, change the color of a Philips HUE LED lightbulb, or even open your garage door.

   .. note::

      If you are familiar with Zapier, you can also use that instead of IFTTT. Zapier can connect to over 1000 services, including Salesforce, PagerDuty, and many enterprise applications.

   For this lab we are using its Slack service as an example. You are free and **encouraged** to choose any other service in this step.

   .. note::

     X-Play also includes a native Slack action without requiring 3rd party services such as IFTTT.

#. Click **+that**.

#. Search and click **Slack**.

#. Click **Connect**.

#. When prompted, sign into Slack.

#. Click **Post to channel** and fill in the channel and message.

   You have three values can pass from from X-Play to IFTTT:

   In this example, Value 1 is the Alert name, Value 2 is the VM name, and Value 3 is the Playbook name.

#. Click **Add Ingredient** to specify **Values 1-3**.

#. Fill in the Following:

   - **Which channel** - Direct Messages & @yourSlackHandle
   - **Message** - Nutanix X-FIT just detected an issue of {{Value1}} in {{Value2}} VM. Playbook "{{Value3}}" has increased its memory by 1GB. -- This message was sent by Prism Pro on {{OccurredAt}}.
   - **Title** - Nutanix Prism Pro just fixed an issue for you.

   .. figure:: images/xplay_39.png

#. Click **Create Action > Finish**.

   You now have an IFTTT applet that can be called from X-Play through a generic Webhook!

Create Custom REST API Action
.............................

#. In **Prism Central** > select :fa:`bars` **> Operations > Actions Gallery**.

#. Select **REST API** action, and then select **Clone** from the **Action** dropdown.

#. Fill in the following fields:

   - **Name**  - *Initials* - Slack an X-Play Message by IFTTT
   - **Description** - Using with IFTTT
   - **Method**  - Post
   - **URL** - *Your IFTTT URL*, (e.g. \https://maker.ifttt.com/trigger/xplay/with/key/xxxxxyyyzzz)
   - **Request Body**  - { "value1": "{{trigger[0].alert_entity_info.name}}", "value2": "{{trigger[0].source_entity_info.name}}", "value3": "{{playbook.playbook_name}}" }
   - **Request Headers** - Content-Type: application/json

   .. figure:: images/xplay_40.png

#. Click **Copy**.

Create Playbook
...............

#. In **Prism Central** > select :fa:`bars` **> Operations > Playbooks**.

#. Select *Initials* - **Auto Remove Memory Constraint** created in the earlier lab, and click **Update** from the **Action** drop-down menu.

#. Click :fa:`ellipsis-v` next to the action **Email** and then choose **Add Action Before**.

   .. figure:: images/xplay_41.png

#. Select the :fa:`terminal` *Initials* - **Slack an X-Play Message by IFTTT** action.

#. Click **Save & Close**

#. Toggle to **Enabled**, and click **Save**.

Cause Memory Constraint
.......................

#. Click **Alerts**, Select **Alert Policy** from **Configure** drop-down menu.

#. Select *Initials*-**VM Memory Constrained**, and **Enable** the policy.

#. Open a console session or SSH into Prism Central, and run the **paintrigger.py** script:

   - **Username** - nutanix
   - **password** - nutanix/4u

   .. code-block:: bash

     python paintrigger.py

#. After 2-5 minutes you should receive both an email and a Slack message from Prism.

#. Verify the amount of memory assigned to *Initials*\ **-Linux-ToolsVM** has increased.

Takeaways
+++++++++

What are the key things you should know about **Prism Pro: X-Play**?

- Prism Pro is our solution to make IT OPS smarter and automated. It covers the IT OPS process ranging from intelligent detection to automated remediation.

- X-Fit is our machine learning engine to support smart IT OPS, including forecast, anomaly detection, and inefficiency detection.

- X-Play, the IFTTT for the enterprise, is our engine to enable the automation of daily operations tasks.

- X-Play enables admins to confidently automate their daily tasks within minutes.

Getting Connected
+++++++++++++++++

Have a question about **Prism Pro: X-Play**? Please reach out to the resources below:

+---------------------------------------------------------------------------------+
|  X-Play Product Contacts                                                        |
+================================+================================================+
|  Slack Channel                 |  #prism-pro                                    |
+--------------------------------+------------------------------------------------+
|  Product Manager               |  Harry Yang, harry.yang@nutanix.com            |
+--------------------------------+------------------------------------------------+
|  Product Marketing Manager     |  Mayank Gupta, mayank.gupta@nutanix.com        |
+--------------------------------+------------------------------------------------+
|  Technical Marketing Engineer  |  Brian Suhr, brian.suhr@nutanix.com            |
+--------------------------------+------------------------------------------------+
