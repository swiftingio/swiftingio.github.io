---
layout: post
author: bartek
title: \#45 Can you sing well?
excerpt: 
---
#### Remark

After reading this post you will find out if you can sing or not. If you want to stay not aware about it please don't read this post further.

#### Story

I'm passionate about music. I have played violin for 3 years. One year ago I bought a digital piano to make first steps as a composer. Also I'm very interested in human voice, especially because I have always wondered: how it is that some people can sing and some can't?

My girlfriend is a professional musician and she have played violin since her childhood. People who are experts in this business have developed their hearing to master level. They can say without any problem if you can sing very well or very badly. If you have never been involved in playing instruments or singing, it can be probably very hard for you to say that someone sings in tune or not.

One day my girlfriend and I were traveling by car (I have to say that I love singing while driving:P)

![](https://raw.githubusercontent.com/swiftingio/blog/%2345-Can-I-Sing/Images/james-corden.jpg)

And after a while she said, Bartek... hmmm, it's not good...

Her comment made me think: even though I have some music experience it's not easy for me to be aware of my singing and I totally don't know if I sing in tune or not. This day when I came back to home I started thinking: hmmm... it would be great to create some mobile app for checking if you can sing. Maybe it would be great to install this application before you decide to go to some voice talent show :P

#### How to test your sing abilities?

I have never been in a music school and that's why I used to wonder how teachers check kids vocal predispositions?

Algorithm steps are the following:

**1)** Teacher plays few sounds on a piano, by pressing single keys.

**2)** Then a student has to sing the sound in the same order by singing syllable (for examle 'la') on each note.

**3)** Teacher hears and says if sounds are correct.

So generally the best way to check if you can sing is just to ask an expert about that. But what if you don't have an expert in your neighborhood, or you are too shy to ask someone? What then? That's why I started to thinking about  a mobile app solution to solve those problems :)

#### Idea

The idea is to create an iOS application available on the AppStore which can tell you if you have predisposition to sing or not. But even if you were not born as the Michael Jackson the application would show you your mistakes and in that way you can become more aware about your voice.

#### Some background

If you are familiar with music theory and you are only interested in technical issues: please go directly to the chapter **iOS Application**. But if you aren't, I have prepared for you a few chapters which will be helpful to understand the way how application works.

##### Note

A note is a sign used in musical notation to represent the relative duration and pitch of a sound (♪, ♫). The pitch of the note means how high or low a note is (pitch is described by frequency in Hz). Below some examples:

