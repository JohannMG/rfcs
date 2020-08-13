# Summary

**Better OBS settings setup using a URL protocol.**

Add a URL hook into OBS to configure settings. This will enable applications and webpages that help setup your OBS instance. These settings packages could be shared. Or, alternatively, a broadcast platform's website can suggest import settings, with a "send to OBS" or "export BSS settings" button on sites.

By making the protocol open and able to work both from multiple vendors and to multiple streaming packages we would encourage adoption. The ability to export from the service webpages also would require fewer log-ins and service-specific API implementations.


**Proposed Protocol**
Implement `bss:` for "broadcast software setup", a simple example would be `bss://settings?streamkey=XXXXX` where the user can export the stream key using a URL hook.

A longer example would be

`bss://settings?service=facebook&service_url=cnRtcHM6Ly9saXZlLWFwaS1zLmZhY2Vib29rLmNvbTo0NDMvcnRtcC8%3D=&stream_type=rmtp&stream_key=NjQzNzgzMT9zX2JsPTEmc19zbWw9Mw%3D%&max_bitrate=6000&bitrate=4300&max_resolution_v=1280&max_resolution_h=720&gop_seconds=2`

which carries all the following fields to be set into OBS (or another streaming product)

 - `service` <string, encoded>
   - facebook
   - ZmFjZWJvb2s=
   - ZmFjZWJvb2s%3D
 - `stream_type` <string, encoded>
   - rmtp
   - cm10cA==
   - cm10cA%3D%3D
 - `service_url` <string, encoded>
   - rtmps://live-api-s.facebook.com:443/rtmp/
   - cnRtcHM6Ly9saXZlLWFwaS1zLmZhY2Vib29rLmNvbTo0NDMvcnRtcC8=
   - cnRtcHM6Ly9saXZlLWFwaS1zLmZhY2Vib29rLmNvbTo0NDMvcnRtcC8%3D
 - `stream_key` <string, encoded> (API keys can have special characters)
   - 6437831?s_bl=1&s_sml=3
   - NjQzNzgzMT9zX2JsPTEmc19zbWw9Mw==
   - NjQzNzgzMT9zX2JsPTEmc19zbWw9Mw%3D%3D
 - `backup_stream_key` <string, encoded>
   - 58463875?s_bl=1&s_sml=3
   - NTg0NjM4NzU/c19ibD0xJnNfc21sPTM=
   - NTg0NjM4NzU%2Fc19ibD0xJnNfc21sPTM%3D
 - `max_bitrate` <int, plain> (maximum kpbs)
   - 6000
 - `bitrate` <int, plain> (kbps, possibly if the service has it's own speed test built in)
   - 4300
 - `max_res_h` <int, plain> (highest supported by the service, for the user)
   - 720
 - `max_res_w` <int, plain> (highest supported by the service, for the user)
   - 1280
 - `gop_seconds` <int, plain> (GOP length in seconds, important for latency but also varied per service)
   - 2

this would import the settings into the software with a confirmation dialog. All the fields are optional e.g., I would see cases `max_bitrate` and `bitrate` are mutually exclusive when a speedtest determined the bitrate.

**Encoding**

All string fields are base64 encoded and then URL encoded because URL encoding is removed by both Windows and MacOS when passes to the application from browsers.

> Because Internet Explorer will decode all percent-encoded octets in the URI before passing the resulting string to ShellExecute, URIs such as alert:%3F? will be given to the alert application pluggable protocol handler as alert:??. The handler won't know that the first question mark was percent-encoded. To avoid this issue, pluggable protocol handlers and their associated URI scheme must not rely on encoding. If encoding is necessary, protocol handlers should use another type of encoding that is compatible with URI syntax, such as Base64 encoding. Double percent-encoding is not a good solution either; if the application protocol URI isn't processed by Internet Explorer, it will not be decoded.

> Source: https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/platform-apis/aa767914(v=vs.85)?redirectedfrom=MSDN


**Implementation**

_MacOS_

I wrote an example application as an example to open and handle url protocols. In this casem, the application is already registered and it's a matter of handling Qt's [QFileOpenEvent](https://doc.qt.io/qt-5/qfileopenevent.html) event: https://github.com/JohannMG/QtUrlOpenApplication/

This also requires changes to the application installer but this is a small change: [Apple Developer: Defining a Custom URL Scheme for Your App](https://developer.apple.com/documentation/xcode/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)

Mac seems to be the easier implementation. With the caveat of needing a signed copy to test URL/protocol registration.

_Windows_

Windows requires registration as well. But additionally, while mac applications attempt to keep only one instance of an application running, windows  always handles URL protocols by spawning a new executable and it is your responsibility to have that new exe process signal the already running one if that's the logic you want.

OBS already has a check to verify to warn the user to use one running instance of OBS: https://github.com/obsproject/obs-studio/blob/478f1de8468223524f92aafd194675c89947c544/UI/obs-app.cpp#L1940

The solutions I found here that are native to Qt are using `QSharedMemory` and `QLocalSocket` to pass information between between instances. Ideally then we use the same parser for both Mac and PC since they should be identical by the parsing time.

After handoff the newer process should focus the first application version and then terminate.

_Linux_

I'm unclear on launch protocols for URL handling but Qt messageing in windows should work similarly.


# Motivation

_What problem is this solving? What are the common use cases?_

Users have a lot of setting to tweak on first setup and per-service. It would be ideal in many cases if the service could export ideal settings for the user.

From the streaming perspective, it's always ideal when can get a user on the best settings and without asking them to log in anywhere additionally. Some services have configuration APIs but they require creating an additional application and another authentication step.

# Drawbacks

_What is the potential detriment for adding this feature/change?_

 **There are many states to handle:**
 - OBS is closed and needs to open with correct settings
   - Open and put settings in. Show success dialog.
 - OBS is open with the wizard open
   - Put in settings, update wizard form. Show success.
 - OBS is open with main window open
   - Update settings. Show success dialog.
 - OBS is open and already streaming
   - No update. Show failure: won’t update during stream.
 - OBS is open and already recording
   - Update stream settings only. Show success for stream settings.
 - OBS is open and settings is open
   - Update settings field if showing. Show success dialog.

**Verification**

Will need a confirmation UI for either the entire batch of settings or per-setting to avoid security issues and errors.

**Maintenance**

I've listed above the primary endpoints I think would benefit users. When adding more we'd need to document / surface somewhere for web applications to use the protocol to it's fullest extent.

# Additional Information

Any additional information that may not be covered above that you feel is relevant. External links, references, examples, etc.
