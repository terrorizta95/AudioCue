# AudioCue

**AudioCue** is a new Java resource for playing back sound files, 
designed for use with game programming. 

## Why?

Java's **Clip** class (_javax.audio.sampled.Clip_) was not built 
with game programming needs in mind. The class has a tricky, 
non-intuitive syntax and a limited feature set. A _Clip_ cannot
be played concurrently with itself, can only be played at its 
recorded pitch, and the _Control_ class provided for real time
changes such as panning and volume is system-dependent upon
implementation and limited by only allowing changes at buffer 
increments. 

**AudioCue** addresses these issues:

* ### Easy to Use
  * Very light: Download or copy/paste five class files from GitHub directly into your project.
  * Syntax is simpler than Java's _Clip_ class.
  * API and demonstration programs provided.
* ### Powerful
  * Runs directly on Java's _SourceDataLine_.
  * Allows concurrent playback of cues.
  * Allows  playback at varying speeds.
  * Real-time volume, panning and frequency faders.
  * Highly configurable.
  * Messaging system for coordination with graphics.
* ### BSD License (open source, free, donations greatly appreciated)
* ### Now includes *AudioMixer* for consolidating AudioCues into a single output line.

## Installation

AudioCue requires five files:
* [AudioCue.java](https://github.com/philfrei/AudioCue/blob/master/com/adonax/audiocue/AudioCue.java)
* [AudioCueInstanceEvent.java](https://github.com/philfrei/AudioCue/blob/master/com/adonax/audiocue/AudioCueInstanceEvent.java)
* [AudioCueListener.java](https://github.com/philfrei/AudioCue/blob/master/com/adonax/audiocue/AudioCueListener.java)
* [AudioMixer.java](https://github.com/philfrei/AudioCue/blob/master/com/adonax/audiocue/AudioMixer.java)
* [AudioMixerTrack.java](https://github.com/philfrei/AudioCue/blob/master/com/adonax/audiocue/AudioMixerTrack.java)

In addition, there are two optional file folders with demo content and resources used by the demo programs:
* [supportpack](https://github.com/philfrei/AudioCue/tree/master/com/adonax/audiocue/supportpack)
* [supportpack.res](https://github.com/philfrei/AudioCue/tree/master/com/adonax/audiocue/supportpack/res)

Installation involves copying and pasting the five files into your 
program.

* Method 1) navigate to, then copy and paste the five files
directly into your program.
* Method 2) download and unzip [audiocue.zip](http://adonax.com/AudioCue/audiocue.zip), which holds
these five files, into your project.
* Method 3) download [audiocue.jar](http://adonax.com/AudioCue/audiocue.jar), 
which includes source code, the "supportpack" and "res" content, 
and import into your IDE.
_[NOTE: I'm not clear if the .jar file, which I generated from Eclipse, 
can be imported into other IDEs.]_

## Usage

    // Simple case example ("fire-and-forget" playback):
    // assumes sound file "myAudio.wav" exists in same file folder,
    // we will allow up to four concurrent instances.
    
    // Preparatory steps (do these prior to receiving playback call): 
    URL url = this.getClass().getResource("myAudio.wav");
    AudioCue audioCue = AudioCue.makeStereoCue(url, 4); //allows 4 concurrent
    audioCue.open();  // see API for parameters to override defaults
    
    // For playback, normally done on demand:
    audioCue.play();  // see API for parameters to override default vol, pan,
                      // pitch, and add Listeners 

    // release resources when sound is no longer needed
    audioCue.close();

### Usage: accessing real time control

An important feature of *AudioCode* is the the ability to drill down 
to individual playing instances of a cue and alter properties in real
time. To drill down to a specific instance, we can one of two methods
to capture an *int* handle that will identify the instance. The first 
is to capture the return value of the play method, as follows:

    int handle = myAudioCue.play(); 

Another way is to directly poll a handle from the pool of available 
instances, as follows:

    int handle = myAudioCue.obtainInstance(); 

An instance that is obtained in the second manner must be directly 
started, stopped, and released (returned to the pool of available 
instances). 

    start(handle); // to start an instance
    stop(handle);  // to stop an instance
    release(handle); // to return the instance to available pool

An important distinction between an instance handle 
gotten from a _play()_ method and the _obtainInstance()_ method is 
that the default value of the boolean field _recycleWhenDone_ differs.
An instance arising from _play()_ has this value set to _true_, and an 
instance arising from _obtainInstance()_ has this value set to _false_.
When an instance finishes playing, if the boolean _recycleWhenDone_ is 
true, the instance is automatically returned to the pool of available 
instances and no longer available for updating. If the value is false,
properties of the instance can continue to be updated, and the 
instance can be repositioned and restarted.

Properties that can be altered for an instance include the following

    //*volume*: 
    myAudioCue.setVolume(handle, value); // double ranging from 0 (silent)
                                         // to 1 (full volume)
    //*panning*: 
    myAudioCue.setPan(handle, value); // double ranging from -1 (full left)
                                      // to 1 (full right)
    //*speed of playback*: 
    myAudioCue.setSpeed(handle, value); // value is a factor, 
                // multiplied against the normal playback rate, e.g.,
                // 2 will double playback speed, 0.5 will halve it 
    //*position*:
    myAudioCue.setFramePosition(handle, frameNumber);
    myAudioCue.setMillisecondPosition(handle, int); // position in millis
    myAudioCue.setsetFractionalPosition(int, double); // position 
                // as a fraction where 0 = first frame, 1 = last frame

