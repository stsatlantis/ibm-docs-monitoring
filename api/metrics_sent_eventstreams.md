---

copyright:
  years:  2018, 2021
lastupdated: "2021-03-28"

keywords: IBM Cloud, monitoring, query, api

subcollection: monitoring

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}
{:important: .important}
{:note: .note}
{:external: target="_blank" .external}

# Extracting notifications from {{site.data.keyword.mon_short}} to IBM Cloud Event Streams
{: #metrics_sent_eventstreams}

You can extract notifications from {{site.data.keyword.mon_short}} to IBM Cloud Event Streams by using the Event Streams REST API.
{:shortdesc}

Complete the following steps:

## Step 1. Provision an Event Streams instance and provision a topic
{: #metrics_sent_eventstreams_step1}

Complete the following steps:

1. Provision an Event Streams instance with an Enterprise Plan. Ensure that the Event Streams instance uses private endpoints.
2. Create a topic in the Event Streams instance, for example, `my-sample-topic`.

## Step 2. Obtain the Event Streams API connection details
{: #metrics_sent_eventstreams_step2}

Obtain the following Event Streams API connection details that you must use when you configure the notification channel:
- api_key
- kafka_http_url

Choose 1 of the following methods:

### Get credentials using the IBM Cloud console
{: #metrics_sent_eventstreams_step2-1}

1. Locate your {{site.data.keyword.messagehub}} service on the dashboard.
2. Click your service tile.
3. Click **Service Credentials**.
4. Click **New Credential**. 
5. Complete the details for your new credential like a name and role and click **Add**. A new credential appears in the credentials list.
6. Click this credential by using **View Credentials** to reveal the details in JSON format.



### Get credentials using the IBM Cloud CLI
{: #metrics_sent_eventstreams_step2-2}

Complete the following steps:

1. Locate your service instance.

    ```
    ibmcloud resource service-instances
    ```
    {: codeblock}

2. Create a service key via the command line.

    ```
    ibmcloud resource service-key-create KEY_NAME Writer --instance-name INSTANCE_NAME --service-endpoint private
    ```
    {: codeblock}

    Where `KEY_NAME` is the name of the key, and `INSTANCE_NAME` is the name of the Event Streams instance.

    The output of this command includes the details that are needed to make a REST call to send notifications to the Event Streams instance. 



## Step 3. Create a notification channel
{: #metrics_sent_eventstreams_step3}

From the {{site.data.keyword.mon_short}} UI, setup a Webhook notification channel.

Complete the following steps to add a notification channel:

1. Launch the web UI. For more information on how to launch the Web UI, see [Navigating to the Web UI](/docs/monitoring?topic=monitoring-launch#launch). 
    
2. From the *Selector* button in the navigation bar, choose **Settings**.

3. Select **Notification Channels**.

4. Click **MY CHANNELS** ![add icon](images/add.png). Then, select a channel.

    **Note:** The first time you configure a notification channel, click on any of the channels.

5. Configure a notification channel:

    * Enter the name of the channel.

    * Enable the *Notify when OK* field to receive a notification when the alert condition is no longer triggered.

    * Enable the *Notify when Resolved* condition to receive a notification when the alert is manually resolved by a user.

    Enter the webhook *URL*. The format is the following: `<kafka_http_url>/topics/<topic>/records?apikey=<api_key>`

    Where <topic> is the name of your topic.

    For example, the URL field can be `https://y545tf67898h.svc01.eu-gb.eventstreams.cloud.ibm.com/topics/my-sample-topic/records?apikey=27836238765487558695`
    
    When an alert is triggered, the notification is sent as a POST in JSON format to your webhook endpoint. 

    * Optionally, and for integrations that allow a test, enable the *Test notification* condition to receive a test notification. If you do not receive a test notification in 10 minutes, review your channel configuration. 

6. Click **CREATE CHANNEL**. 


Currently additionalHeaders cannot be created on the original create. However, you can patch the notification channel. For example, you can use cURL to modify a notification channel: 

1. [Get the notification channel ID](/docs/monitoring?topic=monitoring-notifications_api#notifications_api_get_all).

2. Run the following command:

    ```
    curl -X PUT ’https://us-east.monitoring.cloud.ibm.com/api/notificationChannels/<CHANNEL_ID>' \
    --header ‘Content-Type: application/json’ 
    --header ‘Authorization: Bearer <token>’ -d @/tmp/notification.json
    ```
    {: codeblock}



## Next. Setup alerts
{: #metrics_sent_eventstreams_step3}

Next setup alerts in {{site.data.keyword.mon_short}} by using this notification channel. 




