# Music Recommendation System 
### By Jacob Posz

## Problem
Lets say we want to build a recommendation system for music based on a user's input... How do we do this? Where would we even start? Luckily, Erik Bernhardsson, a former Spotify employee, has already implemented a music recommendation system that uses an approximate nearest neighbors recommendation engine called ANNOY.

## Overview

![image](https://user-images.githubusercontent.com/89123268/202051443-6bc35412-03af-4bdf-b09e-82d9c2a0a156.png)

- First we start with our audio dataset: which includes YouTube videos specifically 
- Then, we want to pass it into the MAX audio embedding generator: tool that helps convert raw .WAV files into byte strings/vectors
- The vector is then passed into a recommendation engine: also know as ANNOY: an approximate nearest neighbors method implementated by Erik Bernhardsson, who worked at Spotify; it takes the given vector then spits out neighboring vectors which can then be translated back into music
- The recommendation engine will then reccomend the closest values to the original piece of audio/song

## AudioSet Dataset
![image](https://user-images.githubusercontent.com/89123268/202083469-dd7ba0ad-3f31-4c5f-9c2c-086780705f61.png)

![image](https://user-images.githubusercontent.com/89123268/202052181-cc1138f3-4770-40d7-98c9-50edfc999e1b.png)

- this entire json looking structure is just one sample video out of thousands in a dataset
- the "video_id" is basically a YouTube URL ID
- "start_time_seconds" and "end_time_seconds" are the 2 time stamps (6-16 seconds); this 10 second chunk is what will be used
- "labels" are tags that are given to the video (ex: muisc, guitar, talking, dog barking, etc.); these labels are stored in a list
- "feature lists": lists that every part of that audio is encoded into (they're all 128 bit feature strings)
- for example, if we have a 10 second piece of audio; we are chunking that 10 second piece of audio into 10 chunks of 1 second each and each of those seconds are encoded into an array of 128 numbers (also known as a byte array) -> this is repeated for each second of audio so we will have 10 lists at the end

## MAX Audio Embedding Generator
![image](https://user-images.githubusercontent.com/89123268/202083572-379baed0-dd80-4fd2-b7e8-f325092be46a.png)

- This is going to read a wav file and get the corresponding embeddings
- When it generates an embedding, it's going to take the wav file and output it to a json file
- Let's say we extract the embedding -> we want the audio length to be cut at 10 seconds -> we want to make sure that every audio clip is the same size -> in this case, each embedding will be 10x128 
- We would then want to flatten it so that we have a single vector of 1280 dimensions that represents a single WAV file
- Now that its a much more compressed representation, we can find the cosign similarity between 2 sounds -> if the cosign similarity is higher, that means the 2 vectors/sounds are more "similar"
- 2 examples given in the code were:
birds2 vs birds1 = .906
jazz guitar vs guitar = .893
- both have high cosign scores which makes sense because they are different types of the same thing

- Below is a sample cosign similarity matrix:
![image](https://user-images.githubusercontent.com/89123268/202065065-43a5bcde-d6b4-409a-a0d2-a77a81ab837d.png)

- as we can see, the jazz guitar and regular guitar have a cosign similarity of .89 which is super high
- on the other hand, guitar and train have a cosign similarity of .69 which is really low


- we are now able to have a set of these byte vectors which we can pass into the recommendation engine (ANNOY)

## Recommendation Engine (ANNOY)
- ANNOY uses approximate nearest neighbors for recommendations -> much faster than trying to do a brute force linear scan
- This slide deck was made by the creator of ANNOY, Erik Bernhardsson:


![image](https://user-images.githubusercontent.com/89123268/202068571-592edd51-7d27-4f0c-90c2-9837dfda3464.png)
- lets say that each x in the embedding space is a song

![image](https://user-images.githubusercontent.com/89123268/202068961-11dd1a77-a817-4ec9-9e75-8ed71b90198a.png)
- now lets pick any random 2 songs in our embedding then draw a line between them and then split that line with a perpendicular line
- now we have 2 separate halves

![image](https://user-images.githubusercontent.com/89123268/202069523-9c58c96e-bb11-4a13-9aa1-d98eaea02ad3.png)
- next, lets choose 2 songs above the perpendicular line and 2 songs below the perpendicular line then split both of them again with perpendicular lines

- lets do this again and again:
![image](https://user-images.githubusercontent.com/89123268/202069723-489a5af0-9372-40de-8254-39fbb9ac2049.png)

- Until we end up with something that looks like this:
![image](https://user-images.githubusercontent.com/89123268/202069805-716128f9-d297-42bc-82b9-6104d81e8d05.png)

- this is a partition of an embedding space that you can also format as a binary decision tree, as shown below:
![image](https://user-images.githubusercontent.com/89123268/202073125-34704020-7803-40ef-bb39-89eb29ebf0f7.png)

- the top point would be the start where all of the songs are in one embedding space then every node is where the space was split
- we keep doing this until we end up at the circular leaf nodes which correspond to blocked off sections in the previous graph

- now, lets say that you want find the nearest neighbors of a particular song... so what do you do?
- lets say that the actual song in the embedding space lies somewhere here (in the white "x"):
![image](https://user-images.githubusercontent.com/89123268/202073423-9ae54fb4-9a37-4c16-a22d-c5779a9cbeb4.png)

- so, should we just traverse the tree which corresponds to that particular region/leaf in the decision tree?
![image](https://user-images.githubusercontent.com/89123268/202073651-27605448-2eba-4923-8150-efe054d6b8ba.png)

- **Yes and no**

![image](https://user-images.githubusercontent.com/89123268/202073941-ddf8a038-ed62-4de5-bf1b-bb686914015b.png)

- As we can see, there are "xs" very close to the x we chose, however, since they aren't in the same region as the x we chose, they will not be recommended
- this all happened arbitrarily because of a random split -> obviously, this isn't good, so how can we fix this? -> by traversing both sides of the tree:

![image](https://user-images.githubusercontent.com/89123268/202074240-2bfbc062-96ee-43e5-8b3e-6cdbae0cb7ea.png)

- now we have 5 other regions that could potentially be our nearest neighbors -> so if we highlight these 5 regions in the actual space, it'll looks like this:

![image](https://user-images.githubusercontent.com/89123268/202074682-03f21827-199b-41b8-97ae-c2ce44e875a7.png)

- so this tells us that if we input a song (that is at the red "x"), it'll recommend all of the songs in the 5 highlighted regions
- the problem now, is that the song recommendations are formed by just one tree 

![image](https://user-images.githubusercontent.com/89123268/202076341-2eb8ede2-84d0-487f-aaca-bc29de838aa3.png)
- for example, all 3 of these could be the trees that we get at the end in 3 separate iterations... however, its always better to have multiple trees, so that, by random chance, we won't be missing certain songs
- the more trees we build and take into consideration when making the decision for determining nearest neighbors, it becomes much more concrete
- For example, lets say that the red "x" is the song that we input -> we get the set of songs in the purple region for nearest neighbors
- then using the same inputted song in a different iteration, we're going to run ANNOY again but this time we get random splits going in different directions -> we end up with the set of points in the yellow region
- Lastly, we do it a third time and yet again, we get different splits and end up with the set of points in the red region
- so, lets say that we wanted to recommend 10 songs for a single song that a user inputs -> in our example, that would mean that there are 10 songs in each of the purple, yellow and red regions, respectively, giving us 30 songs total... however, some of the songs are overlapping -> so, what do we do?

![image](https://user-images.githubusercontent.com/89123268/202080143-cbdc1dab-ec19-4f6d-9b4c-45bab177d592.png)
- Lets say that out of the 30 songs, 12 of them are duplicates -> now, we take the union and remove these duplicates -> we now only have 18 remaining
- now since there are only 18 songs, we can actually compute raw distances for every single one of the 18 points from the query vector (aka, the song we want recommendations for)
- once we do this, we can sort them in a ranked order of nearest neighbors/nearest distances:
- we then find the nearest neighbors for all 18 points from the query point/inputted song -> we then can eliminate the 8 furthest, giving us the closest 10 songs:

![image](https://user-images.githubusercontent.com/89123268/202080466-b605f10e-ea40-4dbd-a117-83274367e51e.png)

## Building a Recommender with ANNOY (Code Demonstration Video)
- the objective is to get the top 10 recommendations for a piece of music
- to run this notebook, we need the music_set.json file from the AudioSet processing.ipynb notebook 
- first, we have to install 'annoy'
- We then need to read the class labels from the csv file containing all of the categories and classes
- Once the json file is read, we take the length of the dataset (2000 examples); we have 2000 pieces of music to play with
- We want to create an ANNOY index where we now need to pass in the number of dimensions of each of our samples (1280) -> then for every single piece of music, we want to add it to the index (the samples we want to use to find the nearest neighbors of)




## Critical Analysis
### What is the impact of this project?

### What does it reveal or suggest

### What is the next step?








## Video Recording

## Resource Links




