# Music Recommendation System 

## Overview

![image](https://user-images.githubusercontent.com/89123268/202051443-6bc35412-03af-4bdf-b09e-82d9c2a0a156.png)

- First we start with our piece of audio/song
- Then, we want to pass it into the embedding generator; this takes our audio then converts it into a byte vector
- The vector is then passed into a recommendation engine
- The recommendation engine will then reccomend the closest values to the original piece of audio/song

*** in order to build the above components, there are 3 major resources used:
- audioset dataset: this includes YouTube videos, mp3s, mp4s, etc. 
- MAX audio embedding generator (from IBM): tool that helps convert raw .WAV files into a byte strings/vectors
- ANNOY: an approximate nearest neighbors implementation by Erik Bernhardsson, who works at Spotify; it takes the given vector then spits out neighboring vectors which can then be tranlated back into music (recommended music based on the input)

## AudioSet
![image](https://user-images.githubusercontent.com/89123268/202052181-cc1138f3-4770-40d7-98c9-50edfc999e1b.png)

- set of YouTube video IDs
- huge file, consisting of json like information
