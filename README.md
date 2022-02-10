# <a href="https://par-ity.github.io/platypus">Platypus</a>

<img src="https://i.pinimg.com/originals/17/5e/f2/175ef22c95918002bba266a898644de8.jpg">

## 2021 Update

This was my first big web project and it is structurally quite monolithic and problematic - error handling and setIntervals and driven by global mutations and such.the control flow is quite difficult to follow but largely appears to work performantly.

I've made a few fixes and changes, mostly unloading resources on error and modularising the feature detection script, and because of better browser-security a breaking change around asking for user media. running the feature detection module on input fixed this.

### Diagrams

![I/O diagram of how realtime microphone data is processed](img/platypus-io-process.png?raw=true "Platypus I/O processing diagram")

This node diagram outline the core roles of the API’s and Web Worker in turning microphone access into a mp3 and displaying that to the user.

The paths within light blue areas are callbacks which repeat until stopped to handle the live stream:

The Microphone → Canvas Display loop repeats (max: 60 fps) to produce the visualisation.

The Microphone → Mp3 Buffer loop fires once for every 16384 bytes the processor receives.
 
The red arrow between Mp3 Array Buffer and Blob URL is not a loop - it indicates data is sent back to main.js once per recording to be constructed into a Blob and displayed. the light red Blob URL node represents the end of these resources runtime.

Because the real encoding work is done on repeat to build up the Mp3 Buffer, when the user clicks stop the buffer is sent back to main.js, they can expect to get an mp3 to play almost immediately.

![Diagram of how mp3 frames are sliced for audio editing](img/mp3-frame-editing.png?raw=true "Mp3 frame slicing diagram")

Mpeg-1 layer 3 (mp3) encoding uses a ‘padding-bit’ to ensure the frame uses sensible values like 418 bytes instead of an absurd float like 417.95918367. The [LAME FAQ](https://lame.sourceforge.io/tech-FAQ.txt) specifies if a padding-bit exists, then the bits per frame value is to be rounded to the nearest byte. If no padding-bit exists (as in other mpeg layers) float values are to be rounded down to the nearest byte.

## 2017 Readme

An audio tool which uses the Web Audio API to record, visualize and edit mp3 blobs

Adapted from Zhuker's excellent LAME.js library https://github.com/zhuker/lamejs

The full 4.47mb Library can be found there but is omitted here to reduce app size

Platypus uses a callback polyfill for the Streams API getUserMedia promise

Platypus uses a monkey patch to alias Web Audio API syntax for WebKit browsers

Platypus uses FileSaver.js to give blob downloads semantic filenames instead of UID's

~ | ~

This repo also contains a feature detection script for all core dependencies and also<br>
run Platypus audio specific tests properties on the following two bugs:

- Sometimes the audio blob is properly encoded but the browser fails to display it;<br>
  to counteract this a sample blob mp3 from Platypus is included to test blob URLs

- Sometimes the hardware and browser prefers sample rates and buffer sizes which<br>
  can cause a sporadic mixture of sped-up playback and high-end artifacts; while this<br>
  is currently fixed via a forced huge bufferSize of 16384 bytes, new devices and<br>
  operating systems may have issues reoccur. To help debug them the preferred<br>
  sample and buffer values are also saved.

When run, this script saves the level of support and preferred sample/buffer values;<br>
these are all returned as an object to the IsMicSupported global module for access