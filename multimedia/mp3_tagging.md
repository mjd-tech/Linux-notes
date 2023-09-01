# MP3 files: Tagging
- A Tag is "metadata" contained within an mp3 file, stored in "ID3" format.
- Tags consist of key-value pairs called "frames"
- Unfortunately, ID3 is a mish-mash of cryptic, vaguely defined 4-letter "frames", and incompatible versions: ID3v1, ID3v2.3, ID3v2.4
- software such as MusicBrainz Picard and Mp3tag, map "friendly" names to the cryptic 4 letter sequences

"frames" used by Jellyfin, Kodi, etc.

| Friendly Name | Frame Name |
|:------------- |:-------    |
| Title         | TIT2       |
| Artist        | TPE1       |
| Album Artist  | TPE2       |
| Album         | TALB       |
| Track Number  | TRCK       |
| Genre         | TCON       |
| Date          | TDRC       |

These are the most important
- Title: name of the song (track)
- Track Number: makes sure things play in order
- Album: name of the album.
- Album Artist: Who recorded it
- Artist: Normally same as Album Artist, but not always. Classical music, for example.

Date: the frame name depends on whether it's ID3v2.3 (TYER) or ID3v2.4 (TRDC). very confusing.

Genre: there are dozens of "genres" in MusicBrainz, these tend to be subjective, and confusing (Pop vs Popular?)
I use only a few genres, when in doubt, it goes in "Pop"

Special cases:
- Classical music: Album Artist is the orchestra, Artist is the composer
- Compilations with multiple Artists: Album Artist is "Various Artists", Artist is the performer.
- My ad-hoc compilations of same artist: Album is "Misc"

## Jellyfin
- Jellyfin uses Folder Structure, ID3 tags and MusicBrainz lookups to organize music in its SQLite database.
- But it's not clear how all this works.
- When you go to "edit metadata", it's not clear where the metadata comes from.

Due to the way Jellyfin and Finamp present data, I use different "libraries" for each "genre".
That way I get the "Album Artists" for each "genre". If you just put everything in one "library",
you can only get a list of Albums for each genre. 

Jellyfin web client has "filters" that supposedly would display Album Artists per genre,
but it doesn't seem to work right.

## Finamp
- Finamp displays Albums, Artists (Album Artists), Playlists, Genres, Songs
- Does not have "filters"
Therefore if you want Album Artists per genre, you need to use separate music "libraries" for each genre.

## Folder Structure
Under the top level directory: artist/album/songs

1.  top level directory contains **only folders**, no files
2.  Each **artist** folder contains **only folders**, no files
3.  Each **album** folder contains **only files**, no folders
4.  Each **album** folder contains only **mp3** files and **cover.jpg** or **cover.png**

Songs should be "01 - Name of song.mp3" where 01 is the track number

### Testing Folder Structure
cd to the "top level directory"


```
# Find violations of Rules 1 and 2
find . -maxdepth 2 -type f    # should display nothing

# Find violations of Rule 3
find . -mindepth 3 -type d    # should display nothing

# Find violations of Rule 4
find . -type f -not -name "*.mp3" | sed -E 's/(.*\/)(.*)/\2\t\1/'
# should display only cover.jpg or cover.png
```

## Multi-CD Albums with folders for CD1 CD2, etc
- Jellyfin does **not** deal with these too well.
- It ends up treating the Album as **separate albums**.
- The best thing to do is **combine** all the songs into the **"album"**
  folder

You must **edit mp3 tags** before combining 2 or more "CD" folders.  
Ignore this at you own peril...

When you combine folders, you need to **change the track number tags**

- Usually, each CD will begin track numbers at 1
- You **do not** want multiple songs with track number 1.
- Because when you play the album, things will play in the wrong order.
- The easiest thing to do is add 100 to track numbers on CD1, 200 to
  track numbers on CD2, etc.

Example - Before:
```
artist/
  album/
    CD 1
      01 - Foo.mp3
      02 - Bar.mp3
    CD 2
      01 - Baz.mp3
      02 - Bif.mp3
```

After:
```
artist/
  album/
    101 - Foo.mp3
    102 - Bar.mp3
    201 - Baz.mp3
    202 - Bif.mp3
```

