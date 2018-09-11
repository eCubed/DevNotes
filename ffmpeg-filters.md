# FFMPEG Filters

A filter is a process that will alter the physical look/sound of a stream of video, audio, or both. Filters can be chained together in one command, sort of like
an assembly line, and we would get the one final output.

The simpler filters may be to resize or rescale an image or video. Picture-affecting filters may be stuff like blur, oil painting, pixellate, etc.

## Scaling (Will Require Reencoding)

`ffmpeg -i {input} -vf scale=iw/2:-1 {output}`

We specify `-vf` to tel FFMPEG that we're applying a filter. The filter name, in this case, `scale` will need its own parameter. The `scale` filter takes a
value of the form `{width}:{height}`. For our example, we're taking the input width (`iw`), and dividing it by two, so we're halving the width. The -1 for the
`{height}` is a special indicator to mean "calculate the height for the new output so that the picture retains its dimension proportions as the original."

## Other Filters

We cannot list every single filter here, since there are too many. We will only list the ones we will use often. The list of filters are
[here](https://ffmpeg.org/ffmpeg-filters.html).

## General Filter Syntax
All filters will have the following syntax:

`{filterName}={setting1=value1}:{setting2=value2}:...:{settingn=valueN}`

* We first specify the filter name, and those could be found [here](https://ffmpeg.org/ffmpeg-filters.html).
* Each filter comes with settings, which are also in the reference, and each setting is set with an equals sign.
* Settings of a filter are delimited by a colon.
* There are no spaces between settings and their values, nor between setting-value pairs.

## Filter Chain (with Only One Input)

We can actually perform filters in succession before saving the final file. The general syntax is

`ffmpeg -i {input} -vf {filter1},{filter2},..,{filterN} {output}`

The various filters are comma-separated, WITH NO TRAILING SPACE AFTER THE COMMAS!

Here's an example where we'll desaturate a video and then increase the video size by a factor of 1.5:

`ffmpeg -i {input} -vf eq=saturation=0, scale=iw*1.5:-1 {output}`