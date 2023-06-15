# Linux + Thinkpad speaker improvements

Thinkpads ship by default with Windows, which comes with Dolby Atmos software that improves the sound quality coming from the speakers significantly. This is very noticeable if you turn the Dolby effects on and off while playing music. This software is not available under Linux, which means the sound quality is roughly the same as what you hear with the Dolby effects turned off. The resulting quality can be pretty bad. For example, at high volumes, I can hear significant chassis resonance for certain laptop models while playing music.

The good news is that there is a way to improve the speaker sound quality. It seems like the Dolby software is doing at least two things to improve speaker quality:

1. It applies a convolution on the actual sound directly in the operating system.
2. It might also be interacting with the speakers differently than Linux.

The first item is something we can emulate on Linux while the second item is not. Thus the bad news is the sound quality from Windows + Dolby will always be better than Linux, until someone can reverse engineer with Dolby is doing to the speakers post convolution. This guide shows you how to apply the same convolution that Dolby is applying in Linux.

## Obtaining the impulse response profile (.irs) files from Windows

We need to figure out how the filtering is done with Dolby. To do this, we can simply play an impulse audio file, and measure the output audio, which is the impulse response and we can use it in a convolution filter. Step by step:

1. Install `Audacity` and `VLC` on Windows.
2. Enable all Dolby effects using the Dolby app (`Dolby Audio Premium` at the time of writing this). Convince yourself that it is working by playing some music and turning the Dolby effects on and off.
3. Open `Audacity`, select `Edit` -> `Preferences` and then select the `Audio Settings` on the left tree.
4. Under `Interface` -> `Host`, select `Windows WASAPI`
5. Under `Playback` -> `Device`, select the speakers (`Speaker (Realtek(R) Audio)` for me).
6. Under `Recording` -> `Device`, select the speakers loopback (`Speaker (Realtek(R) Audio) (loopback)` for me).
7. Under `Quality`, make sure the `Project Sample Rate` and `Default Sample Rate` is `48000Hz`.
8. I selected 24-bit for the `Default Sample Format`. 32 bit float might be fine as well.
9. Click `OK` to save the settings.
10. Download the impulse WAV file from either [the original source](https://freesound.org/people/unfa/sounds/205620/) or [a mirror of it in this repo](impulse48khz-2sec.wav).
11. Open the WAV file with VLC. Pause the playback.
12. Go to `Tools` -> `Preferences` in VLC. On the bottom left, it says `Show settings` and there are two radio boxes, `Simple` and `All`. For me, `Simple` is selected, click `All` to get a more detailed preferences menu.
13. On the left tree, go to `Audio` -> `Output modules`.
14. On the right side, the `Audio output module` should be `Windows Multimedia Device output` and the `Media role` should be set to `Video`. I also tried a `Media role` of `Music`, but it made no difference in the impulse response profile on my machine.
15. Click `Save` to save the settings.
16. Go back to Audacity, click the Record button (big circle at the top). The recording might be stuck (no waveform shows on screen), which is OK.
17. Click play in Audacity. This should cause the recording to show a waveform in audacity.
18. Stop the recording in Audacity and stop playback in VLC (if necessary).
19. Zoom into the peak of the wavform and see something like the following. You have to zoom in quite far. In my screenshot below, that's a range of abouy 4ms. If you see just a single peak without anything else even if you zoomed in very far, you probably did something wrong?

<img src="./ImpulseResponseScreenshot.PNG" />

20. Select the audio clip centered around the maximum of the waveform, as shown in the above screenshot. Then go to `File` -> `Export Selected Audio`. Save the file as a WAV file.
21. Rename the resulting file's extension from `.wav` to `.irs`. Keep this file around.

## Ubuntu 22.04 via PulseEffects

PulseEffects is unmaintained as development has moved to the pipewire based EasyEffects. That said, pulseeffect is widely available in existing distros. For Ubuntu 22.04:

1. `sudo apt intall pulseeffects`
2. Activate the convolver.
3. Load the irs file.
4. Make sure the default sound output is to PulseEffect.

## Results

I found that after the filter, the sound is still quieter than Windows although most of the chassis resonance is gone. Under Windows + Dolby, the sound is also a bit more clear. I suspect maybe the Dolby software is causing the system to interact with the speakers differently, perhaps in a similar manners to [how Macbooks does it](https://news.ycombinator.com/item?id=34935483).

There's also a slight increase in latency (~10ms?), which is mostly ok for music. I have not tested video yet.

## Sources

1. https://stackoverflow.com/a/53688163
2. https://github.com/wwmm/easyeffects/issues/2209
3. https://old.reddit.com/r/thinkpad/comments/q5pt38/x1_extreme_gen_4_dolby_atmos_setup_for_linux/
4. https://news.ycombinator.com/item?id=34935483
5. https://freesound.org/people/unfa/sounds/205620/
