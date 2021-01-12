---
layout: post
author: bartek
title: \#19 Exploring music basics with AudioKit
excerpt: 
---
Have you ever had a chance to use some audio frameworks in your mobile application project in purpose of signal processing, audio streaming or spectrum analyzing? I have to say that I haven't... but I'm quite interested in music. This is the reason why I wanted to explore audio functionality on iOS devices and make first step with digital audio. During exploring the internet I found some interesting library: [AudioKit](http://audiokit.io/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), it seemed to me quite powerful and advanced, that's why I wanted to take a look on it.

Have you ever had a chance to use audio frameworks in your mobile application project for the purpose of signal processing, audio streaming or spectrum analysis? I have to say that I haven't... but I'm quite interested in music. This is the reason why I wanted to explore audio functionality on iOS devices and make first step with digital audio. During my exploration of the Internet I have found an interesting library: [AudioKit](http://audiokit.io/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). It seemed quite powerful and advanced, that's why I wanted to take a look on it.

#### What is AudioKit and how to start learning? 

AudioKit is an audio synthesis, processing, and analysis platform for OS X, iOS, and tvOS that offers:

* create sounds with many synthesizers, physical models, and sample playback operations, and then process those sounds with effects, filters, delays, reverbs, and more

* 100+ Playgrounds serve as the new interactive tutorials

* built-in functionality for plotting waveforms, amplitudes and spectra

* and what is the most important: you can create everything using Swift!

How to start learning? My suggestion is to just download playgrounds from [here](http://audiokit.io/downloads/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and explore examples:

![](https://raw.githubusercontent.com/swiftingio/blog/%2319-AudioKit/Images/audioKitPlaygrounds.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 

In this post I would like to present, with help of AudioKit, much easier stuff, for music beginners like me:)

#### My story

Two and a half years ago I started playing the violin from scratch. For me there was a lot of new things related to music such as:

* Metronome
* Tuner
* Notes
* Tempo
* Scale
* Octave
* and much more...

Now I can say that more or less I know these terms. But what I would like to do in the near future is to learn more about audio processing in digital perspective. It would be great if I could connect my two passions - programming and music!:)

Today, by making first step in connecting these two, let's start with some basic terms in sound synthesis area.

##### Oscillator

Sound synthesis is the electronic production of sounds—starting from their basic properties, such as sine tones and other simple waves.
In a synthesizer, the task of pitch generation falls to a component known as an oscillator. Going further... the [pitch](http://music.stackexchange.com/questions/3262/what-are-the-differences-between-tone-note-and-pitch?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is a musical sound that has a steady and measurable frequency.

AudioKit creates Oscillator using few lines:

```
var oscillator = AKOscillator()
oscillator.amplitude = 0.1
oscillator.frequency = 440.0
AudioKit.output = oscillator
AudioKit.start()
```

Demo:
<iframe src="https://player.vimeo.com/video/171498298" width="640" height="340" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

##### FFT (Fast Fourier Transform)

Digital Music couldn't exist without the Fourier Transform.
What this transformation does is just converting our signal from the time domain to the frequency domain:

![](https://raw.githubusercontent.com/swiftingio/blog/%2319-AudioKit/Images/FFT.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

*Source: [toptal.com](https://www.toptal.com/algorithms/shazam-it-music-processing-fingerprinting-and-recognition?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)*

How AudioKit deals with it?

```
var oscillator = AKOscillator()
oscillator.amplitude = 0.2
oscillator.frequency = 440
var mixer = AKMixer(oscillator)

AudioKit.output = mixer
AudioKit.start()
oscillator.start()


let plot = AKNodeFFTPlot(mixer, frame: CGRect(x: 0, y: 0, width: 500, height: 500))
plot.shouldFill = true
plot.shouldMirror = false
plot.shouldCenterYAxis = false
plot.color = UIColor.purpleColor()
```

And the result plot is:

![](https://raw.githubusercontent.com/swiftingio/blog/%2319-AudioKit/Images/FFT%20Plot.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

By the way: Have you ever thought how [Shazam](http://www.shazam.com/apps?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) app works? ...Yes, FFT is the answer! if you want to know more I recommend visiting this [site](https://www.toptal.com/algorithms/shazam-it-music-processing-fingerprinting-and-recognition?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

##### Notes

Notes represent the relative duration and pitch of a sound (♪, ♫):

![](https://raw.githubusercontent.com/swiftingio/blog/%2319-AudioKit/Images/Notes-Example.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
*Source: [musicnotes](http://www.musicnotes.com/blog/2014/04/11/how-to-read-sheet-music/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)*

Do you want to know more? Visit [this](http://www.musicnotes.com/blog/2014/04/11/how-to-read-sheet-music/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) page.

##### Octave

An octave is an interval between one musical pitch and another that doubles its frequency. Thus the international standard pitch A vibrates at 440 Hz, the octave above this A vibrates at 880 Hz, while the octave below it vibrates at 220 Hz.

##### Music Scale 

Musical scales are typically written using eight notes, and the interval between the first and last notes is an octave. For example, the C Major scale is typically written C D E F G A B C.

![](https://raw.githubusercontent.com/swiftingio/blog/%2319-AudioKit/Images/c_major.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Let's play C major scale using AudiKit:

```
let sampler = AKSampler()
sampler.loadWav("Sounds/fmpia1")
let ampedSampler = AKBooster(sampler, gain: 3.0)
var delay  = AKDelay(ampedSampler)
delay.time = pulse * 1.5
delay.dryWetMix = 0.0
delay.feedback = 0.0

let cMajor = [72, 74, 76, 77, 79, 81, 83, 84]

var mix = AKMixer(delay)
var reverb = AKReverb(mix)

AudioKit.output = reverb
AudioKit.start()

for note in cMajor {
    sampler.playNote(note)
    sleep(1)
}
```

##### MIDI (Musical Instrument Digital Interface)

What are strange numbers in previous section? MIDI will explain everything:

MIDI is a protocol designed for recording and playing back music on digital synthesizers. Ok, so how to translate MIDI format into real notes (C, D, E...?), let's look on below table: 

![](https://raw.githubusercontent.com/swiftingio/blog/%2319-AudioKit/Images/Midi.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
*Source: [midimountain](http://www.midimountain.com/midi/midi_note_numbers.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)*

##### Tempo

Tempo tells you how fast or slow a piece is intended to be played, and often is shown at the top of a piece of music sheet. A tempo of, say 60 BPM (beats per minute) would mean you’d play 60 of the signified notes every minute or a single note every second. 

##### Tuner 

Tuning is the process of adjusting the pitch of one or many tones from musical instruments until they form a desired arrangement.  

##### Metronome
 
A metronome is a practice tool that produces a steady pulse (or beat) to help musicians play rhythms accurately (settable in beats per minute: bpm). AudioKit has very easy interface to create this kind of things, let's look on code below:

```
var currentFrequency = 60.0
let beep = AKOperation.sineWave(frequency: 480)

let trig = AKOperation.metronome(AKOperation.parameters(0) / 60)

let beeps = beep.triggeredWithEnvelope(
    trig,
    attack: 0.01, hold: 0, release: 0.05)

let generator = AKOperationGenerator(operation: beeps)
generator.parameters = [currentFrequency]

AudioKit.output = generator
AudioKit.start()
generator.start()
```
In that way we just created metronome with 60 bpm.

**Demo:**
<iframe src="https://player.vimeo.com/video/171503295" width="640" height="640" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
#### Effects

AudioKit offers a lot of regarding audio analysis, but I would like to show you, just for fun, a few examples of library usage:

##### Reverb

Reverb, is created when a sound or signal is reflected causing a large number of reflections to build up and then decay as the sound is absorbed by the surfaces of objects in the space. It’s just a few lines with AudioKit to create reverb effect programmatically:

```
let bundle = NSBundle.mainBundle()
let file = bundle.pathForResource("drumloop", ofType: "wav")
var player = AKAudioPlayer(file!)
player.looping = true
var reverb = AKReverb(player)

//Load factory preset and give the dry/wet mix amount here
//Dry (value: 0) simply means without any effects. Wet (value: 1) means with effects
reverb.dryWetMix = 0.5

AudioKit.output = reverb
AudioKit.start()

player.play()
reverb.loadFactoryPreset(.LargeHall)
```

Demo:
<iframe src="https://player.vimeo.com/video/171503422" width="640" height="368" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

##### Mixing

In its simplest form an audio mixer combines several incoming signals into a single output signal:

```
//: This section prepares the players
let bundle = NSBundle.mainBundle()
let drumFile   = bundle.pathForResource("drumloop", ofType: "wav")!
let bassFile   = bundle.pathForResource("bassloop", ofType: "wav")!
let guitarFile = bundle.pathForResource("guitarloop", ofType: "wav")!
let leadFile   = bundle.pathForResource("leadloop", ofType: "wav")!

var drums  = AKAudioPlayer(drumFile)
var bass   = AKAudioPlayer(bassFile)
var guitar = AKAudioPlayer(guitarFile)
var lead   = AKAudioPlayer(leadFile)

drums.looping  = true
bass.looping   = true
guitar.looping = true
lead.looping   = true

//: Any number of inputs can be summed into one output
let mixer = AKMixer(drums, bass, guitar, lead)
```

Demo:
<iframe src="https://player.vimeo.com/video/171503295" width="640" height="640" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

#### Summarizing

I hope to read your comments about your music experience on iOS platform! From my side: I'm still learning..., so for sure in the nearly future I will share with you my new experiences from music-programming world!:)


#### Resources

* [Logic Studio Documentation](https://documentation.apple.com/en/logicstudio/instruments/index.html#chapter=A%26section=2%26tasks=true?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [MIDI notes numbers](http://www.midimountain.com/midi/midi_note_numbers.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Tone, note and pitch definitions](http://music.stackexchange.com/questions/3262/what-are-the-differences-between-tone-note-and-pitch?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Time-Frequency Analysis of Musical Instruments](http://epubs.siam.org/doi/pdf/10.1137/S00361445003822?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Chrome experiments with Oscillators - nice visualizations explaining of how ocillators work](https://musiclab.chromeexperiments.com/Oscillators?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Pitch in music definition lesson](http://study.com/academy/lesson/what-is-pitch-in-music-definition-lesson-quiz.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
