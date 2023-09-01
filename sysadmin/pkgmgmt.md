# Ubuntu/Debian Package Management
Packages are "zip" files that contain executables, config files, man pages and "metadata"

The "Rosetta" lists equivalent commands for Arch, Red Hat/Fedora, Debian/Ubuntu, Suse and Gentoo.
https://wiki.archlinux.org/title/Pacman/Rosetta

## Debian-based Tools:
In general, use apt for interactive use. Use apt-get, apt-cache etc for
scripts.

| Command (run as root)              | Action                                                                              |
|------------------------------------|-------------------------------------------------------------------------------------|
| apt update                         | get package information from repositories                                           |
| apt upgrade                        | install all available upgrades, but never remove existing packages                  |
| apt full-upgrade                   | as above, but removes existing packages if needed to satisfy dependencies           |
| apt install package-name           | install package from repository. Can specify more than one package-name             |
| apt install foo=1.2.3              | install specific version of package foo, from repository, the version must exist    |
| apt install /path/to/something.deb | install a .deb file. same as `sudo dpkg -i something.deb`                           |
| apt reinstall package-name         | use this to reset a packages configs to default                                     |
| apt remove package-name            | remove package but leave config files                                               |
| apt purge package-name             | remove package and config files (never removes config stuff in your home directory) |
| apt autoremove                     | remove packages no longer needed as dependencies                                    |

| Command                    | Action                                                                          |
|----------------------------|---------------------------------------------------------------------------------|
| apt search `regex`         | search available package using regex                                            |
| apt show package-name      | show information about a package                                                |
| apt list --installed       | list all installed packages                                                     |
| apt list foo*              | list all packages beginning with foo (glob pattern not regex), installed or not |
| apt list foo* --installed  | list installed packages beginning with foo                                      |
| apt list --upgradable      | list packages that can be upgraded                                              |

| Command                | Action                                                                         |
|------------------------|--------------------------------------------------------------------------------|
| sudo apt-get clean     | remove downloaded .deb files from /var/cache/apt/archives/                     |
| sudo apt-get autoclean | only removes package files that can no longer be downloaded from their sources |

| Command                           | Action                              |
|-----------------------------------|-------------------------------------|
| sudo apt-mark hold package-name   | prevent package from being upgraded |
| sudo apt-mark unhold package-name | reset to normal                     |
| sudo apt-mark showhold            | show held packages                  |

## dpkg

| Command                           | Action                                                                                       |
|-----------------------------------|----------------------------------------------------------------------------------------------|
| dpkg -i /path/to/some_package.deb | Install a package                                                                            |
| dpkg -r package-name              | remove a package without removing its configuration files                                    |
| dpkg --purge package-name         | remove a package along with its corresponding configuration files, but not in home directory |
| dpkg -l `regex`                   | List available packages and the state of those packages. regex is optional                   |
| dpkg -L package-name              | List files in an installed package                                                           |
| dpkg --contents /path/to/some.deb | List files in a not-installed package file                                                   |
| dpkg -S /path/to/file             | show which package provides a file                                                           |

Beware! recent Ubuntu and Debian symlink /bin to /usr/bin.  
For example a core utility like grep is installed to /bin, but
`which grep` shows /usr/bin.  
If you run `dpkg -S /usr/bin/grep` it **doesn't work right**.  
So do this instead: `dpkg -S /bin/grep` 

## Synaptic - Quick Filter

How to get Quick Filter back in Synaptic

you can try a couple of things:

    sudo apt-get install apt-xapian-index
    sudo update-apt-xapian-index -vf

If that doesn't work you can try:

    sudo apt-get install --reinstall synaptic
    sudo dpkg-reconfigure synaptic

## Disable Snaps
```
# First remove snapd and the snap folder in the home directory:
sudo apt purge snapd
rm -vrf ~/snap

# prevent snapd from re-installing:
cd /etc/apt/preferences.d
sudo vi nosnap.pref 

# Put this in the file you just created:
Package: snapd
Pin: release a=*
Pin-Priority: -10
```

(this is taken from Linux Mint, which disables snaps by default)

## Chromium Browser

```
# Add the System76 PPA in the Terminal.
sudo add-apt-repository ppa:system76/pop
sudo apt update
```

Search for Chromium in the Synaptic Package Manager and install.

**Important:** Disable, but don't remove System76 PPA in System Sources
once Chromium is installed.

## Compiling from source

<https://help.ubuntu.com/community/CompilingSoftware>

Look for a README or INSTALL file for instructions.

- Install build tools


    sudo apt install build-essential checkinstall

- Download source code. It's usually a .tar.gz or .tar.bz2 file extension. (tarball)
- Extract the tarball. It creates a new directory


    # Typical commands to extract tarball
    tar -xzf file.tar.gz
    -or-
    tar -xjf file.tar.bz2

    * Change into the new directory.
    * Run configure (sometimes the command is different, check the README or INSTALL file)

    ./configure

- It will tell you if it's missing dependencies.
- Usually these are libraries and they might have a **slightly different name** than the package name.
- Then proceed to look for the package using apt search
- Install the missing dependencies, then run ./configure again.
- When ./configure is satisfied, run `make`

    make

- Next you typically run `sudo make install`
- But instead run `sudo checkinstall`

    sudo checkinstall

- This will create a .deb file and install it to the system.
- The deb file can be used on another similar system. (same release of Ubuntu ie. 20.04 )

If you are compiling a package already in the repositories, because you
want a newer version, install the build dependencies of the package:

    sudo apt-get build-dep <package>

This will ensure that you have all the dependencies of the package are
installed, and hopefully the configure script will not complain about
your system having old version of the dependencies installed, in which
case you will have to compile the dependencies also.

## List Manually Installed Packages

Using apt-mark:

    comm -23 <(apt-mark showmanual | sort -u) <(gzip -dc /var/log/installer/initial-status.gz | sed -n 's/^Package: //p' | sort -u)

Using aptitude:

    comm -23 <(aptitude search '~i !~M' -F '%p' | sed "s/ *$//" | sort -u) <(gzip -dc /var/log/installer/initial-status.gz | sed -n 's/^Package: //p' | sort -u)

Note: This method is not 100% perfect. `comm -23` means "compare 2 files, and output the lines in file 1 that are not present in file 2."

