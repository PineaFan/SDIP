# Bugs in iOS Shortcuts

While making SDIP, I have found a few unavoidable bugs. Below are examples of how to reproduce them in the easiest way to check if they persist.

### 1. Split Text
| Status | iOS | iPadOS |
|-|-|-|
| Last working | Unknown | Unknown |
| Still broken in  | iPadOS 18.2 (22C5109p) | iOS 18.1 (22B83) |

Working examples:
```
01 Text [12345 123456:123]
02 Split [01] by [Spaces]
Returns: "12345", "123456:123"

01 Text "12345 123456:12345"
02 Split [01] by [Spaces]
Returns: "12345", "123456:12345"
```

Failing example:
```
01 Text "12345 123456:1234"
02 Split [01] by [Spaces]
```

The example above fails with no reason. The characters can be replaced with anything, except the ` ` and `:`.
The _length_ is what matters, this is the only combination I can find that fails


### 2. Find Contacts
| Status | iOS | iPadOS |
|-|-|-|
| Last working | Unknown | Unknown |
| Still broken in  | iPadOS 18.2 (22C5109p) | iOS 18.1 (22B83) |

Being able to find contacts was important for SDIP, however, the find contacts instruction does not always work.
The instruction works as expected, except when a phone number is in use. Try:
```
01 Find [All Contacts] where
   [Phone Number] [is] [HELLO WORLD]
Returns: All contacts on your device
```
Obviously, nobody's phone number is `HELLO WORLD`, except this seems to match everyone in your phone?
