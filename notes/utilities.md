## Taking Screenshots

Cute small utility for taking screenshots directly from the termianl is scrot.

On Debian based system, you can install it in the following way:

    sudo apt install scrot

Minimal example:

    scrot screenshot.png

Some more options:

    scrot -b -d 3 screenshot.png -e 'mv $f ~/Desktop/'

* The -b option instructs the program to include the window borders in the screenshot.
* The -d option specifies a 3 second delay.
* The command -e 'mv $f ~/Pictures/' instructs the utility to save the screenshot to the *~/Pictures* directory. 