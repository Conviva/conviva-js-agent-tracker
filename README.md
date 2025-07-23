# Conviva JavaScript Agent SDK

[![npm version](https://img.shields.io/npm/v/@convivainc/conviva-js-agent-tracker.svg)](https://www.npmjs.com/package/@convivainc/conviva-js-agent-tracker)

Use Conviva JavaScript Agent SDK to auto-collect events and track application-specific events and state changes.

**Table of Contents**

- [Quick Start](#quick-start)
- [More Features](#more-features)
- [Auto-collected Events](#auto-collected-events)
- [FAQ](#faq)

## Quick Start

### 1. Installation
<!--self-serve[NPM/Yarn]-->

- Install The Conviva JavaScript Agent SDK using either **npm** or **yarn**:
<!-- :::code-tabs[NPM,Yarn] -->

```NPM
npm install @convivainc/conviva-js-agent-tracker
```

```Yarn
yarn add @convivainc/conviva-js-agent-tracker
```
<!-- ::: -->

**Note**: For script-based integrations, refer [Conviva JS Script ECO SDK](https://github.com/Conviva/conviva-js-script-appanalytics) for guidelines.
 <!--eof-self-serve--> 

### 2. Initialization

- Import the required packages into your project:

```js
import {
	convivaAppTracker,
	setUserId,
	trackPageView,
} from '@convivainc/conviva-js-agent-tracker';
```

- Initialize The Conviva JavaScript Agent SDK:

**Note**: It is recommended to initialize the tracker as early **as possible** during the DOM load sequence, such as in `App.js`.

```js
convivaAppTracker({
	appId: 'YOUR_APP_NAME',
	convivaCustomerKey: 'YOUR_CUSTOMER_KEY',
	appVersion: '1.1.0',
});
```

**YOUR_APP_NAME** - A string value that uniquely identifies your app across platforms. For example: `"WEB App"`, `"LGTV App"`.

**YOUR_CUSTOMER_KEY** - A string to identify a specific customer account. Use different keys for dev and prod. Find them in [Pulse](https://pulse.conviva.com/app/profile/applications) under My Profile (_Conviva login required_).

**appVersion** - Set app version in string format.

### 3. Set the User ID

User ID is a unique string identifier to distinguish individual viewers. If using [Conviva Video Sensor](https://github.com/Conviva/conviva-js-coresdk), match it with the **Viewer ID**.
Note: Set the User ID as soon as Conviva sensor is initialized so that none of the events are missed from User's timeline.

```js
setUserId('replace_me_by_the_userId');
```

### 4. Report Page View

trackPageView() should be called after the page is considered "loaded" by your application—that is, when content is rendered and ready for user interaction. Avoid calling it too early (e.g., before the main content or layout is visible), as this may result in incomplete timing data.

What defines a "page change"?
This depends on your app type:

For MPAs: Every full page reload is a new page.
For SPAs: A new page is usually identified by URL path or route changes.

Developers should trigger pageview() based on what they define as a meaningful navigation or view transition in their application.

By default, when `trackPageView()` is called, the _Page Title_ is set using `document.title`. However, you can override this by passing a custom title in the `trackPageView()` API:

```js
// Uses document.title as the Page Title
trackPageView();

// Pass a custom Page Title
trackPageView({ title: 'Custom Page Title' });
```
**Note**: The Web (JS/React) SDK does not collect page views if the trackPageView() API is not explicitly called during a navigation event. As a result, corresponding metrics (such as Page Views, Avg Perceived Page Load Time, Avg Largest Contentful Paint Time) will be missing from the Pulse dashboard. Conviva does not support auto-collection of missing page views.

After steps 1–4, verify [auto-collected events](#auto-collected-events) in the [validation dashboard](https://pulse.conviva.com/app/appmanager/ecoIntegration/validation) . (_Conviva login required_)

### 5. Cleanup API

The `cleanup()` API is used to completely remove all Conviva tracker listeners, timers, and resources from the current page. This is useful for single-page applications (SPAs) or when you need to fully reset or remove the tracker instance (for example, during logout, navigation, or teardown).

**What does cleanup do?**
- Removes all event listeners (network, click, visibility, etc.)
- Clears all timers and intervals used by the tracker
- Disables error and click tracking
- Cleans up network request tracking
- Resets internal tracker state and removes the tracker instance

**Usage Example:**

```js
import { cleanup } from '@convivainc/conviva-js-agent-tracker';

// Remove all Conviva tracker listeners and resources
cleanup();
```

> **Note:**
> The `cleanup` API is not supported in older browsers (Chrome < 66, Mozilla < 57, Safari < 12.1).

## More Features

<details>
<!--self-serve-custom-event-->
<summary><b>Track Custom Event</b></summary>
    
Use the **trackCustomEvent()** API to track all kinds of events. This API provides 2 fields to describe the tracked events:

**name** - Name of the custom event

**data** - Any type of data in JSON format.

```js
import { trackCustomEvent } from '@convivainc/conviva-js-agent-tracker';

let customData = {
	identifier1: 'test',
	identifier2: 1,
	identifier3: true,
};

trackCustomEvent({
	name: 'Custom Event Name',
	data: customData,
});
```
<!--eof-self-serve-custom-event--> 
</details>

<details>
<!--self-serve-custom-tags-->
<summary><b>Set Custom Tags</b></summary>

Custom Tags are global tags applied to all events and persist throughout the application lifespan, or until they are removed.

**Set the custom tags:**

```js
import { setCustomTags } from '@convivainc/conviva-js-agent-tracker';

let customTagsData = { tagKey1: 'tagValue1', tagKey2: 1, tagKey3: true };
setCustomTags(customTagsData);
```

**Clear previously set custom tags:**

```js
import { unsetCustomTags } from '@convivainc/conviva-js-agent-tracker';

// Remove custom tags tagKey2 & tagKey3
let customTagsData = ['tagKey2', 'tagKey3'];
unsetCustomTags(customTagsData);
```
<!--eof-self-serve-custom-tags--> 
</details>

<details>
    <summary><b>Error Reporting</b></summary>
    
Uncaught exceptions and unhandled rejections are automatically collected and enabled by default. To report caught exceptions or other errors, use the following API:

```js
import { trackError } from '@convivainc/conviva-js-agent-tracker';

try {
	//...
} catch (error) {
	trackError({
		message: 'Cannot get user object',
		filename: 'shop.js',
		error: error, // Passing the caught error object.
	});
}
```

</details>

<details>
<summary><b>Client ID Synchronization</b></summary>

When using multiple Conviva JavaScript Agent SDK instances across different environments (e.g., subdomains of the same root domain), the Client ID will be shared automatically using **local storage** & **cookies** 

To ensure consistency, the SDK provides the following advanced APIs for manual synchronization. These APIs are intended for developers who require fine-grained control over Client ID management across multiple instances.
Use Cases:

- Synchronizing Client ID between a mobile app and WebView.

**Note**: The Conviva JavaScript Agent SDK utilizes **local storage** & **cookies** to cache some data.

- `getClientId()` – Retrieves the current Client ID
- `setClientId(clientId)` – Sets a specific Client ID

**Retrieve the Client ID**

```js
import {
	convivaAppTracker,
	getClientId,
} from '@convivainc/conviva-js-agent-tracker';

convivaAppTracker({
	appId: 'YOUR_APP_NAME_AS_STRING',
	convivaCustomerKey: 'CONVIVA_ACCOUNT_CUSTOMER_KEY',
	appVersion: '1.1.0',
});

// Always call getClientId() after initializing convivaAppTracker()
clientId = getClientId();
```

**Set the Client ID**

```js
import {
	convivaAppTracker,
	getClientId,
} from '@convivainc/conviva-js-agent-tracker';

// Always call setClientId() before initializing convivaAppTracker() to set a specific clientId
setClientId(clientId);

convivaAppTracker({
	appId: 'YOUR_APP_NAME_AS_STRING',
	convivaCustomerKey: 'CONVIVA_ACCOUNT_CUSTOMER_KEY',
	appVersion: '1.1.0',
});
```

</details>

<details>
<summary><b>Meta Tags Collection </b></summary>

This feature enables tracking of meta tags from the `<head>` section of an HTML page based on the provided configuration.

Example meta tags in an HTML Page:

```js
<html>
    <head>
        <meta name="keywords" content="HTML, CSS, JavaScript">
        <meta name="description" content="Free Web tutorials for HTML and CSS">
        <meta name="author" content="John Doe">
        <meta http-equiv="refresh" content="30">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta property="site_name" content="goole.com">
        <meta property="title" content="Sample app">
        <meta property="description" content="TV series content">
        <meta property="locale" content="es_ES">
        <meta property="type" content="video">
    </head>
</html>
```

**Configure Meta Tags Tracking**

To collect meta tag data, you need to define the `metaTagsTracking` configuration during SDK initialization.

Example Configuration:

```js
convivaAppTracker({
	appId: 'YOUR_APP_NAME_AS_STRING',
	convivaCustomerKey: 'CONVIVA_ACCOUNT_CUSTOMER_KEY',
	appVersion: '1.1.0',
	configs: {
		metaTagsTracking: {
			tags: [
				{
					key: 'name', // Target meta tags with "name" attributes
					value: 'content', // Extract their "content" values
				},
				{
					key: 'property', // Target meta tags with "property" attributes
					value: 'content', // Extract their "content" values
					condition: ['title', 'locale'], // Optional: Filter by specific property values
				},
			],
		},
	},
});
```

</details>

<details>
<summary><b>Set Device Metadata</b></summary>

`deviceMetadata` is an object containing key-value pairs for predefined values, such as DeviceType and DeviceCategory, as well as additional properties like DeviceBrand, DeviceManufacturer, and DeviceModel.

Conviva automatically collects deviceMetadata for Web apps and mobile browsers. However, for devices like set-top boxes, smart TVs, gaming consoles, and others, you will need to manually set the `deviceMetadata`.

**Example of setting deviceMetadata:**

```js
import {
	convivaAppTracker,
	ConvivaDeviceMetadata,
} from '@convivainc/conviva-js-agent-tracker';

const deviceMetadata: ConvivaDeviceMetadata = {
	DeviceBrand: 'Samsung',
	DeviceManufacturer: 'Samsung',
	DeviceModel: 'UTU7000',
	DeviceType: 'SmartTV',
	OperatingSystemName: 'Tizen',
	OperatingSystemVersion: '8.0',
	DeviceCategory: 'SAMSUNGTV',
	FrameworkName: 'Angular',
	FrameworkVersion: '8.0.0',
};

convivaAppTracker({
	appId: 'YOUR_APP_NAME_AS_STRING',
	convivaCustomerKey: 'CONVIVA_ACCOUNT_CUSTOMER_KEY',
	appVersion: '1.1.0',
	deviceMetadata: deviceMetadata,
});
```

<details>
    <summary><b>The table of predefined metadata keys for deviceMetadata</b></summary>


| Key                    | Type                                | Description                                                                                                              | Example Values                                                                          |
| ---------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- | --- |
| DeviceBrand            | string                              | Brand of the device                                                                                                      | `"Comcast"`, `"LG"`, `"Google"`, `"Vizio"`                                              |
| DeviceManufacturer     | string                              | Manufacturer of the device                                                                                               | `"Sony"`, `"Comcast"`, `"Google"`, `"Microsoft"`                                        |
| DeviceModel            | string                              | Model of the device                                                                                                      | `"Comcast Flex"`, `"UTU7000_KA"`, `"Xbox One"`                                          |
| DeviceType             | Prescribed values of DeviceType     | Type of the device. Only allows the DeviceType values and discards any other string values                               | DESKTOP, Console, Mobile (see [table below](#devicecategory-pre-defined-string-values)) |
| DeviceVersion          | string                              | Device firmware version                                                                                                  | `"10"`, `"9"`                                                                           |
| OperatingSystemName    | string                              | Name of the operating system used by the device, in uppercase                                                            | `"Tizen"`, `"webOS"`, `"Vizio`", `"Linux`", `"Xbox OS"`, `"Chrome OS"`                  |
| OperatingSystemVersion | string                              | Version of the operating system used by the device                                                                       | `"10.10.1"`, `"8.1"`, `"T-INFOLINK2012-1012"`, `"1.56.500000"`                          |
| DeviceCategory         | Prescribed values of DeviceCategory | Device category to which the used device belongs. Only allows DeviceCategory values and discards any other string values | WEB, AND, PS (see [table below](#devicetype-pre-defined-string-values))                 |
| FrameworkName          | string                              | Application framework name                                                                                               | `"React TV"`, `"LightningJS"`, `"Angular"`                                              |
| FrameworkVersion       | string                              | Application framework version                                                                                            | `"1.2.3"`                                                                               |     |

#### DeviceCategory Pre-defined String Values:

| Value     | Description                                                                                                                                                                                             |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AND       | The device is an Android device like Samsung Galaxy, Amazon Fire TV, Android TV, or Android Tablet.                                                                                                     |
| APL       | The device is an Apple device like iPhone or Apple TV.                                                                                                                                                  |
| CHR       | The device is a Google Chromecast STB or Android TV with built-in Chromecast.                                                                                                                           |
| DSKAPP    | The device is a desktop computer (including notebooks) where video is played in an installed app, as opposed to a browser.                                                                              |
| SIMULATOR | The device is a simulated video session used for testing.                                                                                                                                               |
| KAIOS     | The device is a phone or other device based on KaiOS OS, such as the Lyf Jio F30C.                                                                                                                      |
| LGTV      | The device is an LG smart TV, including NetCast and webOS.                                                                                                                                              |
| LNX       | This mostly covers various Set-Top Boxes and Smart TVs that use custom Linux-based SDKs.                                                                                                                |
| NINTENDO  | The device is a Nintendo games console, including Wii and Switch.                                                                                                                                       |
| PS        | The device is a PlayStation console, including PS3 and PS4.                                                                                                                                             |
| RK        | The device is a Roku device.                                                                                                                                                                            |
| SAMSUNGTV | The device is a Samsung Smart TV, including Orsay and Tizen.                                                                                                                                            |
| VIDAA     | Vidaa-based devices, using an operating system developed by Hisense.                                                                                                                                    |
| VIZIOTV   | Category for native app integrations on Vizio TVs using the SmartCast platform (from 2016 onwards).                                                                                                     |
| WEB       | The device can be any device with an in-browser HTML5-based player. Video is played in the browser using HTML5 technology, in browsers like Chrome, Edge, Firefox, Internet Explorer, Opera, or Safari. |
| WIN       | The device is a Windows OS-based handheld device, like a Windows Phone or Windows Tablet.                                                                                                               |
| XB        | The device is an Xbox console, including Xbox 360 and Xbox One.                                                                                                                                         |

#### DeviceType Pre-defined String Values:

| Value   | Description                                  |
| ------- | -------------------------------------------- |
| DESKTOP | The device is a desktop or laptop computer.  |
| Console | The device is a gaming console.              |
| Settop  | The device is a set-top box.                 |
| Mobile  | The device is a mobile phone.                |
| Tablet  | The device is a tablet.                      |
| SmartTV | The device is a smart TV.                    |
| Vehicle | The device is a vehicle infotainment system. |
| Other   | Other device types.                          |

</details>

</details>

## Auto-collected Events

Conviva automatically collects rich set of app performance metrics through app events after completing the [Quick Start](#quick-start).

<details>
  <summary><b>Auto-collected events table</b></summary>

| Event                    | Occurrence                                                                                                                              |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| network_request          | After receiving the network request response. [Refer limitations](#limitations).                                                        |
| application_error        | When an error occurrs in the application.                                                                                               |
| button_click             | On the button click callback. [Refer limitations](#limitations).                                                                        |
| link_click               | On the link click callback. [Refer limitations](#limitations).                                                                          |
| application_background   | When visibility state change to `hidden`.                                                                                               |
| application_foreground   | When visibility state change to `visible`.                                                                                              |
| Largest Contentful Paint | Timing information about the largest image or text paint before user input on a web page.                                               |
| First App Launch         | First time launch in the browser. Custom Tag Context.                                                                                   |
| page_loaded              | On `"load"` event listener.Used to compute Page Loads, Avg Document Load Time, Avg DNS Lookup Time, Avg Document Response Time metrics. |
| sse_conversation_message | When a Server-Sent Event (SSE) of type conversation_message is received via Fetch eventstream. |
| websocket_events | When WebSocket events occur (open, close, send, receive, error) for real-time communication tracking. |

To learn about the default metrics for analyzing the native and web applications performance, such as App Crashes, Avg Screen Load Time, and Page Loads, refer to the [App Experience Metrics](https://pulse.conviva.com/learning-center/content/eco/eco_metrics.html) page in the Learning Center.

</details>

<details>
<summary><b>Auto-collection of SSE Events (EventStream via Fetch API)</b></summary>

The Conviva JavaScript Agent SDK supports auto-collection of Server-Sent Events (SSE) when they are delivered as an `eventstream` response via the Fetch API. This enables tracking of real-time server-push events in modern web applications.

- **Current Coverage:**
  - Primary events of type `CONVERSATION_MESSAGE` are automatically collected and tracked.
  - First token events (`CONVERSATION_STREAMING_TOKEN`) are tracked when chat begins.
  - Added support to count Tokens shared between start and end of chat.
  - Extra events can be configured via `extraEvents` array (e.g., `CONVERSATION_CLOSE_CONVERSATION`, `CONVERSATION_PARTICIPANT_CHANGED`).

No extra configuration is required—if your application receives SSEs as an `eventstream` through Fetch, supported events will be tracked automatically.

</details>

<details>
<summary><b>Auto-collection of WebSocket Events</b></summary>

The Conviva JavaScript Agent SDK supports auto-collection of WebSocket events for real-time communication tracking. This enables comprehensive monitoring of WebSocket connections, messages, and errors in modern web applications.

- **Current Coverage:**
  - **Connection Events**: WebSocket open and close events with connection details (URL, status codes, reasons).
  - **Message Events**: Both client-sent messages (`conviva_ws_send`) and server-received messages (`conviva_ws_server`).
  - **Error Events**: WebSocket errors with comprehensive error details, network information, and browser context.

- **Event Details:**
  - `conviva_ws_open`: Tracks when WebSocket connections are established
  - `conviva_ws_close`: Tracks connection closures with status codes and reasons
  - `conviva_ws_send`: Tracks outgoing messages with URL and message data
  - `conviva_ws_server`: Tracks incoming server messages with URL and parsed data
  - `conviva_ws_error`: Tracks errors with detailed error information and network context

No extra configuration is required—if your application uses WebSocket connections, all events will be tracked automatically.

</details>

### Limitations

<details>
  <summary><b>Clicks</b></summary>

The collection of all types of clicks is automatically supported, including those from standard HTML elements as well as elements created using React, Angular, and Vue frameworks. We also offer an experimental remote configuration specifically for click events, aiming to dynamically add support for non-standard or unsupported frameworks. For further assistance, please contact the Conviva support team.
**Note:** `preventDefault` and `stopPropagation` will prevent the auto-collection of button and link click events.

**Migration of Pulse dimensions for clicks**

Starting with version [v1.1.2](https://github.com/Conviva/conviva-js-appanalytics/releases/tag/v1.1.2) of the SDK, the attribute keys for click events have been updated.
If you are using v1.1.1 or earlier and currently mapping `elementText`, you must update your configuration when upgrading to v1.1.2 or later. Specifically, update the mapping in [DPI Activation](https://pulse.conviva.com/app/activation/home) by mapping `elementText` to `text`, then redeploy to apply the changes.

To ensure metrics reflect the updates, please review and update your event/metric mappings in [DPI Activation](https://pulse.conviva.com/app/activation/home) if you are using any of the following attributes:
| <=v1.1.1 | >=v1.1.2 |
|--------------------------------|--------------------------------|
| elementType | elementType |
| elementText | text |
| elementName | elementName |
| elementValue | value |
| elementId | id |
| elementClasses | class |

</details>

<details>
  <summary><b>network_request</b></summary>

This feature supports tracking network requests triggered within the application using `XMLHttpRequest` and the Fetch API.

**Request and Response Body Collection:**

Collected only when:

- Size is < 10KB.
- Response body is type JSON.
- Content-type contains `"json"` or equals any of `"text/plain"`, `"text/javascript"`, `"application/javascript"`, `"text/html"`
- Response Type is not "opaque"

**Request and Response Header Collection:**

Collected only when:

- The server is provisioned with `"Access-Control-Expose-Headers:"`.
- Response Type is not "opaque"

</details>

<details>
  <summary><b>Clean up</summary>
"cleanup" api support is not available in older browsers(Chrome: < 66, Mozila: < 57, Safari: < 12.1)
</details>

## FAQ

[DPI Integration FAQ](https://pulse.conviva.com/learning-center/content/sensor_developer_center/tools/eco_integration/eco_integration_faq.htm)
