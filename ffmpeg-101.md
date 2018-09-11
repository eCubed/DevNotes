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

## Extractic a Clip Starting at a Time for a Certain Duration

`ffmpeg --ss {startTime} -i {pathToFilenameWithFileExtension} -t {duration} {pathToOutputWithFileExtension}`

`{startTime}` and `{duration}` will be in the format `hh:mm:ss` for simplicity. We could be more accurate to fractions of milliseconds. We will let FFMPEG
encode with the same codecs as the original, so we don't specify codecs. Note that if we specified `-vcodec copy` and `-acodec copy`, we will get inaccurate
results for some reason.

## Extracting a Clip Starting at a Time and Ending at a Time

`ffmpeg --ss {startTime} -i {pathToFilenameWithFileExtension} -to {endTime} {pathToOutputWithFileExtension}`

`{startTime}` and `{endTime}`  will be in the format `hh:mm:ss` for simplicity. We could be more accurate to fractions of milliseconds. We will let FFMPEG
encode with the same codecs as the original, so we don't specify codecs. Note that if we specified `-vcodec copy` and `-acodec copy`, we will get inaccurate
results for some reason.

## Replacing the Audio of a Video (Both Same Length), No Reencoding

Here's an example where we'll have two inputs: the original video whose audio will be replaced, and the audio file whose content will replace the audio of the original
video.

`ffmepg -i {videoFile} -i {audioFile} -map 0:v:0 -map 1:a:0 -c copy {newVideoFile}`

* The `-map` flags are crucial in this example. There's a mapping for each of the video and audio streams.
* The value of the `-map` flag is `{whichInputByIndex}:{a|v}:{whichOutputByIndex}`. In our example, we have two indexes where the first one specified is the
original video, which will have an index of 0, and the second input being the audio file, which will have an index of 1. The second value is going to be an `a` or a
`v` indicating audio and video, respectively. The last value is the index of the output. Since we'll only have one output, its index will be 0.
* Note the second `-map flag` for handling the audio. We needed to specify that the audio comes from the second input parameter (index 1), and `a` to indicate
audio, and finally, 0 to indicate the index of the one output.
