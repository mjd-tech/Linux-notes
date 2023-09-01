# Handbrake

The command line version HandBrakeCLI can be used to batch process
files.

The easiest way is to put the source files in one directory and the
destination files in another. Change directory into the source, and do
something like this:

    for i in * ; do 
        HandBrakeCLI --preset "Very Fast 480p30" -i "$i" -o ./dest_dir/"${i/.mkv/.m4v}"
    done

Can use bash parameter substitution \${i//mkv/mp4} to change the file
extension if needed.

If source files are in a remote directory (for example, a smb share
point)  
Change directory into the destination and do something like this:

    for i in /run/user/1000/gvfs/smb-share\:server\=myserver\,share\=videos/Television/SomeShow/* ; do \
    HandBrakeCLI --preset "Very Fast 480p30" -i "$i" -o \
    "$(basename "$i" | sed 's/\.mkv$/.m4v/')"; done

The input file parameter -i of course needs the full path, but we need
to remove the path part, so we use basename.

This processes a video, preserves subtitles in "passthrough" mode (Does
not burn the subtitle into the movie's video track), and optimizes for
streaming. --subtitle=1-10, preserve up to 10 subtitle tracks, you can go
as high as 99. This is for when you don't know how many subtitle tracks there are.

    HandBrakeCLI --preset "Very Fast 480p30" --subtitle=1-10 --optimize -i somevideo.ISO \
   -o somevideo.mkv