## mid3v2

install the `python3-mutagen` package.

mid3v2 uses ID3 "frame" names for the tags. Here's a comparison with MusicBrainz Picard, which uses "friendly" tag names.

| Picard       | mid3v2 |
|:-------------|:-------|
| Title        | TIT2   |
| Artist       | TPE1   |
| Album Artist | TPE2   |
| Album        | TALB   |
| Track Number | TRCK   |
| Date         | TDRC   |
| Genre        | TCON   |

If the mp3 has an embedded image, mid3v2 calls it APIC

### Common Command Options

    mid3v2 [options] filename …

| Option                 | Description                                                                                                        |
|:-----------------------|:-------------------------------------------------------------------------------------------------------------------|
| -f, --list-frames      | Display all supported ID3v2.3/2.4 frames and their meanings.                                                       |
| -l, --list             | List all tags in the files.                                                                                        |
| -D, --delete-all       | Delete all ID3 tags.                                                                                               |
| --delete-frames=FRAMES | Delete specific ID3v2 frames (or groups of frames). FRAMES is a “,” separated list of frame names e.g. "TPE1,TALB" |

| Option                                                      | Description                                                                                                               |
|:------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| -a, --artist=ARTIST                                         | Set the artist information (TPE1).                                                                                        |
| -A, --album=ALBUM                                           | Set the album information (TALB).                                                                                         |
| -t, --song=TITLE                                            | Set the title information (TIT2).                                                                                         |
| -p, --picture=\<FILENAME:DESCRIPTION:IMAGE-TYPE:MIME-TYPE\> | Set the attached picture (APIC). Everything except the filename can be omitted in which case default values will be used. |
| -g, --genre=GENRE                                           | Set the genre information (TCON).                                                                                         |
| -y, --year=\<YYYY\>, --date=\<YYYY-\[MM-DD\]\>              | Set the year/date information (TDRC).                                                                                     |
| -T, --track=\<NUM\>                                         | Set the track number (TRCK).                                                                                              |

Track number can also be in the form NUM/NUM  
For example: 7/12 would indicate track 7 of 12  
Not all players interpret this correctly.

### Mp3Tag
**NOTE:** I used Mp3tag to bulk edit thousands of my mostly untagged mp3 files. 
But I don't use it anymore for adding new stuff to the library. I use `mid3v2`

Mp3tag is a Windows application, but it runs in Linux, Use PlayOnLinux to install.

This is a common scenario:

    artist
      album
        CD 1
          01 - Foo.mp3
          02 - Bar.mp3
        CD 2
          01 - Baz.mp3
          02 - Bif.mp3

Note:

- Sometimes you see Disc 1, Disc 2, etc.
- It doesn't matter as long as the rightmost character is a number.
- We are going to use the rightmost character (multiplied by 100) to set
  the track numbers.
- This will work for albums with up to 9 CDs.

Launch Mp3tag

- add the "album" directory. This will load all the CDs.
- Make sure the "title" tags are sane. Ideally, the title is in the
  filename.
- Select all files Ctrl-A
- Choose "Convert Tag - Filename
- Use the following incantation


    $num($add(%track%,$mul($right(%_directory%,1),100)),3) - %title%

Note: Use the Preview button to see what it will do. If it looks ok then
hit OK.

What it does:

- get the rightmost character of the directory name (should be a digit)
- multiply it by 100
- add it to the existing track number
- rename the file. ex. 101 - Foo.mp3

Next:

- choose "Convert Filename - Tag
- Use the following incantation


    %track% - %_dummy%

What it does:

- set track numbers based on the first 3 digits of the filename.
- it reads "track followed by a space, a dash, another space, ignore the
  rest"

Next:

- Set the Album tag in the left hand panel to correct Album name.
- Set the Album Artist tag in the left hand panel. It should be the same
  as Artist, unless it's a compilation, then use "Various Artists"
- Click on the disk of File...Save Tag

Finally:

- using file manager, move all the files into one folder.
- delete extraneous folders
- make sure everything follows the rules.

Result:

- Album will appear as **one Album**
- Songs are listed in the correct order in the filesystem
- Songs are played in the correct order, no matter how you do it.

