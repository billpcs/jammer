# jammer
A Bash script to automate the continuous circular deauthentication of all the wifi networks in your reach

## I am not responsible for any misuses of the script 
Keep in mind that it is generally illegal to use the script at your neihborhood

It is designed for pen-testing purposes

It has only been tested on my two machines, so there may still be bugs that can even cause data loss

### That's why I suggest you take a good look at the code before you execute it

There will be updates as soon as I fix something or make a nice improvement

Not that anyone will see this

```
Jammer v0.3


Usage: jammer [OPTION] ... 
Jam Wifi Networks That Your Wireless Card Can Reach.

 -d, --deauths: Set the number of deauthentications for each station. Default is 10
 -y, --yes: Make 'Yes' the answer for everything the script asks
 -s, --endless: When reaching the end of the list, start again
 -f, --whitelist: A file with ESSID's to ignore during the attack
 -k, --keep: Keep the scan files after the script ends
 -n, --name: Choose the names the scan files are saved as
 -e, --ethernet: Set the name for the ethernet interface. Default is 'eth0'
 -w, --wireless: Set the name for the wireless interface. Default is 'wlan0'
 -h, --help: Show this help message
 ```
 Looking at this help message a suggested way to call the script is
 ```
 $ sudo ./jammer -y -s -d 20 -f whitelist.txt
 ```
 
 
