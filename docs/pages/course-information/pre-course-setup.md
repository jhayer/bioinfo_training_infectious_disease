# Setup your work environment

## Unix system

For this course we need a Unix system, see below how to install that on your machine.

??? abstract "Unix installation"
    ### Setup for Mac / Linux users

    You are lucky, using a UNIX based system as always been ideal for
    bioinformatics.  

    ### Setup for Windows users

    Using a Windows computer for bioinformatic work has sadly not been ideal most of
    the time, but large advanced in recent years have made this quite feasible
    through the Windows 10 *Linux subsystem*. This is the only setup for Windows
    users that we allow for participants of this course, as all the material has
    been created and tested to work on Unix-based systems.

    Using the Linux subsystem will give you access to a full command-line bash shell
    based on Linux on your Windows 10 PC. For the difference between the Linux Bash
    Shell and the PowerShell on Windows 10, see *e.g.* [this article](
    https://searchitoperations.techtarget.com/tip/On-Windows-PowerShell-vs-Bash-comparison-gets-interesting).

    Install Bash on Windows 10, follow the instructions at *e.g.* one of these
    resources:

    - [Installing the Windows Subsystem and the Linux Bash](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
    - [Installing and using Linux Bash on Windows](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/)
    - [Installing Linux Bash on Windows](https://itsfoss.com/install-bash-on-windows/)

    !!! note

        If you run into error messages when trying to download files through the Linux
        shell (_e.g._ `curl:(6) Could not resolve host`) then try adding the Google
        nameserver to the internet configuration by running `sudo nano /etc/resolv.conf`
        then add `nameserver 8.8.8.8` to the bottom of the file and save it.

    !!! tip

        You can find the directory where the Linux distribution is storing all its
        files by typing `explorer.exe .`. This will launch the Windows File Explorer
        showing the current Linux directory.
        Alternatively, you can find the Windows C drive from within the bash shell
        Linux terminal by navigating to `/mnt/c/`.

## Container technology (optional)

Using container engines such as Docker or Apptainer allows you to work on any computer system in a reproducible environment.
Apptainer is not straightforward to use on Windows or macOS, so Docker is often preferred on local machines, while Apptainer is generally used on HPC systems.
Container engines are often already installed on HPCs, but installing Docker locally would give you the possibility to work on your own machine.

??? abstract "Docker installation"
    <!-- start-install-docker -->
    ## Installing Docker

    Installing Docker (specifically Docker Desktop) is quite straightforward on Mac, Windows and Linux distributions. Note that Docker runs as root, which means that you have to have `sudo` privileges on your computer in order to install or run Docker. When you have finished installing docker, regardless of which OS you are on, please type `docker --version` to verify that the installation was successful.

    !!! note "Docker for older versions of OSX/Windows"
        The latest version of Docker may not work if you have an old version of either OSX or Windows. You can find older Docker versions that may be compatible for you if you go to [https://docs.docker.com/desktop/](https://docs.docker.com/desktop/) and click "Previous versions" in the left side menu.

    ### MacOS

    Go to [mac-install](https://docs.docker.com/desktop/install/mac-install/) section of the Docker documentation and select the download option that is suitable for your computer’s architecture (i.e. if you have an Intel chip or a Apple silicon chip). This will download a `dmg` file - click on it when it’s done to start the installation. This will open up a window where you can drag the Docker.app to Applications. Close the window and click the Docker app from the Applications menu. Now it’s basically just to click “next” a couple of times and we should be good to go. You can find the Docker icon in the menu bar in the upper right part of the screen.

    ### Linux

    Go to the [linux-install](https://docs.docker.com/desktop/install/linux/) section of the Docker documentation and make sure that your computer meets the system requirements. There you can also find instructions for different Linux distributions in the left sidebar.

    ### Windows

    Go to the [windows-install](https://docs.docker.com/desktop/install/windows-install/#download-docker-for-windows) section of the Docker documentation and select the download option that is suitable for your computer’s architecture (i.e. if you have an Intel chip (x86) or ARM). Once the download is complete, execute the file and follow the [instructions](https://docs.docker.com/desktop/install/windows-install/#install-docker-desktop-on-windows). You can now start Docker from the Start menu. You can search for it if you cannot find it; the Docker whale icon should appear in the task bar.

    You will probably need to enable integration with the Linux subsystem, if you haven’t done so during the installation of Docker Desktop. Right-click on the Docker whale icon in the task bar and select Settings. Choose *Resources* and select *WPS integration*. Enable integration with the Linux subsystem and click *Apply & Restart*; also restart the Linux subsystem.

    <!-- end-install-docker -->