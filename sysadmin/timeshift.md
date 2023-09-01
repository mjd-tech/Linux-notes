# Timeshift

In case the gui isn't working:

List snapshots:

    sudo timeshift --list

    Mounted '/dev/nvme0n1p2' at '/run/timeshift/1623577/backup'
    Device : /dev/nvme0n1p2
    UUID   : 68a383a9-4633-43c9-8ded-2a1fa6023b63
    Path   : /run/timeshift/1623577/backup
    Mode   : BTRFS
    Status : OK
    4 snapshots, 96.6 GB free

    Num     Name                 Tags  Description                                    
    ------------------------------------------------------------------------------
    0    >  2022-09-08_07-25-24  O                                                    
    1    >  2022-09-17_06-50-36  O     {timeshift-autosnap} {created before upgrade}  
    2    >  2022-09-25_23-40-01  O     {timeshift-autosnap} {created before upgrade}  
    3    >  2022-10-04_07-07-49  O     {timeshift-autosnap} {created before upgrade} 
     
    Found stale mount for device '/dev/nvme0n1p2' at path '/run/timeshift/1623577/backup'
    Unmounted successfully

Create snapshot:

    sudo timeshift --create --btrfs --comments "Stable settings" 

Delete snapshot (use the "name" of the snapshot)

    sudo timeshift --delete --snapshot '2022-09-17_06-50-36' 

Sometimes, timeshift does not delete the snapshot and there are leftovers.  
In that case, you can make use of a few BTRFS commands

    sudo btrfs subvolume list / 

    ID 257 gen 73498 top level 5 path @cache
    ID 258 gen 73499 top level 5 path @log
    ID 2064 gen 73375 top level 5 path timeshift-btrfs/snapshots/2022-10-08_14-56-29/@
    ID 2598 gen 73375 top level 5 path timeshift-btrfs/snapshots/2022-10-04_09-46-54/@
    ID 2606 gen 73375 top level 5 path timeshift-btrfs/snapshots/2022-10-05_18-42-59/@
    ID 2626 gen 73499 top level 5 path @

To delete 2022-09-17_06-50-36 use:

    sudo btrfs subvolume delete /run/timeshift/backup/timeshift-btrfs/snapshots/2022-09-17_06-50-36/@ 

To restore a snapshot, use this:

    sudo timeshift --restore

Here you need to choose the one you want to go back, and enter the
selection.

- Press the ‘y’ key when It asks about reinstalling the GRUB2
  bootloader,
- then press the enter key again
- and finally, press the ‘y’ key to start the system restore…

Because timeshift-GUI might be working fine there, you can also restore
a snapshot using a live USB of Manjaro, Ventoy is highly recommendable
for creating one if you still don’t have it. However, I haven’t tested
restoring a snapshot with a recent image, reason why I said it might
work.


