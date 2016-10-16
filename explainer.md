## Explainer
Some projects using WebRTC functionality have indicated that they need to
record data on "source to screen" performance - that is, information about
how much time it takes between some event (typically frame capture) occurs in
"real life" on the media-generating side and the serving of the same event
to the user on the media-consuming side.

For video, this is typically composed of:
* Frame capture time
* Time spent to deliver the frame from the camera to the computer
* Time spent encoding the frame using a given codec
* Time spent preparing the frame for transmission
* Time spent sending the frame (possibly subject to congestion control)
* Time spent waiting for the frame to transit the network
* Time spent reassembling the frame
* Time spent decoding the frame
* Time spent transferring the frame to the display device's framebuffer
* Time spent in the display device

In WebRTC, we typically have some idea of the middle step (network transit),
usually estimated by "half the round trip time", and this is exposed through
the stats API, but other elements of the event chain aren't explicitly recorded
anywhere, and any stats that are available are usually described using averages
or accumulated counters, not per-frame data.

This API proposal is intended to expose such data on a per-frame basis.

### Examples

#### Measuring action response in a remote-desktop app

In a remote-desktop app, it is important to know the time from an
action is taken (a mouse click, for instance) and the time the frame
showing the result of the action is presented to the user.

This can be done, for example, by:
* Measuring in JS the time at system A when the mouse click happens
  (ignoring driver latencies)
* Sending over a WebRTC datachannel the mouse event, with an
  identifier for it
* Having system B (where the action happens) record the RTP timestamp
  of the first video frame generated after the mouse click, and
  returning this RTP timestamp + the identifier for the click over the
  datachannel back to system A
* Having system A record the time taken from the arrival of frames
  generated after the mouse click to the display of the frame, using
  the mechanism described here
* Picking the right frame from the data stream based on the RTP
  timestamp

The app now has information enough to definitely measure the time from
the mouseclick to the display of the frame generated after the
mouseclick.

This particular example uses the API described here only on the
receiving side of the video stream; all other information is already
available through other APIs.


