# Thumbnails

## Locations

- ~/.cache/thumbnails/fail
- ~/.cache/thumbnails/large
- ~/.cache/thumbnails/normal

## Clear the thumbnail cache

Launch a terminal window:

    rm -v -f ~/.cache/thumbnails/*/*.png ~/.thumbnails/*/*.png

Followed by:

    rm -v -f ~/.cache/thumbnails/*/*/*.png ~/.thumbnails/*/*/*.png

Note: this will probably affect the thumbnails on your desktop as well;
in that case it should suffice to simply refresh your desktop (or log
out and in again), which will create them anew.

Repeat the above in each user account.

## Limit the size of the thumbnail cache

Note: for this, you have to install the package dconf-tools first.

Launch a terminal window:

    dconf-editor

Navigate to:  
org:gnome:desktop:thumbnail-cache  
and set the maximum-age to 90 (press Enter) and the maximum-size to 64  
(press Enter again).

Repeat this in each user account.

## Do not use a thumbnail cache

To prevent all thumbnails from being cached to disk, delete the
~/.cache/thumbnails folder then link it to /dev/null

    rm -r ~/.cache/thumbnails && ln -s /dev/null ~/.cache/thumbnails

Thumbnails of specific file types can be disabled using dconf editor.  
search for thumbnail.
