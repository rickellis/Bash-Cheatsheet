# Bash-Cheatsheet
A collection of useful code for Bash.


# Variables

Declare a variable and set its value, but only if the variable
is empty or undefined. It uses indirect parameter expansion. The ! means set the variable name, not just the value.

    # Required arguments:
    #   1. Variable name
    #   2. Variable value
    #
    # Usage:
    #
    #   setvar foo bar
    #
    # In the above example, if a variable named foo
    # exists and is not empty it will be set to bar.
    # Otherwise its value will not be altered.
    setvar() {
        eval "${1}=\"${!1:-${2}}\"";
    }


## Paths

#### Find the full path of a script
When a script gets invoked via an alias $PWD returns the directory where the alias resides, not the script being called. To find the full canonical path to the current script use:

    BASEPATH=$(dirname $0)

    echo "$BASEPATH"


#### To remove the trailing slash from a directory:

    if [[ $DIR =~ .*/$ ]]; then
        DIR="${DIR:0:-1}"
    fi

#### Extract the filename from the path

    filename="${filepath##*/}"

#### Remove the file extension, leaving only the name:
    
    name="${filename%%.*}"


# Determine if a package is installed

This looks for the existence of its shell command:

    if [[ $(type curl) ]]; then
        echo " Curl installed!"
    fi

For other app types use command with the -v flag to prevent the application from being launched:

    if command -v inkscape &>/dev/null; then
        echo "Inkscape installed!"
    fi


## Arrays

#### Read the each line of a file into an array:

    # -t means remove trailing newlines

    readarray -t MYARRAY < /path/to/filename

#### Read a directory and put all filenames with a particular extension into an array:

    i=0
    declare -A MY_ARRAY
    for file in /path/to/*.txt; do
        if [[ -f "$file" ]]; then
            MY_ARRAY[${i}]="$file"
            ((i++))
        fi
    done

#### Loop through an array in reverse order

    MYARRAY=('one' 'two' 'three' 'four' 'five' 'six')

    for (( i=${#MYARRAY[@]}-1 ; i>=0 ; i-- )) ; do
        echo "${MYARRAY[i]}"
    done


#### Count how many items are in an array:

    MYARRAY=(1, 2, 3, 4, 5, 6)

    count="${#MYARRAY[@]}"

    echo "$count"

#### Validate an array

    if [[ ${#MY_ARRAY[@]} -eq 0 ]]; then
        echo "Array empty"
    fi


## Loops

#### Iterate through all files in a directory that have a certain extension

    for filepath in /path/to/*.png; do
        echo "$filepath"
    done


#### Modulus to alternate in a loop

    for ((i=1; i<=10; i++))
    {
        if (( $i%2 == 0 )); then
            echo "Even"
        else
            echo "Odd"
        fi
    }

## Regular Expressions

#### Remove newlines:

    MYVAR=${MYVAR//$'\n'/}


#### Remove spaces

Using sed

    str="Hello, my name is Foo"

    str=$(echo $str | sed "s/[ ]//g")

    echo $str # Hello,mynameisFoo

Using parameter expansion

    str="Hello, my name is Foo"

    str=${str// /}

    echo $str # Hello,mynameisFoo


#### Parameter expansion

    str="John, at the Bar is a friend-of-mine"

    str=${str,,}    # lowercase
    str=${str//,/}  # remove commas
    str=${str//-/}  # remove dashes
    str=${str// /}  # remove spaces

    echo $str # johnatthebarisafriendofmine


#### Remove everything after a certain character

    str="Nancy Davis # Will bring jello to the potluck"

    str=$(echo $str | sed "s/#.*//")

    echo $str # Nancy Davis


#### Change case

    MYVAR=${MYVAR,,} # Lowercase all characters
    MYVAR=${MYVAR^^} # Uppercase all characters


#### Truncate a string to x characters

    MYSTRING="foobarbaz"

    $MYSTRING="${MYSTRING:0:3}" # returns "foo"
    $MYSTRING="${MYSTRING:6:3}" # returns "baz"


#### Does the variable contian only integers?

    if [[ $MYVAR =~ ^?[0-9]+$ ]]; then
        echo "Only integers allowed!"
    fi

#### Extract the hex color value from a string:

    MYSTRING=$(echo "$MYSTRING" | sed 's/.*[[:space:]]\(#[a-zA-Z0-9]\+\)[[:space:]].*/\1/')

#### Escape certain characters:

    MYVAR=${MYVAR//&/\\&} # Excape ampersands


#### To find a value in a string:

    SEARCHFOR="bar"

    if echo "$MYSTRING" | egrep -q "(^|\s)${SEARCHFOR}($|\s)"; then
        echo "$SEARCHFOR exists!"
    fi

#### Delete everything until the first slash

    MYVAR="this is some text /path/to/foo.jpg"
    
    MYVAR=$(echo $MYVAR | sed 's/^[^/]*\///g')

#### Delete everything after a colon:

    MYVAR="foo:bar"

    MYVAR=$(echo $MYVAR | sed "s/[:].*//")




## Math

#### To perform arithmatic use:

    echo $(( 5+2 ))

#### To assign the result to a variable

    n=5
    n=$(( n-1 ))

#### Generate a random number between two values:

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

#### Fixes echo for POSIX compatibility

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


#### Change the input field separator from a space to a null

    IFS=$'\n'

#### Get the returned value of the last command

    ls >/dev/null 2>&1

    echo $?  # Returns 0

    fofofofo >/dev/null 2>&1

    echo $?  # Returns 127 (command not found)