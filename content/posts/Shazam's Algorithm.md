---
title: Shazam's Algorithm
date: 2025-01-27
draft: true
tags:
---

Shazam, the music finding app. 

Hearing the name and utility it provides. The first thought anyone would've in their mind about the underlying tech would be that they have to be using some kind of CNN (convolutional neural network) model in their machine learning algorithm. 

But that's not the reality Shazam AI is not a reality. So what algorithm are they actually using and why this is so fast in searching from a billion songs out there in the world.

Their algorithm consists for various audio transformations and utilising theories from the past. And simply playing around with the frequencies of the audio.
These things just made me curious and can't help myself but to deeply understand each aspect of Shazam's algorithm to understand Why it works the way it work. And how FFT ( Fast Fourier Transformation ) utilised in the process. So, as I do my research and learn new things on the go, its better that I document everything, being beneficial for both, staying in my mind for a longer time and helping others who also want to understand the algorithm.

Also after figuring out the technical details of the algorithm I will be developing my own version of it in Go Lang.


Overview of the most important components/processes of the algorithm.

1. Audio Sampling
2. FT (Fourier Transform)
3. Spectograms
4. Hashing and Searching


!![Image Description](/images/Pasted%20image%2020250303162835.png)



# Audio Sampling - 

What is a sound?

A sound is just a collection of waves travelling through a medium at a certain frequency and with an amplitude which an be decrypted by the human ears


















