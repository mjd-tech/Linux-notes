# Video Tools

## Handbrake

Use handbrake to convert most any format to mp4, m4v or mkv.

Many DVD's are at "480p" resolution

## ffmpeg

For a while, this was not included in Ubuntu / Mint repositories.
Instead, there was avconv, which is similar but different syntax. Recent
releases now have ffmpeg available.

### ffprobe

- Gets info about a media file. Is included with the ffmpeg package.
- Most common usage: `ffprobe -hide_banner somefile.mp4`

lists information about the streams (video, audio, subtitle)

Get resolution:
```
ffprobe -v error -select_streams v -show_entries stream=width,height -of csv=p=0:s=x file.mp4
```

### ffmpeg command

Most common usage

    ffmpeg [global_options] -i input_file [output_file_options] output_file

Simple conversion

    ffmpeg -i somefile.avi somefile.mp4

### Convert flash video to mp4

    ffmpeg -i input.flv -c copy -copyts output.mp4

### Convert container to mp4

Prior to HTML5, web video was handled by Adobe Flash, implemented as a
plug-in to the web browser. Microsoft and others had competing formats,
but all of them required a proprietary plug-in.

Modern HTML5-compliant browsers have media playing capabilities built
in. This allows for media streaming, using a `<video>` tag.

Initially, the open source browsers, Chrome, Firefox, and Opera all went
with the **WebM** format which is royalty free. Safari, Internet
Explorer, & Chrome supported the widely accepted, but not royalty free
**MP4** format.

Over time this has changed and now all major browsers provide support
for the MP4 format.

For best compatability with browsers, and media player applications,
your media files should be

|             |          |
|-------------|----------|
| container   | MP4      |
| video codec | H.264    |
| audio codec | AAC      |
| subtitles   | mov_text |

Say you have an mkv file with one H.264 video and one AAC audio stream.
No subtitles.  
You just want to change the container, without any transcoding:

    ffmpeg -i "somefile.mkv" -c:v copy -c:a copy "somefile.mp4"

This uses the "copy" codec for both video and audio stream. "copy" does
not transcode, so this operation does not take much time to complete.

Often a file will contain one video stream but multiple audio streams,
usually for different languages. By default, ffmpeg will only copy or
transcode ONE video stream and ONE audio stream.

To get more than one stream, use the -map option.

- Map uses a `<num>:<num>` format.
- The first number is the input file (can use more than one input file)
- The second number is the stream
- files and streams numbers start at 0 not 1
- Normally you are using just one input file, so the first number is 0.
- Use ffprobe to identify the streams.

<!-- -->

    Using map
    ---------
    -map 0:1            the 2nd stream
    -map 0:1 -map 0:3   the 2nd and 4th stream, yep two map commands
    -map 0              all streams

Now if the MKV file has a subtitle track, we would probably need to
transcode the subtitle. MKV and MP4 use different formats for subtitles.

    ffmpeg -i somefile.mkv -c:v copy -c:a copy -c:s mov_text somefile.mp4

If the MKV file uses the AC3 format for the audio stream, we need to
convert it to AAC. MP4 does not support AC3. We also want to preserve
5.1, using the Dolby Pro Logic.

      ffmpeg -i somefile.mkv -c:v copy -c:a aac -dsur_mode 2 -c:s mov_text somefile.mp4

This will take a little while since the audio stream is being
transcoded. But nowhere near as long as transcoding the video stream

### Reduce resolution for smaller file size

This is usually much quicker than transcoding.

#### Simple Rescaling

If you need to simply resize your video to a specific size (e.g
320x240), you can use the scale filter in its most basic form:

    ffmpeg -i input.avi -vf scale=320:240 output.avi

#### Keeping the Aspect Ratio

If we'd like to keep the aspect ratio, we need to specify only one
component, either width or height, and set the other component to -1.
For example, this command line:

    ffmpeg -i input.avi -vf scale=320:-1 output_320.avi

will set the width to 320 pixels and will calculate the height according
to the aspect ratio.

Some codecs require the size of width and height to be a multiple of n.
You can achieve this by setting the width or height to -n:

    ffmpeg -i input.avi -vf scale=320:-2 output_320.avi

Using Variables

There are also some useful variables which can be used instead of
numbers.

For example, you can use something like this (iw = input width, ih =
input height):

    ffmpeg -i input.avi -vf scale=iw*2:ih output_double_width.avi

If you want to half the size, just multiply by .5 or divide by 2:

    ffmpeg -i input.avi -vf "scale=iw*.5:ih*.5" output_half_size.avi
    ffmpeg -i input.avi -vf "scale=iw/2:ih/2" output_half_size.avi
