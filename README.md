# Music Recommendation System 

## Overview

![image](https://user-images.githubusercontent.com/89123268/202051443-6bc35412-03af-4bdf-b09e-82d9c2a0a156.png)

- First we start with our audioset dataset: this includes YouTube videos, mp3s, mp4s, etc. 
- Then, we want to pass it into the MAX audio embedding generator (from IBM): tool that helps convert raw .WAV files into a byte strings/vectors
- The vector is then passed into a recommendation engine: ANNOY: an approximate nearest neighbors implementated by Erik Bernhardsson, who worked at Spotify; it takes the given vector then spits out neighboring vectors which can then be tranlated back into music (recommended music based on the input)
- The recommendation engine will then reccomend the closest values to the original piece of audio/song

## AudioSet Dataset
![image](https://user-images.githubusercontent.com/89123268/202052181-cc1138f3-4770-40d7-98c9-50edfc999e1b.png)

- this entire json looking structure is just one sample video out of thousands in a dataset
- the first "video_id" is basically a YouTube URL ID
- "start_time_seconds" and "end_time_seconds" are the 2 time stamps (6-16 seconds); the 10 second chunk is the frame that is used
- "labels" are tags that are given to the video (ex: muisc, guitar, talking, dog barking, etc.); these labels are stored in a list
- feature lists: every part of that audio is now encoded into a 128 bit feature string
- for example, we have a 10 second piece of audio; we are chunking that 10 second piece of audio into 10 chunks of 1 second each and each of those seconds are encoded into an array of 128 numbers (also known as a byte array) -> this is repeated for each second of audio so we will have 10 lists at the end
- **This is our entire dataset -> This is repeated 12,228 times (2.4 GB)**
- all of these files are stored in the form of tf records -> each of these records have exactly all of this information for 12,228 examples
- when you look at raw WAV music data, they're typically stored in kilohertz... for example a CD has 44.1 kilohertz which is equivalent to 44.1 thousand numbers being used to represent one second of data -> this is a lot of information -> that's why the output of 128 features is a lot less and also represents a lot more information -> this is the embedding -> the embeddings are pre-genertaed by the MAX audio embedding generator

## Generate EMbedding from WAV
- This is going to read a wav file and get the corresponding embeddings
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


- we are now able to have a set of these byte vectors which we can pass into the recommendation engine (ANNOY)

## Recommendation Engine (ANNOY)
- before getting into the next notebook
- ANNOY uses approximate nearest neighbors for recommendations -> much faster than trying to do a brute force linear scan
- This slide deck was made by the creator of ANNOY, Erik Bernhardsson:


![image](https://user-images.githubusercontent.com/89123268/202068571-592edd51-7d27-4f0c-90c2-9837dfda3464.png)
- lets say that each x in the embedding space is a song

![image](https://user-images.githubusercontent.com/89123268/202068961-11dd1a77-a817-4ec9-9e75-8ed71b90198a.png)
- now lets pick any random 2 points in our embedding then draw a line between them and then split that line with a perpendicular line
- now we have 2 separate halves

![image](https://user-images.githubusercontent.com/89123268/202069523-9c58c96e-bb11-4a13-9aa1-d98eaea02ad3.png)
- next, lets choose 2 points above the perpendicular line and 2 point below the perpendicular line then split both of them again with perpendicular lines

- lets do this again and again:
![image](https://user-images.githubusercontent.com/89123268/202069723-489a5af0-9372-40de-8254-39fbb9ac2049.png)

- Until we end up with something that looks like this:
![image](https://user-images.githubusercontent.com/89123268/202069805-716128f9-d297-42bc-82b9-6104d81e8d05.png)

- this is a partition of an embedding space that you can also format as a binary decision tree, as shown below:
![image](https://user-images.githubusercontent.com/89123268/202073125-34704020-7803-40ef-bb39-89eb29ebf0f7.png)

- the top point would be the start where all of your poins are in one embedding space then every node is where the space was split
- we keep doing this until we end up at the circular leaf nodes which correspond to blocked off sections in the previous graph

- now, lets say that you want find the nearest neighbors of a particular point... so what do you do?
- lets say that the actual point in the embedding space lies somewhere here (in the white "x"):
![image](https://user-images.githubusercontent.com/89123268/202073423-9ae54fb4-9a37-4c16-a22d-c5779a9cbeb4.png)

- so, should we just traverse the tree which corresponds to that particular region/leaf in the decision tree?
![image](https://user-images.githubusercontent.com/89123268/202073651-27605448-2eba-4923-8150-efe054d6b8ba.png)

- Yes and no

![image](https://user-images.githubusercontent.com/89123268/202073941-ddf8a038-ed62-4de5-bf1b-bb686914015b.png)

- As we can see, there are "xs" very close to the x we chose, however, since they aren't in the same region as the x we chose, they will not be recommended
- this all happened arbitrarily because of a random split -> obviously, this isn't good, so how can we fix this? -> by traversing both sides of the tree:

![image](https://user-images.githubusercontent.com/89123268/202074240-2bfbc062-96ee-43e5-8b3e-6cdbae0cb7ea.png)

- now we have 5 other regions that could potentially be our nearest neighbors -> so if we highlight these 5 regions in the actual space, it looks like this:

![image](https://user-images.githubusercontent.com/89123268/202074682-03f21827-199b-41b8-97ae-c2ce44e875a7.png)

- so this tells us that if we input a song (that is at the red "x"), it'll recommend all the points in the 5 highlighted regions
- the problem now, is that the recommendations are formed by just one tree 

![image](https://user-images.githubusercontent.com/89123268/202076341-2eb8ede2-84d0-487f-aaca-bc29de838aa3.png)
- for example, all 3 of these could be the trees that we get at the end in 3 separate cases... however, its always better to have multiple trees so that, by random chance, we won't be missing certain points
- the more trees we build and take into consideration when making the decision for determining nearest neighbors, it becomes much more concrete
- For example, lets say that the red "x" is the song that we put in -> we get the set of points in the purple region for nearest neighbors
- then with these same set of points, we're going to run ANNOY again but this time we get random splits going in different directions then end up with the set of points in the yellow region
- Lastly, we do it a third time and yet again, we get different splits and end up with the set of points in the red region
- so, lets say that we wanted to recommend 10 songs for a single song that a user inputs -> in our example, that would mean that there are 10 points/songs in each of the purple, yellow and red regions, giving us 30 total points/songs... however, some of the points/songs are overlapping -> so, what do we do?

![image](https://user-images.githubusercontent.com/89123268/202077958-ab255c80-0436-40df-833d-58f9108783ce.png)
- Lets say that out of the 30 items/songs, 12 of them are duplicates -> now, we take the union and remove these duplicates -> we now only have 18 items/songs
- now since there are only 18 points, we can actually compute raw distances for every single one of the 18 points from the query vector (aka, the song we want recommendations for)
- once we do this, we can sort them in a ranked order of nearest neighbors/nearest distances:

![image](https://user-images.githubusercontent.com/89123268/202078586-df6fc1f6-adf3-497e-9624-111492ab0096.png)

- we then we get the union (18 total songs) of all leaves:
![image](https://user-images.githubusercontent.com/89123268/202079382-1324b485-b7f5-40ff-9a0a-8ce1f1c79761.png)

- we then find the nearest neighbors for all 18 points from the query point (inputted song) -> we can then rank them to find the nearest 10 neighbors, eliminating the 8 furthest



