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
- ANNOY: an approximate nearest neighbors implementated by Erik Bernhardsson, who worked at Spotify; it takes the given vector then spits out neighboring vectors which can then be tranlated back into music (recommended music based on the input)

## AudioSet Dataset
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
- when you look at raw WAV music data, they're typically stored in kilohertz... for example a CD has 44.1 kilohertz which is equivalent to 44.1 thousand numbers being used to represent one second of data -> this is a lot of information -> that's why the output of 128 features is a lot less and also represents a lot more information -> this is the embedding -> the embeddings are pre-genertaed by the MAX audio embedding generator

## Generate EMbedding from WAV
- This reads a wav file and gets the corresponding embeddings
- We're going to fetch all of the wav files in demo assets (all the stuff in this repository; all of these wav files)
- The goal here is to convert these wav files into the embeddings using the max audio embedding generator
- So for every single WAV file, we're going to execute this curl request -> I'm passing the audio wav file to the URL (right here) -> its then going to run the prediction model through our embedding and when it generates an embedding, it's going to output it to a json file
- Let's say we extract the embedding which is a list of lists where each list within that list is 128 characters of bytes -> we want the audio length to be cut at 10 seconds -> we want to make sure that every audio clip is the same size -> in this case, each list of lists will contain 10 lists and each of those 10 lists will be 128 characters for bytes
- We then print out every call and encode every one of those wav files printed above into an embedding

- now, since its a list of lists, each of those lists is 10x128 for every single sample -> We then flatten it so that we have a single vector of 1280 dimensions that represents a WAV file

- Now that its a much more compressed representation, we can find the cosign similarity between 2 sounds -> if the cosign similarity is higher, that means the 2 vectors/sounds are more "similar"
- lets test out a few values:
- #birds2 vs birds1 = .906
- #jazz guitar vs guitar = .893
- both have high cosign scores which makes sense because they are different types of the same thing

- now, lets print all of these into a cosign similarity matrix:
![image](https://user-images.githubusercontent.com/89123268/202065065-43a5bcde-d6b4-409a-a0d2-a77a81ab837d.png)

- as we can see, the jazz guitar and regular guitar have a cosign similarity of .89 which is super high
- on the other hand, guitar and train have a cosign similarity of .69 which is really low
