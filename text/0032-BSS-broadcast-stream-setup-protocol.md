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

**Implementation**


# Motivation

What problem is this solving? What are the common use cases?

# Drawbacks

What is the potential detriment for adding this feature/change?

# Additional Information

Any additional information that may not be covered above that you feel is relevant. External links, references, examples, etc.
