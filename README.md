# SDIP
iOS Shortcuts Device Interaction Protocol

SDIP is designed to be an easy to use way to request and recieve data from your own devices, or others who have SDIP installed.
It runs entirely through iMessage, or through emails, which can be picked up by the other user's shortcut automations.

The whole shortcut is designed to need minimal setup, as described below.

# Setting up
Firstly, you'll need to install all the shortcuts. There are a lot, but they should work with no issues on new devices.
The shortcuts below are from the `SDIP Internals` folder. I'd recommend creating a new folder in Shortcuts for these.
| Shortcut | Required? | Purpose |
|-|-|-|
| Inbound | Yes | Handles incoming messages for processing, so the automation can be simplified |
| Outbound | Yes | Sends outgoing messages in the required format when given specific parameters as an input |
| Device Block | Yes | Ignores requests made to other decices, so that only the devices you need will reply |
| Dictionary to Text | Yes | Converts a dictionary of parameters into the `SDIP://x@y:z?...` form for messages |
| Dictionary to Parameters | Yes | Creates the `?w=x&y=z` parameter form from a dictionary input |
| Text to Dictionary | Yes | Takes a '?w=x&y=z' input and creates a dictionary from it. Could fail in rare situations due to [Bug 01](https://github.com/PineaFan/SDIP/blob/production/BUGS.md#1-split-text) |
| Find Contact ID | Yes | Due to [Bug 02](https://github.com/PineaFan/SDIP/blob/production/BUGS.md#2-find-contacts), this function is designed to more accurately find contacts based on given inputs |
| Helpers - Opposite Device | Likely | If run on an iPhone, this returns "iPad", and vice versa. Relied upon by most modules |
| Modules - Battery | No | Responds to and shows battery requests |
| Modules - Silent | No | Remotely puts another device on silent |
| Modules - Notify | No | Sends any text as a notification to another device (Except `=` and `&`, as it would affect parameters |
| Modules - Lock | No | Remotely locks *all* devices with SDIP enabled |
| Modules - Location | No | Currently bugged (due to timing out, most likely), returns the device's address, lat/lon/alt, and if it's in motion (walking, car, stationary) |

Once the modules above are installed, you need to create an automation. This has been designed to be as minimal as possible:

```
# For messages:
When I get a message from [Any allowed senders, you must be included in this list]; if message contains "SDIP://"; Run immediately
# For emails:
When I get an email from [Any allowed senders, you mist be included in this list]; if subject contains "SDIP://"; Run immediately

# For both
00 Recieve (Messages/Emails) as input
01 Dictionary
   "message": [Shortcut Input > Content]
   "contact": [Shortcut Input > Sender]
02 Run Shortcut [SDIP Inbound] with input [Dictionary 01]
```

The final step is to tell SDIP who you are. This is done by creating a new line in your contact's note and adding `SDIP:$Me`.
To send messages to other contacts, you can replace `$Me` with any text, usually their name (e.g. `SDIP:Ash`)

That's it, you're all configured! All that's left is to install some requests. These are the buttons you can actually click to run a request. They can be found in the SDIP Requests folder.


# Creating a Module
SDIP is designed to be expandable, and so, it should be easy to create your own!

There are two types of SDIP shortcuts - Requests and Modules.
- A *request* sends an initial message, letting another device know it should respond.
- A *module* takes an incoming message and does something in response. This could send another message, or show an alert.

### Requests
Let's take the example of a module which lets you know your other device's battery level. First, the request. This needs to send a message to yourself, and request the battery.
SDIP has a few types of requests. While there's nothing enforcing them, you should aim to follow this guide:

| Type | Purpose |
|-|-|
| `get` | Asks another device for information |
| `act` | Tells another device to perform an action |
| `reply` | Either returns the data of a get request, or tells the device it has completed an action |

As this module only needs to fetch the battery level, we'll use a `get` request.

Outbound requests take 4-5 arguments, all of which are defined in a dictionary:

| Key | Type | Value | Example |
|-|-|-|-|
| `contact` | Text | The recipient of a request. This can be a phone number, email, or an ID, such as `$Me` or `Ash` (for a contact with `SDIP:Ash` in their note) | `$Me` |
| `targetDevice` | Text | The intended device type. Currently, this can only be something returned from `[Get Device Details > Device Model]` | `iPad` |
| `action` | Text | The module name | `battery` |
| `requestType` | Text | A request type listed above | `get` |
| `parameters` | Dictionary, optional | A dictionary of text, numbers or boolean values which are sent in the request | `{ "charging": true, "battery": 99 }`

For the battery module, we'll use the opposite device helper:
```
01 Run Shortcut [SDIP Helpers - Opposite Device]
02 Dictionary
   "contact": "$Me"
   "targetDevice": [Shortcut Result 01]
   "action": "battery"
   "requestType": "get"
03 Run Shortcut [SDIP Outbound] with input [Dictionary 02]
```
That's it, the first request! Running this should send a text with `SDIP://get@iPhone:iPad/battery`

### Modules
Modules are provided with a dictionary as an input, and return either nothing or a dictionary.

Provided input by Inbound:
| Name | Type | Value example |
|-|-|-|
| `requestType` | Text | `act`, `get` |
| `parameters` | Dictionary | Any dictionary passed to Outbound as parameters |
| `targetDevice` | Text | `iPad` or `iPhone` |
| `requestingDevice` | Text | `iPad` or `iPhone` |
| `action` | Text | Module name, such as `battery` |

The module can either return nothing (`Stop this shortcut` or `Stop and return []`), or a dictionary with outbound parameters.

This time however, less parameters are required, as the contact and device is already known. For this reason, only the following should be provided:
| Name | Type | Example |
|-|-|-|
| requestType | Text | `reply`, `act` |
| parameters | Dictionary | Any dictionary with Text, Number or Boolean keys |

That's it, you've made a module! By checking the input variables, you can do different things, and you can track complex instructions through parameters.
If you ever get stuck, take a look at the pre-made modules too. As long as shortcut names aren't changed, they should work once installed.
