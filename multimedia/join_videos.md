# Join Videos

Here's how to merge several video files into a single one with Mencoder.

    mencoder -oac copy -ovc copy -idx -o output.avi video1.avi video2.avi video3.avi

- The -oac copy option tells mencoder to just copy the audio stream (no
  reencoding). -ovc copy does the same with the video stream.
- The -idx option asks mencoder to build the index if none was found.
  This makes searching inside the video easier.
- -o stands for "output" and specifies the output file.
- The last few parameters are the input video files.

You can join as many files as you like. This will only work for videos
of the same resolution, and using the same codecs, but any video format
that mencoder recognizes is fine.
