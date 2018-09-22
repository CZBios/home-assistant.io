---
layout: page
title: "IFTTT"
description: "Instructions on how to setup IFTTT within Home Assistant."
date: 2015-09-07 18:00
sidebar: true
comments: false
sharing: true
footer: true
logo: ifttt.png
ha_category: Automation
featured: true
ha_iot_class: "Cloud Push"
---

[IFTTT](https://ifttt.com) is a web service that allows users to create chains of simple conditional statements, so called "Applets". With the IFTTT component you can trigger applets through the **"Webhooks"** service (which was previously the **"Maker"** channel). See the [announcement blog post](/blog/2015/09/13/home-assistant-meets-ifttt/) for examples how to use it.

```yaml
# Example configuration.yaml entry
ifttt:
  key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

`key` is your API key which can be obtained by viewing the **Settings** of the [Webhooks applet](https://ifttt.com/services/maker_webhooks/settings). It's the last part of the URL (e.g., https://maker.ifttt.com/use/MYAPIKEY) you will find under **My Applets** > **Webhooks** > **Settings**.


<p class='img'>
<img src='/images/components/ifttt/finding_key.png' />
Property screen of the Maker Channel
</p>

Once you have added your key to your `configuration.yaml` file, restart your Home Assistant server. This will load up the IFTTT component and make a service available to trigger events in IFTTT.

<p class='note'>
After restarting the server, be sure to watch the console for any logging errors that show up in red, white or yellow.
</p>

### {% linkable_title Testing your trigger %}

You can use the **Developer tools** to test your [Webhooks](https://ifttt.com/maker_webhooks) trigger. To do this, open the Home Assistant frontend, open the sidebar, click on the first icon in the developer tools. This should get you to the **Call Service** screen. Fill in the following values:

Field | Value
----- | -----
domain | `ifttt`
service | `trigger`
Service Data | `{"event": "EventName", "value1": "Hello World"}`

<p class='img'>
<img src='/images/components/ifttt/testing_service.png' />
When your screen looks like this, click the 'call service' button.
</p>

### {% linkable_title Setting up a recipe %}

Press the *New applet* button and search for *Webhooks*.

<p class='img'>
<img src='/images/components/ifttt/setup_service.png' />
Choose "Webhooks" as service.
</p>

<p class='img'>
<img src='/images/components/ifttt/setup_trigger.png' />
You need to setup a unique trigger for each event you sent to IFTTT.
</p>

{% raw %}
```yaml
# Example configuration.yaml Automation entry
automation:
  alias: Startup Notification
  trigger:
    platform: homeassistant
    event: start
  action:
    service: ifttt.trigger
    data: {"event":"TestHA_Trigger", "value1":"Hello World!"}
```
{% endraw %}

IFTTT can also be used in scripts and with `data_template`.  Here is the above automation broken into an automation and script using variables and data_templates.

{% raw %}
```yaml
# Example configuration.yaml Automation entry
automation:
  alias: Startup Notification
  trigger:
    platform: homeassistant
    event: start
  action:
    service: script.ifttt_notify
    data_template:
      value1: 'HA Status:'
      value2: "{{ trigger.event.data.entity_id.split('_')[1] }} is "
      value3: "{{ trigger.event.data.to_state.state }}"
```
{% endraw %}

{% raw %}
```yaml
#Example Script to send TestHA_Trigger to IFTTT but with some other data (homeassistant UP).
ifttt_notify:
  sequence:
    - service: ifttt.trigger
      data_template: {"event":"TestHA_Trigger", "value1":"{{ value1 }}", "value2":"{{ value2 }}", "value3":"{{ value3 }}"}
```
{% endraw %}

### {% linkable_title Sending events from IFTTT to Home Assistant %}

To be able to receive events from IFTTT, your Home Assistant instance needs to be accessible from the web. This can be achieved by forwarding port 8123 from your router to the device running Home Assistant. If your ISP is giving you a new IP address from time to time, consider using [DuckDNS](https://duckdns.org).

In the URL field, you can then put an [API URL](/developers/rest_api/). You probably want to use a POST action, so select `POST` as method. After your request line, you need to add your Home Assistant password, which you defined in the [http section of your config](/getting-started/basic/#password-protecting-the-web-interface), in the form of `?api_password=YOUR_PASSWORD`. For the message body, refer to the API page linked above.

<p class='img'>
<img src='/images/components/ifttt/IFTTT_to_HA.png' />
</p>

### {% linkable_title Additional Channel Examples %}

Additional examples of using IFTTT channels can be found below.

Channel | Description
----- | -----
[Manything](/components/ifttt.manything/) | Automates turning recording ON and OFF based on Home Assistant AWAY and HOME values.

### {% linkable_title Sending events from IFTTT to Home Assistant - first example %}
If you want to make your first example you could take following steps. Dont forget you have to have public IP. Maker channel service on IFTTT is no more, no its called Webhook. 

We will do simple first test, when we will manually call IFTTT applet with an even, and than IFTTT will call back in to the specific script, which we will create. 

### {% linkable_title Create first script %}
We will use a Home assistant UI. Go to the sidebar and choose Settings -> Scripts. Click on bottom left icon and create **New script**. 
Name of the script is up to you. It doesnt matter in this case. 

In **Sequence** part choose action:call service, service:system_log.write. 
In the data field paste this: 
{% raw %}
{
  "message": "test ifttt IN"
}
{% endraw %}
Hit Save. 
Now go to Configuration and locate file **scripts.yaml**. You should see your first script. 
{% raw %}
'<SCRIPT ID>':
  alias: myscriptname
  sequence:
  - data:
      message: test ifttt IN
    service: system_log.write
{% endraw %}

### {% linkable_title Create IFTTT applet with webhooks %}
If **THIS** choose Webhooks and choose receive request. In the field "Event name" write "test in". This will be your event name, which you will call and thus initiate and action from HA to IFTTT and back to HA. 

than **THAT** choose Webhooks again. Now you need to put your URL. Use your own values. 
{% raw %}
   http://<IP ADDRESS>:8123/api/services/script/<SCRIPT ID>?api_password=<PASS>
{% endraw %}
Choose method: POST, type: application/json and you can leave body empty. 
Finish the applet. 
  
### {% linkable_title Test it %}  
Now go back to your HA and click in sidebar on service. From dropdown menu choose service "ifttt:trigger" and put following json data
{% raw %}
   {"event": "test in"}
{% endraw %}
Now hit the button "Call service". 

If you did everything right you should be able in few seconds see in your system log message you configured in the script before. In this case "test ifttt IN". 
