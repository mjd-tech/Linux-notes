# tar
Create and extract archive files.

Compression: gzip, bzip2, xz (and others, but these are most common)

> :memo: **Note:** Options to tar can be specified in 3 ways
> 1. first argument is a cluster of option letters. `tar xvf foo.tar.gz`
> 2. single dash. `tar -xvf foo.tar.gz`
> 3. double-dash. `tar --extract --verbose --file foo.tar.gz`

## Extract

    # to Current Directory 
    tar xf archivefile.tar.gz

    # verbose output, shows files being extracted 
    tar xvf archivefile.tar.gz
    
    # to specified Directory
    tar xf archivefile.tar.gz -C /some/path
    
    # extract a bz2 file - tar infers the compression algorithm based on file extension
    tar xf archivefile.bz2

## Create

    # Create archive with gzip compression
    tar caf path/to/target.tar.gz path/to/source
    
    # Verbose, list the files being archived
    tar cvaf path/to/target.tar.gz path/to/source

    # Create archive with xz compression
    tar caf path/to/target.tar.xz path/to/source

> :memo: **Note:** the `a` option makes tar infer the compression algorithm based on file extension

Sometimes you get an annoying message: "Removing leading '/' from member names".
This is actually a good thing, the archive should not contain absolute paths.
However, tar will return a non-zero return code and that can cause problems.
Also if running tar in a cron job that sends email, you'll get a lot of these messages.

To prevent this, use the -C option

    # archive /etc, without annoying message. 
    tar caf path/to/target.tar.gz -C / etc
    
    # the above is equivalent to
    cd /; tar caf path/to/target.gz etc

Both of these store the etc directory as a **relative path**, without a leading slash.
This is what you want.

## List

    # List contents of tar file
    tar tf path/to/source.tar
    
    # Verbose listing
    tar tvf path/to/source.tar

Pipe the output to `grep` to search a file, or to `less` to browse the list.

## Extract specific file(s)

add the file name after the command...

    tar xf latest.tar.gz wordpress/xmlrpc.php
    
    # More than one file can be specified...
    tar xf latest.tar.gz wordpress/xmlrpc.php wordpress/.htaccess

Extract a directory...

    tar xf latest.tar.gz wordpress/wp-includes

## Extract multiple files using wildcards

    tar -xvf abc.tar.gz --wildcards "*.txt"

Be sure to **quote** the wildcard, to prevent the shell from expanding the `*`
