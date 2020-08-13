# Summary

**Better OBS settings setup using a URL protocol.**

Add a URL hook to OBS to configure the encorder and more. Make this so a webapp can be made to help setup your OBS config or alternatively a braodcast platform's website can suggest import settings, possibly a "send to OBS" or "send to streaming software" button on sites.

Making the protocol open and able to work both from multiple vendors to multiple streaming packages could encuorage adoption.


**Proposed Protocol**
`bss:` for for "broadcast stream setup"

`bss://settings?streamkey=XXXXX`

Longer example would be

`bss://settings?service=facebook&service_url=cnRtcHM6Ly9saXZlLWFwaS1zLmZhY2Vib29rLmNvbTo0NDMvcnRtcC8%3D=&stream_type=rmtp&stream_key=NjQzNzgzMT9zX2JsPTEmc19zbWw9Mw%3D%&max_bitrate=6000&bitrate=4300&max_resolution_v=1280&max_resolution_h=720`

which carries all the following fields to be set into OBS (or another streaming product)

 - `service`
   - facebook
 - `stream_type`
   - rmtp
 - `service_url` (base64Encoded -> URLEncoded)
   - rtmps://live-api-s.facebook.com:443/rtmp/
   - cnRtcHM6Ly9saXZlLWFwaS1zLmZhY2Vib29rLmNvbTo0NDMvcnRtcC8=
   - cnRtcHM6Ly9saXZlLWFwaS1zLmZhY2Vib29rLmNvbTo0NDMvcnRtcC8%3D
 - `stream_key` (URL encoded, API keys can have special characters)
   - 6437831?s_bl=1&s_sml=3
   - NjQzNzgzMT9zX2JsPTEmc19zbWw9Mw==
   - NjQzNzgzMT9zX2JsPTEmc19zbWw9Mw%3D%3D
 - `backup_stream_key` (URL encoded, API keys can have special characters)
   - 58463875?s_bl=1&s_sml=3
   - NTg0NjM4NzU/c19ibD0xJnNfc21sPTM=
   - NTg0NjM4NzU%2Fc19ibD0xJnNfc21sPTM%3D
 - `max_bitrate` (maximum kpbs)
   - 6000
 - `bitrate` (kbps, possibly if the service has it's own speed test built in)
   - 4300
 - `max_res_h` (highest supported by the service, for the user)
   - 720
 - `max_res_w` (highest supported by the service, for the user)
   - 1280

this would import the settings into the software with a confirmation dialog. All the fields are optional e.g., I would see cases `max_bitrate` and `bitrate` are mutually exclusive when a speedtest determined the bitrate.

**Encoding**

The fields are base64 encoded and then URL encoded becuase URL encoding is removed by both Windows and MacOS when passes to the application from browsers.

> Because Internet Explorer will decode all percent-encoded octets in the URI before passing the resulting string to ShellExecute, URIs such as alert:%3F? will be given to the alert application pluggable protocol handler as alert:??. The handler won't know that the first question mark was percent-encoded. To avoid this issue, pluggable protocol handlers and their associated URI scheme must not rely on encoding. If encoding is necessary, protocol handlers should use another type of encoding that is compatible with URI syntax, such as Base64 encoding. Double percent-encoding is not a good solution either; if the application protocol URI isn't processed by Internet Explorer, it will not be decoded.

> Source: https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/platform-apis/aa767914(v=vs.85)?redirectedfrom=MSDN


**Implementation**

_MacOS_

I wrote an example application how to open and handle. In this case the application is already registered and it's a matter of handling Qt's [QFileOpenEvent](https://doc.qt.io/qt-5/qfileopenevent.html) event: https://github.com/JohannMG/QtUrlOpenApplication/

This also requires changes to the application installer but this is a small change: [Apple Developer: Defining a Custom URL Scheme for Your App](https://developer.apple.com/documentation/xcode/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)

Mac seems to be the easier implementation.

_Windows_

Windows requires registration as well. But additionally, while mac applications attempt to keep only one ionstance of an application running, windows  always handles URL protocols by spawning a new executable and it is your responsibility to have that new exe process signal the already running one if that's the logic you want. OBS already has similar


# Motivation

What problem is this solving? What are the common use cases?

# Drawbacks

What is the potential detriment for adding this feature/change?

# Additional Information

Any additional information that may not be covered above that you feel is relevant. External links, references, examples, etc.
