# Flic App, Hub and Smart Buttons Reverse Engineering

Understand how the Flic app, the Flic Hub and the Flic buttons work. Update Flic Smart Buttons configuration with or without the official application.

## Take down notice from Shortcut Labs

On July 4, 2020, Shortcut Labs sent me a take down notice for this repository.
```
I kindly ask you to take down your flic-reverse repo at GitHub.
It contains confidential information, API keys, and also includes (encrypted) firmware which you have no rights to redistribute.

We're continually opening up things more and more, but for now we'd like to keep the API private.
For Flic 2 we have released the protocol specification at https://github.com/50ButtonsEach/flic2-documentation/wiki/Flic-2-Protocol-Specification.

If you would like to try out our beta version of our Hub SDK, please let me know. With it you can do a range of things, such as send IR commands, set up TCP/UDP sockets, make internet requests, scan new buttons, listen to button events, configure wifi etc.
```

As a consequence, the main content of the repository is now private. The original table of content is still there for reference.

At that date, the promises of the HUB SDK are still not fullfiled. Shortcut Labs also refused to publish the proto files for the GRPC communication and the API specification arguing that is "a private API you are not supposed to use", "[we] don't want anyone to use the config API currently", that "the hub <-> app protocol will not be documented for the foreseeable future" and concluding that "[we] don't see how [this repo] this can help anyone for a better Flic experience the way we want".

I will let anyone appreciate the advertised promises of the Kickstarter campaign versus the reality.

<!-- https://luciopaiva.com/markdown-toc/ -->
## Table of content

