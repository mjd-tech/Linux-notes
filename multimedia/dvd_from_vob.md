# Make DVD from VOB files

## Files and Directories

    mkdir dvd && cd dvd
    mkdir AUDIO_TS VIDEO_TS

you need to have an "AUDIO_TS" and a "VIDEO_TS" directory. This is true
even if you are only burning video and therefore have an empty AUDIO_TS
directory.

Under the VIDEO_TS directory you need to have valid IFO, VOB and BUP
files. A directory listing of the VIDEO_TS directory should look
something like this:

    ll -a
    total 4432968
    dr-x------  2 someone group       4096 Aug 11 21:45 .
    drwxr-xr-x  4 someone group       4096 Aug 14 18:14 ..
    -r--------  1 someone group       8192 Aug 12 02:50 VIDEO_TS.BUP
    -r--------  1 someone group       8192 Aug 12 02:50 VIDEO_TS.IFO
    -r--------  1 someone group      81920 Aug 12 02:51 VTS_01_0.BUP
    -r--------  1 someone group      81920 Aug 12 04:41 VTS_01_0.IFO
    -r--------  1 someone group      88064 Aug 12 04:42 VTS_01_0.VOB
    -r--------  1 someone group 1073401856 Aug 13 14:07 VTS_01_1.VOB
    -r--------  1 someone group 1073467392 Aug 13 13:48 VTS_01_2.VOB
    -r--------  1 someone group 1073444864 Aug 13 14:03 VTS_01_3.VOB
    -r--------  1 someone group 1073623040 Aug 13 14:07 VTS_01_4.VOB
    -r--------  1 someone group  193714176 Aug 13 14:07 VTS_01_5.VOB

Use a command looking something like this:

    mkisofs -dvd-video -o dvdimage.iso dvd/
