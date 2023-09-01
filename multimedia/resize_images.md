# Resize images from command line - mogrify

part of the imagemagick suite. *mogrify* converts files in place. To
keep original, use *convert*. To get info, use *identify*.

Using Mogrify:

find all jpgs in current directory and subdirs, strip jpg comment info
and resize to a maximum of 800x800:

    find . -name "*.[Jj][Pp][Gg]" -exec echo {} \; -exec mogrify -strip -resize 800x800 {} \;

transform all the .tiff images in a directory to .jpg

    mogrify -format jpeg *.tiff

Create thumbnails:

    mogrify -geometry 120x120 *.jpg

reduce the size of any given image:

    mogrify -resize 50% *.jpg

resize all JPEG images in a folder to a maximum dimension of 256x256:

    mogrify -resize 256x256 *.jpg

Convert PNG images to the JPEG format:

    mogrify -format jpg *.png

Rotate an Image:

    mogrify -rotate "-90" test.jpg

Add Annotate to Image:

    mogrify -comment 'My holiday highlight' test.jpg

Add some more Decorative Border:

    mogrify -frame 20x20 test.jpeg

more info: mogrify -help

Find files larger than 800x800 and resize, preserve aspect ratio

``` bash
#!/bin/bash
# finds jpegs in specified directory and below,
# checks size, if height or width greater than 800
# resizes to 800x800 and strips metadata

[[ -z $1 ]] && {
    echo "Must specify directory"
    exit 1
}

the_dir=$1

[[ -d $the_dir ]] || {
    echo "Invalid directory"
    exit 1
}

Trim_it () {
    if identify -format "%h %w" "$1" | awk '
        $1 > 800 {exit 0}
        $2 > 800 {exit 0}
                 {exit 1}
        '
    then
        echo "resizing $1 to 800x800"
        mogrify -strip -resize 800x800 "$1"
    else
        echo "skipping $1"
    fi
}

export -f Trim_it

find "$the_dir" -name "*.[Jj][Pp]*[Gg]" -exec bash -c 'Trim_it "$0"' {} \;
```

Note: the find command substitutes {} with the current filename that was found.  
when using bash -c, $0 is the first argument (the filename), not the script name.  
see https://stackoverflow.com/questions/4321456/find-exec-a-shell-function-in-linux
