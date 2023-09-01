# Run Windows app on Ubuntu Desktop

Rather than starting the VM and launching the app from the Windows
Desktop:

- Just directly run the Windows app I want.
- in seamless mode
- So it acts like any other app on my Ubuntu desktop.
- Close the app, the vm stops.

Here's what to do:

- Run the Windows VM
- Log in to windows, but don't run anything
- Switch to seamless mode Host + L
- Save the machine's state

Now we can get full command line control over the virtual machine with
the following commands:

```
# Start the virtual machine from seamless save state
VBoxManage startvm "<Name_of_VM>"
# or (for the Qt frontend)
VirtualBox --startvm "<Name_of_VM>"

# Run an application in the VM
VBoxManage --nologo guestcontrol "<Name_of_VM>" run --exe "C:\full\path\to\program.exe" --username windowsuser --password password --wait-stdout

# Terminate VM in save state
VBoxManage controlvm "Name_of_VM" savestate
```

Put these in a script to enjoy seamless Windows application windows on
your Ubuntu desktop.

In case you have set up a passwordless Windows logon this will not work.
See in the Virtual Box Manual for limitations and how to configure
Windows to get it working.

Also, to use accounts without or with an empty password, the guest's group policy must be changed.
To do so, open the group policy editor on the command line by typing gpedit.msc,
open the key
Computer Configuration\Windows Settings\Security Settings\Local Policies\Security Options
and change the value of Accounts: Limit local account use of blank passwords to console logon only to Disabled.

On operating systems without the Group Policy Editor (gpedit.msc), such
as Home editions of Windows, creating a DWORD at the registry key
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\limitblankpassworduse
and setting it to zero will achieve the same effect, according to this
answer.
