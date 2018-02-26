# Bash-Cheatsheet
A collection of useful code for Bash.


## Paths

When a script gets invoked via an alias $PWD returns the directory where the alias resides, not the script being called. To find the full canonical path to the current script use:

    BASEPATH=$(dirname -- $(readlink -fn -- "$0"))

    echo "$BASEPATH"


To remove the trailing slash from a directory:

    if [[ $DIR =~ .*/$ ]]; then
        DIR="${DIR:0:-1}"
    fi

Does a directory exist?

if [ -d $DIR ]; then
    echo "Directory exists!"
fi


Extract the filename from the path

    filename="${filepath##*/}"

Remove the file extension, leaving only the name:
    name="${filename%%.*}"


# Determine if a package is installed

Is a package installed? This looks for the existence of its shell command:

    if [[ $(type curl) ]]; then
        echo " Curl installed!"
    fi

For other app types use command with the -v flag to prevent the application from being launched:

    if command -v inkscape &>/dev/null; then
        echo "Inkscape installed!"
    fi


## Arrays

Read a directory and put all filenames with a particular extension into an array:

    i=0
    declare -A MY_ARRAY
    for file in /path/to/*.txt; do
        if [ -f "$file" ]; then
            MY_ARRAY[${i}]="$file"
            ((i++))
        fi
    done

Count how many items are in an array:

    count="${#MY_ARRAY[@]}"

    echo "$count"

If no items are in an array issue warming

    if [ ${#MY_ARRAY[@]} -eq 0 ]; then
        echo "Array empty"
    fi


## Loops

Iterate through all files in a directory that have a certain extension

    for filepath in /path/to/*.png
        do
        echo "$filepath"
    done


## Regular Expressions

Remove newlines:

    MYVAR=${MYVAR//$'\n'/}

Change case

    MYVAR=${MYVAR,,} # Lowercase all characters
    MYVAR=${MYVAR^^} # Uppercase all characters


Truncate a string to 7 characters

    $MYSTRING="${MYSTRING:0:7}"


Does the variable contian only integers?

    if [[ $MYVAR =~ ^?[0-9]+$ ]]; then
        echo "Only integers allowed!"
    fi


Extract the hex color value from a string:

    MYSTRING=$(echo "$MYSTRING" | sed 's/.*[[:space:]]\(#[a-zA-Z0-9]\+\)[[:space:]].*/\1/')

Escape certain characters:

    MYVAR=${MYVAR//&/\\&} # Excape ampersands


To find a value in a string:

    SEARCHFOR="bar"

    if echo "$MYSTRING" | egrep -q "(^|\s)${SEARCHFOR}($|\s)"; then
        echo "$SEARCHFOR exists!"
    fi


## Math

To perform arithmatic use:

    echo $(( 5+2 ))

To assign the result to a variable

    n=5
    n=$(( n-1 ))


Generate a random number between two values:

    RND=$(shuf -i 1-20 -n 1)
    echo "$RND"


## Killing a process

By name:

    pkill foo


To look up a process by name

    pgrep foo

To look up all process IDs associated with a package and kill them:

    PID=$(pgrep foo)
    if ! [ -z "$PID" ]; then
        for id in $PID; do
            kill "$id"
        done
    fi


## Misc.

Fixes echo for POSIX compatibility

    echo() (
        fmt=%s
        end=\\n 
        IFS=" "

        while [ $# -gt 1 ] ; do
            case "$1" in
                [!-]*|-*[!ne]*) break       ;;
                *ne*|*en*)      fmt=%b
                                end=''      ;;
                *n*)            end=''      ;;
                *e*)            fmt=%b      ;;
            esac
            shift
        done
        printf "$fmt$end" "$*"
    )
