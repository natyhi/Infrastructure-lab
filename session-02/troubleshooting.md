## Ubuntu Installation and Password Reset Troubleshooting

Ubuntu did not install properly on my first attempt. To troubleshoot the issue, I removed all existing files and started the installation process again from scratch.

One thing that confused me was why I needed to remove the installation media before the final reboot. After doing some research, I found that Ubuntu requires this step for two reasons:

* It prevents the system from booting back into the installer and creating an installation loop.
* It forces the machine to boot from the internal storage where Ubuntu was installed.

After researching this, I understood that the installation media needed to be removed before rebooting. It may boot back into the installer instead of the newly installed Ubuntu system.

After the installation, I needed to reset the root password. I restarted the VPS using the **Reset** option from the Machine toolbar. During boot, I pressed **e** to edit the boot parameters.

I located the parameter:

`ro`

and replaced it with:

`rw init=/bin/bash`

I then pressed **Ctrl + X** to boot with the modified parameters.

Once the system loaded into the shell, I entered:

`passwd root`

to create a new root password. After updating the password, I used:

`reboot -f`

to force a reboot and apply the changes.

I repeated this process several times because I continued receiving a **"Login incorrect"** error. At that point, I became frustrated because I had successfully changed the password, but I still could not access the system. I eventually stopped troubleshooting for the day and came back to it the following morning with a fresh perspective.

The next day, while reviewing each step again, I discovered that the issue was not Ubuntu, the installation process, or the password reset procedure. The actual problem was that I had been entering my username incorrectly during login.

Once I identified the typo, I reinstalled Ubuntu and repeated the setup process. This time, everything worked as expected, and I was able to log in successfully.

### Key Takeaway

This lab reminded me that troubleshooting is not always about finding a complex technical issue. Sometimes the root cause is a simple human error. Before assuming the operating system, configuration, or installation is broken, it is important to verify the basics, including usernames, passwords, and command syntax.
