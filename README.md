# Home Assistant and Google Assistant SDK Custom integration Example

This is an example of using Google Assistant's SDK to take a device only available in Google Home and add it to Home Assistant.

Some devices are only available as cloud-integrated options and lack local control. If you still want to make these devices available in Home Assistant, it's possible to create local objects and then integrate them using Google Assistant. This can mimic the appearance of these devices being locally available in Home Assistant.

## Prerequisites

There are two key steps that you must complete first - integrating Home Assistant with Google Cloud, and integrating with Google Assistant.

### Integrate Home Assistant with Google Cloud

[I followed the steps that Home Assistant provides to take advantage of their Nest integration](https://www.home-assistant.io/integrations/nest/#configuration). The same configuration of Google Cloud, OAuth credentials, and a Device Access Project will enable Home Assistant to talk to Google and leverage the Google Assistant SDK.

### Custom Integration with Google Assistant

Home Assistant comes with a [Google Assistant SDK Integration](https://www.home-assistant.io/integrations/google_assistant_sdk), but this is mostly for mimic speaking to a Google Home device and sending text commands to Google Assistant. As of Februrary 2024 this integration is unable to parse many responses that Google provides.

Using the [Google Assistant SDK Custom Integration for Home Assistant](https://github.com/tronikos/google_assistant_sdk_custom) instead bridges this gap. This alternative enables HTML output in Google's responses and then parses that output, enabling calls against the Google Assistant SDK to read those responses. You will need [HACS](https://hacs.xyz/) to setup this custom integration.

## What do the components in this repo do?

- **Scripts**: These use the Google Assistant SDK Custom Integration to send text commands to Google Assistant. They either instruct Google Assistant to take action (Turn the device on) or to retrieve a status (Is the device on?). Scripts store the resulting status in a Helper.
- **Helpers**: These store the results of the query against Google Assistant SDK.
- **Devices**: This is an example device built as a [Template Switch](https://www.home-assistant.io/integrations/switch.template/). The device uses the text in the Helper to determine its state, and user input to adjust the switch's state triggers Scripts to send updates to Google Assistant.
- **Automations**: The real devices still exist external to Home Assistant, so this automation periodically polls Google Assistant SDK to make sure the status is kept up to date. Adjust the time interval to meet your needs.

## How does it work?

### Triggering from Home Assistant

1. User interacts with the Device in Home Assistant.
1. The Device template calls the Device On script or the Device Off script.
1. The Device On/Off scripts send a text command to Google Assistant SDK, instructing it to turn the external Device on or off.
1. The scripts then call the Refresh Status script to get an update from Google Assistant SDK and make sure the state has changed as intended.

### Triggering external to Home Assistant

1. The Refresh Status automation calls the Refresh Status script.
1. The Refresh Status script submits a question to Google Assistant SDK, and stores the response within a Helper.
1. The Device looks at the value of the Helper and adjusts its value accordingly.

## Limits and Quotas

Google imposes quotes on the number of times you can make requests against their APIs. If you follow the guides in this document, the limit will be 500 calls per day. If you configure multiple devices to update status as part of the Automation or refresh too frequently, you may hit the limit of your Google Cloud Project. Adjust your request interval such that you don't hit the limit and exhaust the resource availability.

You can check your quote usage by signing into Google Cloud and Navigating to [GCP's page on the Google Assistant API](https://console.cloud.google.com/apis/api/embeddedassistant.googleapis.com) for your project and selecting "Quotas & System Limits."

## Possible Improvements

- Find a way to get the status of multiple devices from Google Assistant in a single request.
- Use the resulting feedback from a "turn on" or "turn off" request to get feedback instead of submitting another request to update status.
  - Update the Device On/Off scripts to write the Google Assistant response to the Helper.
  - Adjust the `value_template` of the Device to accept several forms of response from Google Assistant.
- Replace use of the Google Assistant API with that of the Cloud Pub/Sub API.
