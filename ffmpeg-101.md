# FFMPEG 101 - The Most Basic Commands

In general, an FFMPEG command looks like this:

`ffmpeg -i {pathToFilenameWithFileExtension} [{flagsForOutput}] {pathToOutputWithFileExtension}`

* `ffmpeg` is the name of the executable that does everything.
* `-i {pathToFilenameWithFileExtension}` is how to specify the source file that we will want to make a new file out of.
* `[{flagsForOutput}]` are the settings we would want to specify that instruct FFMPEG how to make the conversion to the output file. Flags are optional, but may
be required for specific tasks.
* `{pathToOutputWithFileExtension}` is the full path to the file we want to save when FFMPEG is finished with the task.

## Converting a File to Another Format (File Extension)

`ffmpeg -i {pathToFilenameWithFileExtension} {pathToOutputWithFileExtension}`

Notice that there are no flags specified. What happens here is that FFMPEG looks at the file extensions of both input and output files and intelligently guess what
codecs to use to create the output file. Transcoding happened here, which means that we will most likely lose video quality.

## Preserving the Quality When Converting Files

`ffmpeg -i {pathToFilenameWithFileExtension} -qscale 0 {pathToOutputWithFileExtension}`

The `-qscale 0` flag will preserve the video and audio quality during encoding.

## Removing the Audio from a Video

`ffmpeg -i {pathToFilenameWithFileExtension} -an {pathToOutputWithFileExtension}`

Think of `-an` as "Audio: none".

## Lowering the Frames Per Second

`ffmpeg -i {pathToFilenameWithFileExtension} -r {frameRate} {pathToOutputWithFileExtension}`

{frameRate} is an integer. We would need to know beforehand the frame rate of the source video so that the number we set the frame rate to is lower.

## Extracting the Audio As-Is from the Video

`ffmpeg -i {pathToFilenameWithFileExtension} [-vn] -acodec copy {pathToOutputWithFileExtension}`

Here, we are specifying an audio codec called `copy` which is whatever codec was used to encode the audio part in the first place. the `-vn` flag means do not
include the video in the output (just to make sure), but it's optional!

Note that the desired audio extension may not be compatible with the audio's codec, and we wouldn't be able to extract the audio.