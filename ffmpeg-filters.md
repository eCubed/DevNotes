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

`ffmpeg -i {input} -vf "{filter1}, {filter2},.., {filterN}" {output}`

The various filters are comma-separated. We may put spaces to make things clearer within the double quotes,.

Here's an example where we'll desaturate a video and then increase the video size by a factor of 1.5:

`ffmpeg -i {input} -vf "eq=saturation=0, scale=iw*1.5:-1" {output}`

## Complex Filters (-filter_complex)

What if we wanted two or more inputs, perform filters on each of them independently, and then join the results for the final output? The setup of this kind of feat
is called a filter graph. In general, we would be filtering one or two inputs together, many times, filtering another one or two inputs together at the same time.
We can then combine two or more results of separate filterings, and then perform a filter on those, and so on, until we output to a single file.

First, we will be using the `-filter_complex` flag instead of `-vf` or `-af`. The general syntax of `-filtercomplex` is as follows:

`ffmpeg [-i {input1}]+ -filter_complex "{singleFilterExpression1};{singleFilterExpression2};...;{singleFilterExpressionN}" {output}`

We'll take 1 or more inputs, indicate that we're about to use a filter graph with `-filter_complex`, and then write our complex filter. But now, let's focus
on what the `{singleFilterExpression}` looks like:

`{sources}{filterExpressions}{destination}`

`{sources}` is a list of inputs, each item listed in square brackets ([0][1], for example). If the value of the input is an integer, it will refer to the order
placement of the particular input we've specified with `-i` flags. The value can be a string, which is a handle for the result of an earlier filtering. We will 
address this string index shortly. Note that some filters may require an exact count of inputs, so we have to make sure we have the right number of sources!

We then continue with the the filter expression per documentation and as our project needs.

`{destination}` will be a handle for the result of a single filtering operation, which we can name whatever we want. It is also surrounded by square brackets.
It will be necessary to name a filtering's immediate output because it may be needed for another filtering down the line. The very last filtering may omit the
`{destination}` parameter because FFMPEG would then understand that it's the final output anyway, whose results will be saved to the {output}.

## Drawing Text on a Video

Text has a lot of visual information associated with it just like it does in CSS. We will be using the `drawtext` filter to write some text on a video.

`ffmpeg -i {input} -vf drawtext="{options}" {output}`

There are tons of options we can choose, and we would consult the documentation for what we may want to include.

When we specify the `fontfile` parameter, we'll need to escape the colon character this way: `\:` because the colon is reserved for separating "attribute-value"
pairs.

## Superimposing a Video Onto Base Video

We will need to be given both the start time (offset) of the top video and its actual duration (which can be specified less than the actual top video's duration).

```
ffmpeg -i {base} -i {top} -filter_complex "[1]setpts=PTS+{topStartTime}/TB[top], 
 [0:v][top]overlay=enable='between(t,{topStartTime},{topStartTime+topDuration})'" {output}
```

## Superimposing N Videos Onto Base Video

```
ffmpeg -i {base} -i {top1} -i {top2} ... -i {topN} -filter_complex
   "
   [1]setpts=PTS+{top1StartTime}/TB[top1],
   [0:v][top1]overlay=enable='between(t, {top1StartTime}, {top1StartTime + top1Duration})[res1]',

   [2]setpts=PTS+{top2StartTime}/TB[top2],
   [res1][top2]overlay=enable='between(t, {top2StartTime}, {top2StartTime + top2Duration}}[res2]),

   [3]setpts=PTS+{top3StartTime}/TB[top3],
   [res2][top3]overlay=enable='between(t, {top3StartTime}, {top3StartTime + top3Duration}}[res3]),
   ...
   [N]setpts=PTS+{topNStartTime}/TB[topN],
   [res<N-1>][topN]overlay=enable='between(t, {topNStartTime}, {topNStartTime + topNDuration}}
   " {output}
```