<details>
  <summary>Expand the table of content</summary>

  - [Introduction](#introduction)
  - [Disclaimer](#disclaimer)
  - [Contribute](#contribute)
  - [Security concerns](#security-concerns)
    - [Hardening of the APK](#hardening-of-the-apk)
    - [Default credentials for Flic Hub](#default-credentials-for-flic-hub)
    - [Insecure storage of credentials](#insecure-storage-of-credentials)
    - [Information disclosure and possible flooding through password reset](#information-disclosure-and-possible-flooding-through-password-reset)
    - [Statistics](#statistics)
  - [APK](#apk)
    - [Interesting strings found in the APK](#interesting-strings-found-in-the-apk)
    - [Local storage](#local-storage)
  - [API api.flic.io](#api-apiflicio)
    - [Authentication](#authentication)
    - [/api/v1/users](#apiv1users)
      - [GET /api/v1/users/signup](#get-apiv1userssignup)
      - [POST /api/v1/users/login/basic-auth](#post-apiv1usersloginbasic-auth)
      - [GET /api/v1/users/me](#get-apiv1usersme)
      - [PUT /api/v1/users/me](#put-apiv1usersme)
      - [POST /api/v1/users/connect-hub-user](#post-apiv1usersconnect-hub-user)
      - [POST /api/v1/users/reset-password](#post-apiv1usersreset-password)
    - [/api/v1/buttons](#apiv1buttons)
      - [GET /api/v1/buttons](#get-apiv1buttons)
      - [GET /api/v1/buttons/{button-id}](#get-apiv1buttonsbutton-id)
      - [PUT /api/v1/buttons/{button-id}](#put-apiv1buttonsbutton-id)
      - [POST /api/v1/buttons/{button-id}/unlock](#post-apiv1buttonsbutton-idunlock)
      - [GET /api/v1/buttons/{button-id}/cloud-redirect](#get-apiv1buttonsbutton-idcloud-redirect)
      - [GET /api/v1/buttons/{button-id}/trigger/{trigger-id}](#get-apiv1buttonsbutton-idtriggertrigger-id)
      - [GET /api/v1/buttons/mac-addresses/ranges](#get-apiv1buttonsmac-addressesranges)
      - [GET /api/v1/buttons/tags](#get-apiv1buttonstags)
      - [GET /api/v1/buttons/keys](#get-apiv1buttonskeys)
      - [GET /api/v1/buttons/keys/grab-legacy-button](#get-apiv1buttonskeysgrab-legacy-button)
      - [GET /api/v1/buttons/versions/firmware](#get-apiv1buttonsversionsfirmware)
      - [POST /api/v1/buttons/versions/firmware2](#post-apiv1buttonsversionsfirmware2)
      - [POST /api/v1/buttons/button-icons](#post-apiv1buttonsbutton-icons)
      - [GET /api/v1/buttons/hub/tags](#get-apiv1buttonshubtags)
    - [/api/v1/configs](#apiv1configs)
      - [GET /api/v1/configs](#get-apiv1configs)
      - [PUT /api/v1/configs/{config-id}](#put-apiv1configsconfig-id)
    - [/api/v1/tasks](#apiv1tasks)
      - [GET /api/v1/tasks](#get-apiv1tasks)
      - [PUT /api/v1/tasks/{task-id}](#put-apiv1taskstask-id)
    - [/api/v1/sms](#apiv1sms)
      - [GET /api/v1/sms](#get-apiv1sms)
      - [POST /api/v1/users/devices](#post-apiv1usersdevices)
  - [Statistics endpoint statistics.flic.io](#statistics-endpoint-statisticsflicio)
    - [/api/v1/statistics](#apiv1statistics)
      - [POST /api/v1/statistics](#post-apiv1statistics)
  - [Flic Hub](#flic-hub)
    - [Bluetooth](#bluetooth)
      - [GATT](#gatt)
      - [btsnoop_hci.log](#btsnoophcilog)
      - [Enabling debug logs](#enabling-debug-logs)
      - [Authentication sequence](#authentication-sequence)
    - [HTTP Traffic between Flic Hub and the API](#http-traffic-between-flic-hub-and-the-api)
    - [Flic Hub SDK (myhub.flic.io)](#flic-hub-sdk-myhubflicio)
    - [Reverse engineering of firmware](#reverse-engineering-of-firmware)
  - [Integration with third-party systems](#integration-with-third-party-systems)
  - [Home Assistant](#home-assistant)
    - [REST API](#rest-api)
    - [Webhook](#webhook)
  - [FAQ](#faq)
    - [When using HTTP Request integration, who makes the call? The phone / HUB directly or Flic?](#when-using-http-request-integration-who-makes-the-call-the-phone--hub-directly-or-flic)
    - [Is it possible to use the API to get or set the config of the Flic Hub](#is-it-possible-to-use-the-api-to-get-or-set-the-config-of-the-flic-hub)
    - [Is it possible to call an API to send custom IR commands through the IR module?](#is-it-possible-to-call-an-api-to-send-custom-ir-commands-through-the-ir-module)
    - [Is there a request sent to the API each time a button is clicked?](#is-there-a-request-sent-to-the-api-each-time-a-button-is-clicked)
    - [How can I integrate my Flic buttons with other systems?](#how-can-i-integrate-my-flic-buttons-with-other-systems)
  - [Credits](#credits)
  - [Legal](#legal)
</details>

## Introduction

The guys from Shortcut Labs AB made great products with their Flic Smart Buttons but sadly, there were promises of a SDK to interact with the buttons and the hub but nothing was delivered yet. The team focused on the Android and iOS SDKs for other apps to be able to trigger an action on a button click.

I was not interested by this part. I wanted to see what can be done by fiddling with the application. I wanted a way to get events from buttons paired with my phone or the Flic Hub. I discovered that this was not so easy. The Flic Hub has not any REST API exposed, in fact, no port is open on the Hub. Still, I discovered how the buttons' config can be retrieved and updated when paired on the phone. I also got a first overview about how it's done with Bluetooth, however, I did not fully reversed the link encryption part at this stage.

This review was only done on the Android version of the application. Version 3.7.8.

## Disclaimer
The analysis of the application was exclusively done through traffic sniffing (HTTP and Bluetooth) and static analysis of the Android application. The goal was to find ways to interoperate with the Flic buttons, not to find security vulnerabilities in software or hardware. Potential weaknesses are reported since they can help to understand how the application works.

Credentials, security tokens, serials were updated with fake ones in this documentation.

## Security concerns

### Default credentials for Flic Hub
To ease the initial setup, it is not required to enter the "factory password" of the hub that is printed on its back. It's the same when a factory reset is done (rollback to firmware 1.0). The default password of the hub is **XXX** (the three letters are redacted). However, this default password is only used for the initial setup and it is immediately replaced by a randomly generated one which is hashed and stored hashed on the phone.

### Insecure storage of credentials
Flic Hub password, when **user** resets it or when manually set by the user, is stored in clear text in the SQLite database. It is not the case for the initial pairing or when the hub is **factory** reset. In this case, a random password is generated **and** stored **hashed** (SHA-512) in the SQLite database.

### Statistics
The app sends information about the phone and the executed actions to a dedicated statistics endpoint. Looks like no confidential is leaked but still... I did not find a way to disable this in the app.

Since actions may be related to security related stuff (e.g. alarms), and that exact date and time of execution are sent, it could lead to privacy and security concerns.

## Integration with third-party systems
## Home Assistant
Since there is no way to use the Hub in a very generic way to request the latest events or send a request without configuring the buttons manually. I tried to keep it rather generic to integrate them with Home Assistant.

I used the "Internet Request" feature which is available for the buttons paired with the phone or the hub. 

I send a POST request to the HA endpoint (you can do it with or without authentication, cf. below) with the following information depending on the button id and the type of click:

```json
{"id": "flic1", "event_type": "click"}
{"id": "flic1", "event_type": "double_click"}
{"id": "flic1", "event_type": "hold"}
```

It is also possible to retrieve the `button-serial-number` and the `button-name` (display name if the Flic app) directly in the headers, as described in the FAQ.

Example of automation:
```yaml
- alias: Salle de bains - Flic - Alexa - Joue France Info
  description: ''
  trigger:
  - event_data:
      event_type: click
      id: flic1
    event_type: flic
    platform: event
  condition:
  - condition: not
    conditions:
    - condition: device
      device_id: 3dc4df62da4c4813372b16e3d8aa552
      domain: media_player
      entity_id: media_player.morgane_echo
      type: is_playing
  action:
  - data:
      entity_id: media_player.morgane_echo
      media_content_id: FranceInfo
      media_content_type: TUNEIN
    service: media_player.play_media
```

### REST API
Authenticated, using a Long-Lived Access Token

Fire an event directly from the API and listen for the event as a trigger in the automation:

https://developers.home-assistant.io/docs/api/rest/#post-apieventsevent_type

### Webhook
Non-authenticated, with a webhook trigger:
https://www.home-assistant.io/docs/automation/trigger/#webhook-trigger.

Keep it simple with a single webhook. However, since it is not possible to use the same webhook for multiple automations, the workaround is to use a webhook that will "refire" an event for the other automations:
```yaml
- alias: Salle de bains - Flic 1 - Event
  description: ''
  trigger:
  - platform: webhook
    webhook_id: flic
  condition: []
  action:
  - event: flic
    event_data_template:
      id: '{{ trigger.json.id }}'
      event_type: '{{ trigger.json.event_type }}'
```

## FAQ

### When using HTTP Request integration, who makes the call? The phone / HUB directly or Flic?
It's the phone directly, with a few additional headers:

```http
GET / HTTP/1.1
Accept-Encoding: gzip, deflate
User-Agent: Google-HTTP-Java-Client/1.23.0 (gzip)
button-serial-number: BA48-131337
button-battery-level: 100
button-name: Flic 3
timestamp: 2020-06-30T09:01:04Z
Host: xxxxx.x.pipedream.net
Connection: close
```

I did not receive other requests from other IPs on the test endpoint.

### Is it possible to use the API to get or set the config of the Flic Hub
Sadly no.

* https://community.flic.io/topic/17660/configuring-flic-hub-programmatically

* https://community.flic.io/topic/17253/running-custom-apps-on-the-flic-hub/

### Is it possible to call an API to send custom IR commands through the IR module?
I don't think so. It is not possible to use the API for the hub

### Is there a request sent to the API each time a button is clicked?
Not really but since the configuration is stored online and the requests are sent to the statistics endpoint, it is possible for Shortcut Labs to know exactly when an action was triggered, but not in real time.

### How can I integrate my Flic buttons with other systems?
In the end, the easiest method is to use the "Internet Request" integration. 

## Credits
@righettod for his great help!

## Legal

This research and documentation are in no way affiliated with, authorized, maintained, sponsored or endorsed by Shortcut Labs AB or any of its affiliates or subsidiaries. This is an independent and unofficial documentation. Use at your own risk.
