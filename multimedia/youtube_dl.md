# Download YouTube Playlist And Convert To MP3 Using youtube-dl

You'll need to have youtube-dl and ffmpeg installed on your system.  
youtube-dl stops working regularly due to changes to YouTube, so you'll
want to have the latest version installed on your system.  
<https://ytdl-org.github.io/youtube-dl/index.html>

To download an entire YouTube playlist (must not be private) using the
best available audio format, extract the audio, and convert the
resulting files to 160K MP3, use (this is a single command)

    youtube-dl --ignore-errors --format bestaudio --extract-audio --audio-format mp3 --audio-quality 160K \
    --output "%(playlist_index)02d - %(title)s.%(ext)s" --yes-playlist '<YouTube playlist URL>'

Command explanation:

    --ignore-errors         makes youtube-dl continue in case of errors, for example skip that were removed, or not available in your country
    --format bestaudio      downloads the best available audio quality format
    --extract-audio         as the name implies, it extracts the audio from the video
    --audio-format mp3      specifies the audio format
    --audio-quality 160K    You can specify an exact bitrate, like 128K, 160K, etc., or a VBR quality value between 0 (best) and 9 (worst), with 5 being default.
    --output "%(playlist_index)02d - %(title)s.%(ext)s"    the output filename template; Ex: 01 - Comin' Home.mp3   You want to use the template, the default is ugly.
    --yes-playlist          makes it so if the URL refers to a video and a playlist, it still downloads the whole playlist.
                            This is useful because if you find a playlist on YouTube, click on a video from that playlist, 
                            then copy the URL and try to use youtube-dl to download it without --yes-playlist, only one video will be downloaded, instead of the whole playlist.
    '<YouTube playlist URL>' is the URL to the YouTube playlist you want to download. You'll need to replace this with the actual YouTube playlist URL. 
                             Add single quotes on Linux and double quotes on Windows to avoid running into issues in some cases 
                             (as an example, if you skip the quotes, even when using the --yes-playlist option if the video URL includes a "&" symbol, only one video is downloaded).

It's also worth noting that the original video will be deleted, so once
the command finishes processing all files, you'll only end up with MP3
files.

Example \#1 - download a YouTube playlist and convert it to highest
available quality .mp3:

    youtube-dl --ignore-errors --format bestaudio --extract-audio --audio-format mp3 --audio-quality 160K \
    --output "%(title)s.%(ext)s" --yes-playlist 'https://www.youtube.com/list=PLdYwhvDpx0FI2cmiSVn5cMufHjYHpo_88'

Example \#2 - download a YouTube playlist and convert it to highest
available quality .mp3, even when the link is to both a YouTube video
AND a YouTube playlist (this works thanks to --yes-playlist and the fact
that we've used single quotes around the YouTube URL):

    youtube-dl --ignore-errors --format bestaudio --extract-audio --audio-format mp3 --audio-quality 160K \
    --output "%(title)s.%(ext)s" --yes-playlist 'https://www.youtube.com/watch?v=PsO6ZnUZI0g&list=PLdYwhvDpx0FI2cmiSVn5cMufHjYHpo_88'

Want to only download part of a YouTube playlist? Use --playlist-start
NUMBER to specify the start number and --playlist-end NUMBER to specify
the end number of videos to download.

For example, to download video 5 to 10 from a YouTube playlist, use
--playlist-start 5 and --playlist-end 10, like this:

    youtube-dl --ignore-errors --format bestaudio --extract-audio --audio-format mp3 --audio-quality 160K \
    --output "%(title)s.%(ext)s" --yes-playlist --playlist-start 5 --playlist-end 10 '<YouTube playlist URL>'

But what if you don't have a regular YouTube playlist to download, but
instead you have a text file with links to YouTube videos? You can
download all the YouTube links in a text file by using
--batch-file="/path/to/playlist.txt" instead of '\<YouTube playlist
URL\>':

    youtube-dl --ignore-errors --format bestaudio --extract-audio --audio-format mp3 --audio-quality 160K \
    --output "%(title)s.%(ext)s" --batch-file="/path/to/playlist.txt"

Replace /path/to/playlist.txt with the path and name of the text file
containing the YouTube video links.

For more on youtube-dl, see its help (youtube-dl --help) and the project
GitHub page.