- C (261 Hz) [play](https://www.ee.columbia.edu/~dpwe/sounds/instruments/piano-C4.wav?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- G (392 Hz) [play](https://www.ee.columbia.edu/~dpwe/sounds/instruments/piano-G4.wav?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- C (523 Hz) [play](https://www.ee.columbia.edu/~dpwe/sounds/instruments/piano-C5.wav?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

##### Octave and music scales

Octave is an interval between one musical pitch and another with half or double its frequency. For example, if one note has a frequency of 440 Hz, the note one octave above is at 880 Hz. 

[Music scales](https://en.wikipedia.org/wiki/Scale_(music)?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), are typically written using eight notes which represent an array of pitches. The interval between the first and the last note is an octave. Below is an example of C scale:

| C|D|E|F|G|A|H|C|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 16.35 Hz| 18.35 Hz| 20.60 Hz | 21.83 Hz| 24.50 Hz| 27.50 Hz | 30.87 Hz | 32.70 Hz |

Second C in the array starts a new octave with the same note names but with doubled frequencies:

| C|D|E|F|G|A|H|C|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 32.70 Hz| 36.71 Hz | 41.20 Hz | 43.65 Hz| 49.00 Hz| 55.00 Hz | 61.74 Hz | 65.41 Hz |

and so on and so on....

##### How many octaves exist?

We humans can hear sounds from 20 to 20 000 Hz, so there in no sense for us to care about octaves beyond this range. Below you can find examples of 9 next octaves: 

**Octaves**

|Octave|C|D|E|F|G|A|H|C|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|0| 16.35 Hz| 18.35 Hz| 20.60 Hz | 21.83 Hz| 24.50 Hz| 27.50 Hz | 30.87 Hz | 32.70 Hz |
|1| 32.70 Hz| 36.71 Hz | 41.20 Hz | 43.65 Hz| 49.00 Hz| 55.00 Hz | 61.74 Hz | 65.41 Hz |
|2| 65.41 Hz| 73.42 Hz | 82.41 Hz | 87.31 Hz| 98.00 Hz| 110.0 Hz | 123.5 Hz |  130.8 Hz |
|3| 130.8 Hz| 146.8 Hz | 164.8 Hz | 174.6 Hz| 196.0 Hz| 220.0 Hz | 246.9 Hz |  261.6 Hz |
|4| 261.6 Hz| 293.7 Hz | 329.6 Hz | 349.2 Hz| 392.0 Hz| 440.0 Hz | 493.9 Hz |  523.3 Hz |
|5| 523.3 Hz| 587.3 Hz | 659.3 Hz | 698.5 Hz| 784.0 Hz| 880.0 Hz | 987.8 Hz |  1047 Hz |
|6| 1047 Hz| 1175 Hz | 1319 Hz | 1397 Hz| 1568 Hz| 1760 Hz | 1976 Hz |  2093 Hz |
|7| 2093 Hz| 2349 Hz | 2637 Hz | 2794 Hz| 3136 Hz| 3520 Hz | 3951 Hz |  4186 Hz |
|8| 4186 Hz| 4699 Hz | 5274 Hz | 5588 Hz| 6272 Hz| 7040 Hz | 7902 Hz |  8372 Hz |

##### Frequency range of voices and instruments

Vocal range is a measure of the breadth of pitches that a human voice can phonate. For an untrained voice range is between 1.5 – 2 octaves. For a trained voice it's about 2.5 – 3 octaves. Let's look on vocal ranges of famous sing stars:

- Axl Rose: 5 octaves range
- Dawid Bowie: 4 octaves range
- Bob Marley: 3 octaves range

to see more visit [this](https://www.concerthotels.com/worlds-greatest-vocal-ranges?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) website. On the other hand ...  what with instruments? Instruments have generally a defined range:

- piano:  7 ¼ octave range
- violin: more than 4 octaves
- guitar: 4 octaves

You can find more examples [here](http://www.orchestralibrary.com/reftables/rang.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

##### Men vs women 

Voices of women are divided into 3 categories:

- alto (about F3-A5): is the lowest female voice type
- soprano (about A3-C6): is the highest female voice type.
- mezzo-soprano (G3 - A5): in between, is a type of classical female singing voice

**Note:** F3 meaning: **F** represents note and **3** says from which octave the note is. Knowing that and looking at the **Octaves** table we know that the frequency of this sound is 174.6 Hz.

Among men, the broad categories are:

- bass (about E2-F4): is the lowest male voice type and thus a bass voice type sings the lowest notes possible for a human.
- baritone (about G2-A4): is the most common male voice type
- and tenor (about B2-C5): is the highest male voice type you will find in a typical choir

As we see men and women have different range of octave, generally women sing higher than men (more or less one octave). 

Great image to show differences between voices:

![](https://raw.githubusercontent.com/swiftingio/blog/%2345-Can-I-Sing/Images/ranges.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Image is baaed on the source: https://www.becomesingers.com/wp-content/uploads/2013/03/Vocal-Range-Classification.jpg 

##### Childs scale

Childrens voices from approximately 7 years of age can be classified in the same way as female voices: into altos and sopranos.

#### Vocal range is not everything...

Why do I prefer to listen Frank Sinatra than Freddie Mercury?

Of course our voice is described by more than only one parameter such as:

-  vocal weight,
-  vocal registration, 
-  scientific testing, 
-  speech level, 
-  physical characteristics, 
-  vocal transition, 
-  vocal timbre,
-  and vocal tessitura. 

A combination of all those factors is utilized to determine a singer’s voice, and categorize it into a specific type of voice type.

#### iOS Application

##### Inspiration

Have you ever played a [SingStar](https://www.singstar.com/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) game on a PlayStation? I haven't, but I saw on YouTube how people play:

![](https://psmedia.playstation.com/is/image/psmedia/singstar-ultimate-party-screen-02-ps4-eu-31oct14?$MediaCarousel_LargeImage$)

SingStar requires players to sing along with music in order to score points. The game analyzes a player's pitch and timing which is compared to the original track, with players scoring points based on how accurate their singing is.

##### Goal

Main goal of my application is to give easy access to everyone to test their sing abilities. Especially for those who don't have music background. 

#### How application works?

Application compares frequency of a voice wiath a pitch frequency defined in a sing test and then it visualizes the difference between them. Let's look step by step on application flow to understand it better. 

**1.** The first thing is to choose a test from the list:

![](https://raw.githubusercontent.com/swiftingio/blog/%2345-Can-I-Sing/Images/List.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

**2.** After choosing an item from the list application plays the sing test:

![](https://raw.githubusercontent.com/swiftingio/blog/%2345-Can-I-Sing/Images/FirstStep.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Each bar represents a note. The highrer the note is the higher a bar is located.

**3.** Then by clicking the next button a timer gets started to count 3 seconds. It's a time for preparation:

![](https://raw.githubusercontent.com/swiftingio/blog/%2345-Can-I-Sing/Images/WaitStepIPhone.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

**4.** After that time you should start singing. There is no matter what text you sing, you can help yourself with singing syllable 'la' on each note. 

If you sing properly application shows that you are on track:

![](https://github.com/swiftingio/blog/blob/%2345-Can-I-Sing/Images/SecondStepiPhone.png?raw=true?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

If you sing too high the application shows that your voice is upper the right tune:

![](https://github.com/swiftingio/blog/blob/%2345-Can-I-Sing/Images/HighiPhone.png?raw=true?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

When you sing too low, then samples appear below the right tune:

![](https://github.com/swiftingio/blog/blob/%2345-Can-I-Sing/Images/LowiPhone.png?raw=true?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Whole application flow below:

![](https://github.com/swiftingio/blog/blob/%2345-Can-I-Sing/Images/flow.png?raw=true?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### App Architecture

The application is based on two libraries:

##### ResearchKit 

[ResearchKit](http://researchkit.org/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is an open source framework introduced by Apple that allows researchers and developers to create powerful apps for medical research. You can easily create visual consent flows, real-time dynamic active tasks. 

In my case ResearchKit gave me for free an easy interface for supporting application which is based on steps.

##### AudioKit

[AudioKit](http://audiokit.io/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is an audio synthesis, processing, and analysis platform for iOS, macOS, and tvOS. 

This framework gave me great tools for analyzing pitches from device microphone.

##### Project Source

You can download the whole project from [here](https://github.com/swiftingio/SingTest?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and test it by yourself. I will appreciate any comment (bartlomiej.woronin@gmail.com) for this project or any pull request which could develop this application. 

##### What's next ?

My goal is to develop current application and make it available to everyone via AppStore and maybe also someday on Google Play:) 

#### Resources

[Great website about vocal ranges](https://www.becomesingers.com/vocal-range/vocal-range-chart?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[AudioKit example application for microphone analysis](http://audiokit.io/examples/MicrophoneAnalysis/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[AudioKit Library](http://audiokit.io/downloads/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[Pitchy library which provides a simple way to get a music pitch from a frequency](https://github.com/vadymmarkov/Pitchy?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[Accessing heart rate data ResearchKit study](https://www.raywenderlich.com/117433/accessing-heart-rate-data-researchkit-study?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
