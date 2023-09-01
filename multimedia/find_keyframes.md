# Find KeyFrames

To generate a list of keyframes in a video, using ffprobe (included in ffmpeg)

    ffprobe -show_frames -select_streams v:0 -print_format csv -v quiet -pretty YOUR_VIDEO_FILE | \
    awk -F, '$4 == 1 {print $6}'

ffprobe dumps out a lot of info for each frame, unfortunately there's no
way to limit the output to just the keyframes. Since we are stuck with
all the info, we choose CSV output because it is easy to parse with awk.

These is what you get if you do not specify print_format. If you specify
cvs, the first field will be "frame"

    [FRAME]
    media_type=video
    stream_index=0
    key_frame=1
    pkt_pts=5495
    pkt_pts_time=0:00:05.495000
    pkt_dts=5495
    pkt_dts_time=0:00:05.495000
    best_effort_timestamp=5495
    best_effort_timestamp_time=0:00:05.495000
    pkt_duration=36
    pkt_duration_time=0:00:00.036000
    pkt_pos=1474716
    pkt_size=31830
    width=1024
    height=576
    pix_fmt=yuv420p
    sample_aspect_ratio=N/A
    pict_type=I
    coded_picture_number=150
    display_picture_number=0
    interlaced_frame=0
    top_field_first=0
    repeat_pict=0
    [/FRAME]
