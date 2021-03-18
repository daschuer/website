title: "Has JACK zero latency?"
authors: Daniel Schürmann
tags: 2.3, jack
date: 2021.03.18 02:42:18

In regular intervals, we discuss how much latency the JACK layer introduces. The [JACK FAQ](https://jackaudio.org/faq/no_extra_latency.html) state that:
> There is **NO** extra latency caused by using JACK for audio input and output. When we say none, we mean absolutely zero.
This is true on its own, because JACK uses directly the buffer provided by ALSA to mix the audio sources together. ALSA has a second buffer that is used to feed the samples into the hardware. That's all.

However in the case of Mixxx another buffer is used, because Mixxx has its own realtime mixing stage and uses the PortAudio abstraction layer.

When using the ALSA API directly Mixxx does what Jack does. It uses the ALSA buffer directly, which doesn't add any latency.

This can be confirmed by recording the own sound via a mic and measure the round trip latency:

![Screenshot of audacity showing the round trip latency]({static}/images/news/roundtriplatency.png)


The upper stream is the JACK case. The left channel is the recorded master 440 Hz sine wave and the right channel is the mic input.

JACK is configured with a 1024 frames buffer and reports a latency of 46.4 ms for the sum of two buffers .
The roundtrip latency is 95 ms ~4 buffers = Driver + ALSA + 2 x Mixxx

The lower stream is the ALSA case. Mixxx is configures with the same single buffer of 1024 frames = 23.2 ms
The roundtrip latency is 49 ms ~2 buffers = Driver + ALSA

Not in the picture is the ALSA pulse device. It runs at a latency of 104 ms ~5 buffers. Driver + ALSA + 2 x Pulse + ALSA

With this picture we can verify that Mixxx actually has the same buffer size in both cases. When pressing pause. It fades this signal out over one buffer length these is equal in both cases.
The peaks in the recorded right channel is the sound of the mouse click. You can only barely see the recorded sine wave.

![Screenshot of audacity showing the fade out]({static}/images/news/fadeoutcompare.png)

Another disadvantage of Jack is that it can't deal (well) with two or more sound cards. Mixxx can do this using the ALSA API and this with also no extra latency as long the underlying driver allows it.

## Conclusion

Don't use Jack when you don't need it for another reason.  The same will be true for Pipewire using the same architecture.

## What comes next?

There is a chance to get rid of the extra latency in these cases, and make Mixxx use the buffer provided by Jack/Pipewire. This requires that Mixxx becomes a native Jack or Pipewire application.

A lot to do. Do you have interest to help? Get in contact with us at [Zulip](https://mixxx.zulipchat.com)