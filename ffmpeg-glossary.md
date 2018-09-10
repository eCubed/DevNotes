# FFMPEG Glossary

**base frame rate**. 24 or 30 frames per second, typically used for shooting typical videos.

**bit rate.** How much data it takes for the CPU to play 1 second worth of a clip. Higher values mean greater detail, though larger file, thus computationally
more demanding to play back. For some videos or audio(because they made it that way), the bit rate may vary depending on how demanding the scene is. For example,
a clip where there isn't much movement or detail will be at a lower bit rate than the one that has lots of detail and movement (such as sports).

**buffering.** downloading or putting to RAM the first few mbytes or kbytes of a video or audio first before playback, and simultaneously downloading/loading to RAM
the next few mbytes or kbytes while playing the current clip. This avoids having to wait for the entire video to be downloaded or loaded to memory before it can be
played back.

**codec.** Stands for coder/decoder. It is an algorithm pair that translates light (video and images) or sound (audio) into bytes that can be stored,
and the other pair translates the bytes to their original visual and audio forms that can be seen on screen and heard from speakers. Each codec has a different
approach on determining what data can be discarded to store and reproduce video, image, and/or audio that is "good enough". Some throw more data away than others
resulting in varying audio and video qualities at the end. The encoding part is typically handled by the capture device (camera and/or microphone) and video editing
software when it's time to save the video project, and the decoding is handled by playback devices (including software like VLC or Media Player).

**compression.** The process of removing repetitive sections of a video or audio clip to reduce the file size and to minimize loss of video or audio quality.

**container.** Pretty much the file extension of a final video or audio file. More importantly, it contains some information about the file such as metadata
(like location, date shot, etc.) and which codec was used to encode the video and/or audio, so that when it is played back, the playback software will know which
codec to use.

**demux.** To break down a video file into separate audio and video files.

**frame rate.** How many successive images (frames) will play in succession over one second. The lower the frame rate, the choppier the video. The higher the
frame rate, the smoother the video. Since lower frame rate will have less frames, it results in smaller storage space requirement. Higher frame rate means more
frames for the same duration of a footage, which results in larger storage space requirement. Lower frame rates are better for something we'll intentionally
speed up like time lapse videos. Higher frame rates are better suited for something we want to show a slow motion of.

**interlaced.** Broadcast/playback of a video file where only the alternating lines of an frame is sent out, and the subsequent frame's other alternating lines are
sent, and so on. This reduces the bandwidth in half, and the video quality may or may not suffer depending on what is being shot. Interlacing is not good for videos
with a lot of fast movement (like sports) since at fast movements, we will be able to see the alternating lines since the two adjacent frames would be vastly
different. Interlaces is probably better when there are none to very small and slow movements.

**lossless.** The quality of some codecs that no quality is lost in both the coding/compression and decoding/playback phases of a video's "life cycle". File sizes
in a lossless compression are less than the original files, since some redundant data is actually thrown away. However, only an amount of data is thrown away that
still guarantees 100% quality of the original, thus lossless compression file sizes are significantly larger than their compressed counterparts.

**lossy.** The inevitable quality of some codecs to lose some quality both in the coding/compression and decoding/playback phases of a video's "life cycle". If
carefully configured, the loss may not be visually and/or audibly noticeable.

**mux.** To pack together audios and videos into one file so that they all play simultaneously when the process is done.

**pass.** When saving a video or audio put together in their respective editor, the number of times that the entire video or audio will be analyzed to determine
the most optimal bitrate at all time intervals. The greater the number of passes, the better the quality for the least amount of storage space will result, however,
would increase rendering time.

**progressive.** Broadcast/playback of a video file where all pixels are sent for each frame. This requires more bandwidth since more data is transmitted at a time.

**resolution.** The dimensions of a video (or image) in pixels. Higher numbers mean higher resolution, which will allow greater detailed picture. Lower numbers mean
inevitably blockier picture. Resolution is NOT to be confused with viewport size during playback, which is how large physically on a screen a video will be stretched or
shrunk into. A video player can resize the viewport at playback, and at the same viewport size, two videos of different resolution will have a noticeable difference
in clarity.

**stream.** 