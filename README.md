# Music Recommendation System 

## Overview

![image](https://user-images.githubusercontent.com/89123268/202051443-6bc35412-03af-4bdf-b09e-82d9c2a0a156.png)

- First we start with our piece of audio/song
- Then, we want to pass it into the embedding generator; this takes our audio then converts it into a byte vector
- The vector is then passed into a recommendation engine
- The recommendation engine will then reccomend the closest values to the original piece of audio/song

**In order to build the above components, there are 3 major resources used:**
- audioset dataset: this includes YouTube videos, mp3s, mp4s, etc. 
- MAX audio embedding generator (from IBM): tool that helps convert raw .WAV files into a byte strings/vectors
- ANNOY: an approximate nearest neighbors implementation by Erik Bernhardsson, who works at Spotify; it takes the given vector then spits out neighboring vectors which can then be tranlated back into music (recommended music based on the input)

## AudioSet
![image](https://user-images.githubusercontent.com/89123268/202052181-cc1138f3-4770-40d7-98c9-50edfc999e1b.png)

- set of YouTube video IDs
- huge file, consisting of json like information
- this entire json looking structure is just one sample video out of thousands in the entire collection (12,228 in total)
- the first "video_id" is basically a YouTube URL ID
- "start_time_seconds" and "end_time_seconds" are the 2 time stamps (6-16 seconds); the 10 second chunk is the frame that is used
- "labels" are tags that are given to the video (is this muisc, guitar, someone talking, dog barking, etc.); these labels are stored in a list
- feature lists: every part of that audio is now encoded into a 128 bit feature string
- for example, we have a 10 second piece of audio; we are chunking that 10 second piece of audio into 10 chunks of 1 second each and each of those seconds are encoded into an array of 128 numbers (also known as a byte array) -> this is repeated for each second of audio so we will have 10 lists at the end
- **This is our entire dataset -> This is repeated 12,228 times (2.4 GB)**
- all of these files are stored in the form of tf records -> each of these records have exactly all of this information for 12,228 examples
- when you look at raw music data, 
