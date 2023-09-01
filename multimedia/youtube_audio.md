# Rip Audio From Youtube

I found this video on youtube:

Charlie Parker grandes maestros del Jazz 15  
<https://www.youtube.com/watch?v=G3dv7sDCgAg>

So, I wanted to convert it to just mp3, I don't need the video part.

I went to <http://www.clipconverter.cc> and plugged in the youtube url
and it delivered an mp3 file:  
*Charlie Parker grandes maestros del Jazz 15.mp3*

This file contained 20 tracks, which were conveniently listed with track
number and title on the Youtube page. I copied the list of tracks from
the Youtube page and pasted into a file, Contents.txt. Looks like this:

    01. RED CROSS (Parker -- Grimes)
    02. GROOVIN´ HIGH (WHISPERING) (Parker -- Gillespie)
    03. HOT HOUSE (WHAT IS THIS THING CALLED LOVE?) (Dameron)
    04. CONGO BLUES (Norvo)
    05. THE STREET BEAT (LADY BE GOOD) (Thomson)
    ... etc ...

Now I need to split the one mp3 file into 20 files, one for each track,
so I use *mp3split-gtk*.  
I chose the option to name the files like so:  
*01.mp3*  
*02.mp3*  
*...etc...*  
*20.mp3*  
Next, look up the song titles in the file Contents.txt and rename the
files:

    #!/bin/bash
    while read title; do                  # read a line of text into the variable "title"
        track=${title:0:2}                # Bash substring operator, get first 2 characters
        mv "${track}.mp3" "${title}.mp3"  # moving a file is same as renaming it
    done < Contents.txt                   # the file we are reading from

This renames a file:  
*01.mp3*  
to  
*01. RED CROSS (Parker -- Grimes).mp3*

# Use ffmpeg to extract audio from .webm

If you need to extract the audio from an .WEBM movie file to an .MP3
audio file you can execute the following:

    FILE="the-file-you-want-to-process.webm";
    ffmpeg -i "${FILE}" -vn -ab 128k -ar 44100 -y "${FILE%.webm}.mp3";

The first command will assign the file name to a variable, we do this to
avoid typing errors in the second command where we might want to use the
same name for the audio file.

The second command, will use ffmpeg to extract the audio. The -i flag,
indicates the file name of the input. We used the flag -vn that will
instruct ffmpeg to disable video recording. The -ab flag will set the
bit rate to 128k. The -ar flag will set the audio sample rate to 441000
Hz. The -y flag will overwrite output file without asking, so be careful
when you use it.

In case we want to automatically process (batch process) all .WEBM video
files in a folder we can use the following:

    for FILE in *.webm; do
      echo -e "Processing video '\e[32m$FILE\e[0m'";
      ffmpeg -i "${FILE}" -vn -ab 128k -ar 44100 -y "${FILE%.webm}.mp3";
    done;

The above script will find all .WEBM files in the folder and process
them one after the other.

The following command will find all webm files that are in the current
directory and in all sub-folders and extract the audio to mp3 format.

    find . -type f -iname "*.webm" -exec bash -c 'FILE="$1"; ffmpeg -i "${FILE}" -vn -ab 128k -ar 44100 -y "${FILE%.webm}.mp3";' _ '{}' \;

The filename of the audio file will be the same as the webm video with
the correct extension. The webm extension will be removed and replaced
by the mp3 extension e.g hi.webm will become hi.mp3
