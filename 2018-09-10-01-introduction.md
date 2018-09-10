# FFMPEG Introduction

## Why?

I am a developer of a media company. Although I focus mostly on data, architecture, security, presentation, and usability, I was recently assigned to learn it and
incorporate it in my development. Throughout my development, I have always been handed FFMPEG code in C#. I understand the C# part, but I did not understand at all
the parameters passed to the FFMPEG application (which were long cryptic-looking strings) After all, as a media company, we perform video-related tasks, so now may
be a good time to start diving deeper into FFMPEG.

## What I Know So Far
From the little that I know, FFMPEG is a tool that performs various tasks on existing video and audio files, like concatenation, resizing, extracting audio from 
video, and converting an existing file to a different format (which often means different file extension). There always is an output which is a new audio or video
file, at least in practice because we never want to overwrite any original footage.

## What I Need to Learn
There are some concepts I first need to learn before I go about transforming videos and audio. There are terms like frame rate, bit rate, codec, and audio/video
container. There are procedures like filtering, muxing, extracting, overlaying, and replacing. There may be plugins that I need to find and install that may not 
come with FFMPEG out-of-the-box. I have seen some of these terms as a casual user when I searched for MP3s and videos, but I never really paid attention to them.
The only thing I knew was that some media players could play some file extensions, but some were not able to. It got confusing when one video player would play
an .mp4 file but another woudn't because of codec issues.

## How am I Going to Learn FFMPEG?

Unfortunately, there are no courses that I can watch that would explain to me in great detail what FFMPEG is and what it can do. I know that there are lots of blogs,
as with any technology, that discuss how to do one or a few particular things. I will have to look at a lot of resources and piece things together over time.

Maybe to start, I'll familiarize myself with audio and video file terminology and look up the terms like frame rate, bit rate, and codec. When I'm ready to get into
FFMPEG, I'll search terms like "FFMPEG tutorial beginner", "FFMPEG introduction", "FFMPEG getting started."

For each resource that I find, I will need to try them out, first on a command line. Yes, it will be tedious, but I know that I will build those commands later
in C#.

## FFMPEG Goals

I will start playing with FFMPEG on the command line only. I'll start with some basics:

1. Converting to different file formats. (And no, this isn't merely renaming the file to have a different file extension)
2. Creating a video file of the same codec and file format with a different resolution (lower than the original, though)
3. Creating a video file of the same codec and file format with a different bit rate (for smaller file)
3. Removing audio from a video.
4. Extracting audio from a video and saving it to a new audio file.
5. Extracting a single frame from a video and saving it to a new image file.
6. Extracting a clip from audio or video file with no loss of quality and retain codec and container.

Some more advanced stuff:

1. Writing text to a video