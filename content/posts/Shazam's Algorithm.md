---
title: Shazam's Algorithm
date: 2025-01-27
draft: false
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

A sound is just a collection of waves travelling through a medium at a certain frequency and with an amplitude which an be decrypted by the human ears.

There are mainly two forms of audio 
1. Analog
2. Digital

Sound in its natural form is a analog signal, analog audio represents sounds as a continuous electrical signal that mirrors the original sound wave.
Digital audio , is the discrete numerical value converted from analogs audio (audio being converted to bits).

Our traditional recording devices already have systems in place which held the transformation of an analog audio signal into a discrete numerical bits ready to be stored as a mp3 file.

The transformation of audio from analog to digital is not lossless, there can be a loss of sound in the process. But using several techniques like changing the sampling rate and applying Dithering the transformation from analog to digital is almost indistinguishable.



## Fourier Transformation - 


The most important algorithm of all time. FT is used everywhere there is radio waves and signals involved be it radar, sonar, 5G and Wi-Fi  as well.

At the time of its invention the only purpose of the Fast Fourier Transform algorithm was to detect underground atomic explosions, to prevent the soviet from carrying out their atomic testing underground.

A fourier transform is a way of decomposing a signal into pure sine waves that allows 
This transformation enables critical analysis and processing of signals by breaking them into their constituent sinusoidal components.

Audio is recorded in time-domain. The time domain signal represents the change of amplitude over time, using the FFT it is possible to represent any time-domain signal by simply giving the frequencies, amplitudes and phases corresponding to each sinusoid.

!![Image Description](/images/Pasted%20image%2020250305141720.png)



We'll be implementing the FFT in Go,


## Spectrograms -


The FFT algorithm outputs a spectrogram of a particular wave.

A spectrogram is a visual representation of the frequency content of a signal.

	The x-axis repesents time
	 The y-axis represents frequency
	 The color or intensity represents the amplitude of each frequency at a given  time.

Shazam leverages these spectrograms to create a unique fingerprint for each song.

Shazam scans all the local peaks. These are the points where the amplitude of a particular frequency is comparatively high.
Peaks are robust to noise because they represent most dominant and distinctive frequencies in the audio.
Now these peaks are used as a key in a hash table, other columns in the hash table are 
1. Time (the time at which the peak appeared in the audio)
2. Sound Id
3. Artist


Storing the time with the peaks only increases certainty as the recorded sound by the user will not always be from the start. It can be from any timeframe and therefore it is necessary to add time as a factor of search.



## Hashing and Searching - 

For Hashing, Shazam uses something known as combinatorial hashing.
Combinatorial hashing refers to the process of creating a hash (a unique identifier) by combining **multiple features** extracted from a signal. In the context of Shazam it is the time-frequency peaks

Storing these in a Hash Table will give us a o(1) time complexity while searching through the database.

!![Image Description](/images/Pasted%20image%2020250305145400.png)



Database Lookup

- Shazam’s database contains **precomputed hashes** for millions of songs. Each hash is linked to:
    
    A **song ID** (identifying the song).
    A **timestamp** (indicating where the landmark pair occurs in the original track).

- Shazam calculates a **confidence score** based on:
    The **number of matching hashes**.
    The **consistency of the timestamp alignments**.
        
 If the confidence score exceeds a threshold, the song is identified and returned to the user.



## Implementation

1. Converting the input mp3 file into wav.
Why wav? WAV is known for its high resolution, it helps us in retaining the high quality of audio even in large audio files.


`func mp3ToWav(file multipart.File) ([]byte, error) {`  
    `decoder, err := mp3.NewDecoder(file)`  
    `if err != nil {`  
       `return nil, fmt.Errorf("failed to decode MP3: %v", err)`  
    `}`  
    `var wavBuffer bytes.Buffer`  
    `enc := wav.NewEncoder(&wavBuffer, decoder.SampleRate(), 16, 1, 1)`  
  
    `buf := make([]byte, 1024)`  
    `intBuffer := &audio.IntBuffer{`  
       `Format: &audio.Format{`  
          `NumChannels: 1,`  
          `SampleRate:  decoder.SampleRate(),`  
       `},`  
       `Data: make([]int, 0, 1024),`   
    `}`  
  
    `for {`  
       `n, err := decoder.Read(buf)`  
       `if err == io.EOF {`  
          `break`  
       `}`  
       `if err != nil {`  
          `return nil, fmt.Errorf("failed to read MP3 data: %v", err)`  
       `}`  
  
       `for i := 0; i < n; i += 2 {`  
          `sample := int16(buf[i]) | int16(buf[i+1])<<8`  
          `intBuffer.Data = append(intBuffer.Data, int(sample))`  
       `}`  
       `if err := enc.Write(intBuffer); err != nil {`  
          `return nil, fmt.Errorf("failed to write WAV data: %v", err)`  
       `}`  
       `intBuffer.Data = intBuffer.Data[:0]`  
    `}`  
  
    `if err := enc.Close(); err != nil {`  
       `return nil, fmt.Errorf("failed to close WAV encoder: %v", err)`  
    `}`  
  
    `return wavBuffer.Bytes(), nil`  
`}`


2. Next step after generating our wav file will be creating a spectogram out of it, it'll be done using the STFT (Short-Time Fourier Transform). _STFT_ is an extension of FFT that computes the Fourier Transform of short, overlapping segments of a signal over time.


   