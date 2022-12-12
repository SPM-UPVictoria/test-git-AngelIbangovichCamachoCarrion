# test-git-AngelIbangovichCamachoCarrion


#!/bin/bash  
. /home/angeli/Escritorio/test-git-AngelIbangovichCamachoCarrion/5

6
validfloat()
{
  fvalue="$1"

  if [ ! -z $(echo $fvalue | sed 's/[^.]//g') ] ; then

    decimalPart="$(echo $fvalue | cut -d. -f1)"

    fractionalPart="${fvalue#*\.}"

    if [ ! -z $decimalPart ] ; then
      if ! validint "$decimalPart" "" "" ; then
        return 1
      fi 
    fi

    if [ "${fractionalPart%${fractionalPart#?}}" = "-" ] ; then
      echo "Invalid floating-point number: '-' not allowed \
        after decimal point" >&2
      return 1
    fi 
    if [ "$fractionalPart" != "" ] ; then 
      if ! validint "$fractionalPart" "0" "" ; then
        return 1
      fi
    fi
  else 
    if [ "$fvalue" = "-" ] ; then
      echo "Invalid floating-point format." >&2 ; return 1
    fi
    if ! validint "$fvalue" "" "" ; then
      return 1
    fi
  fi

  return 0
}

if validfloat $1 ; then
  echo "$1 is a valid floating-point value"
fi

exit 0





8

#!/bin/bash
echo "$*" | awk '{ printf "%s", $0 }'

11

#!/bin/bash

initializeANSI()
{
  esc="\033"   

  blackf="${esc}[30m";   
  redf="${esc}[31m";    
  greenf="${esc}[32m"
  yellowf="${esc}[33m"   
  bluef="${esc}[34m";   
  purplef="${esc}[35m"
  cyanf="${esc}[36m";    
  whitef="${esc}[37m"
  
  blackb="${esc}[40m";   redb="${esc}[41m";    greenb="${esc}[42m"
  yellowb="${esc}[43m"   blueb="${esc}[44m";   purpleb="${esc}[45m"
  cyanb="${esc}[46m";    whiteb="${esc}[47m"

  boldon="${esc}[1m";    boldoff="${esc}[22m"
  italicson="${esc}[3m"; italicsoff="${esc}[23m"
  ulon="${esc}[4m";      uloff="${esc}[24m"
  invon="${esc}[7m";     invoff="${esc}[27m"

  reset="${esc}[0m"
}

13

#!/bin/bash
biggest=100                   
guess=0                       
guesses=0                    
number=$(( $RANDOM % $biggest + 1 ))    
echo "Guess a number between 1 and $biggest"

while [ "$guess" -ne $number ] ; do
  /bin/echo -n "Guess? " ; read guess
  if [ "$guess" -lt $number ] ; then
    echo "... bigger!"
  elif [ "$guess" -gt $number ] ; then
    echo "... smaller!"
  fi
  guesses=$(( $guesses + 1 ))
done

echo "Right!! Guessed $number in $guesses guesses."

exit 0

16

#!/bin/bash

archivedir="$HOME/.deleted-files"
realrm="$(which rm)"
copy="$(which cp) -R"

if [ $# -eq 0 ] ; then 
  exec $realrm  
fi

flags=""

while getopts "dfiPRrvW" opt
do
  case $opt in
    f ) exec $realrm "$@"     ;; 
    * ) flags="$flags -$opt"  ;;  
  esac
done
shift $(( $OPTIND - 1 ))

if [ ! -d $archivedir ] ; then
  if [ ! -w $HOME ] ; then
    echo "$0 failed: can't create $archivedir in $HOME" >&2 
    exit 1
  fi
  mkdir $archivedir
  chmod 700 $archivedir# a little bit of privacy, please
fi

for arg 
do
  newname="$archivedir/$(date "+%S.%M.%H.%d.%m").$(basename "$arg")"
  if [ -f "$arg" -o -d "$arg" ] ; then
    $copy "$arg" "$newname"	
  fi
done

exec $realrm $flags "$@"   

19

#!/bin/bash

#TENEMOS QUE EJECUTAR CON ROOT#

locatedb="/var/tmp/locate_19code.db"

if [ "$(whoami)" != "haydo" ] ; then
  echo "Must be root to run this command." >&2
  exit 1
fi

find / -print > $locatedb

exit 0

22

#!/bin/bash
rememberfile="$HOME/.remember"

if [ $# -eq 0 ] ; then
  echo "Enter note, end with ^D: "
  cat - >> $rememberfile
else
  echo "$@" >> $rememberfile
fi

exit 0


29

#!/bin/bash

width=72

if [ ! -r "$1" ] ; then
  echo "Cannot read file $1" >&2
  echo "Usage: $0 filename" >&2; exit 1
fi

while read input
do
   if [ ${#input} -gt $width ] ; then
     echo "$input" | fmt 
   else
     echo "$input"
   fi
done < $1

exit 0


30

#!/bin/bash
flags=""
realquota="$(which quota)"
while [ $# -gt 0 ]
do
  case $1
  in
    --help)  echo "Usage: $0 [--group --verbose --quiet -gvq]" >&2
                       exit 1 ;;
    --group )  flags="$flags -g";       shift ;;
    --verbose)  flags="$flags -v";   shift ;;
    --quiet)  flags="$flags -q";       shift ;;
    --)  shift;           break ;;
    *)  break;         
  esac
done
exec $realquota $flags "$@"


31

#!/bin/bash

/bin/echo -n "User account: "
read account

if [ -z $account ] ; then
  exit 0;       
fi

if [ -z "$1" ] ; then
  /bin/echo -n "Remote host: "
  read host
  if [ -z $host ] ; then
    exit 0
  fi
else
  host=$1
fi
exec sftp -C $account@$host

34

#!/bin/bash
Z="compress"     gz="gzip"     bz="bzip2"
Zout="/tmp/bestcompress.$$.Z"
gzout="/tmp/bestcompress.$$.gz"
bzout="/tmp/bestcompress.$$.bz"
skipcompressed=1
if [ "$1" = "-a" ] ; then
  skipcompressed=0  ; shift
fi

if [ $# -eq 0 ]; then
  echo "Usage: $0 [-a] file or files to optimally compress" >&2; exit 1
fi

trap "/bin/rm -f $Zout $gzout $bzout" EXIT

for name in "$@"
do 
  if [ ! -f "$name" ] ; then 
    echo "$0: file $name not found. Skipped." >&2
    continue
  fi

  if [ "$(echo $name | egrep '(\.Z$|\.gz$|\.bz2$)')" != "" ] ; then
    if [ $skipcompressed -eq 1 ] ; then
      echo "Skipped file ${name}: It's already compressed." 
      continue
    else
      echo "Warning: Trying to double-compress $name" 
    fi
  fi

  $Z  < "$name" > $Zout  &
  $gz < "$name" > $gzout &
  $bz < "$name" > $bzout &
  wait 

  smallest="$(ls -l "$name" $Zout $gzout $bzout | \
     awk '{print $5"="NR}' | sort -n | cut -d= -f2 | head -1)"

  case "$smallest" in
     1 ) echo "No space savings by compressing $name. Left as is."
         ;;
     2 ) echo Best compression is with compress. File renamed ${name}.Z
         mv $Zout "${name}.Z" ; rm -f "$name"
         ;;
     3 ) echo Best compression is with gzip. File renamed ${name}.gz
         mv $gzout "${name}.gz" ; rm -f "$name"
         ;;
     4 ) echo Best compression is with bzip2. File renamed ${name}.bz2
         mv $bzout "${name}.bz2" ; rm -f "$name"
  esac
done

exit 0

38

#!/bin/bash
tempfile="/tmp/available.$$"

trap "rm -f $tempfile" EXIT

cat << 'EOF' > $tempfile
    { sum += $4 }
END { mb = sum / 1024
      gb = mb / 1024
      printf "%.0f MB (%.2fGB) of available disk space\n", mb, gb
    }
EOF

df -k | awk -f $tempfile
exit 0

39
#!/bin/bash

#TENEMOS QUE EJECUTARLO CON ROOT#

locatedb="/var/locate.db"
slocatedb=".slocatedb"

if [ "$(id -nu)" != "root" ] ; then
  echo "$0: Error: You must be root to run this command." >&2
  exit 1
fi

if [ "$(grep '^nobody:' /etc/passwd)" = "" ] ; then
  echo "$0: Error: you must have an account for user 'nobody'" >&2
  echo "to create the default slocate database." >&2; exit 1
fi

cd /            
su -fm nobody -c "find / -print" > $locatedb 2>/dev/null
echo "building default slocate database (user = nobody)"
echo ... result is $(wc -l < $locatedb) lines long.

for account in $(cut -d: -f1 /etc/passwd)
do
  homedir="$(grep "^${account}:" /etc/passwd | cut -d: -f6)"

  if [ "$homedir" = "/" ] ; then
    continue   
  elif [ -e $homedir/$slocatedb ] ; then
    echo "building slocate database for user $account"
    su -m $account -c "find / -print" > $homedir/$slocatedb \
     2>/dev/null
    chmod 600 $homedir/$slocatedb
    chown $account $homedir/$slocatedb
    echo ... result is $(wc -l < $homedir/$slocatedb) lines long.
  fi
done

exit 0


41

#!/bin/bash

#TENEMOS QUE EJECUTARLO CON ROOT#

homedir="/home"         
secs=10                 

if [ -z $1 ] ; then
  echo "Usage: $0 account" >&2 ; exit 1
elif [ "$(id -un)" != "root" ] ; then
  echo "Error. You must be 'root' to run this command." >&2; exit 1
fi

echo "Please change account $1 password to something new."
passwd $1
if who|grep "$1" > /dev/null ; then

  for tty in $(who | grep $1 | awk '{print $2}'); do

    cat << "EOF" > /dev/$tty

*************************************************************
URGENT NOTICE FROM THE ADMINISTRATOR:

This account is being suspended, and you are going to be logged out 
in $secs seconds. Please immediately shut down any processes you 
have running and log out.

If you have any questions, please contact your supervisor or 
John Doe, Director of Information Technology.
*************************************************************
EOF
  done

  echo "(Warned $1, now sleeping $secs seconds)"

  sleep $secs

  jobs=$(ps -u $1 | cut -d\  -f1)

  kill -s HUP $jobs                  
  sleep 1                           
  kill -s KILL $jobs > /dev/null 2>1 

  echo "$1 was logged in. Just logged them out."
fi

chmod 000 $homedir/$1

echo "Account $1 has been suspended."

exit 0


54

#!/bin/bash

if [ $# -eq 0 ] ; then
  echo "Usage: $0 [-d|-i|-x] url"  >&2
  echo "-d=domains only, -i=internal refs only, -x=external only" >&2
  exit 1
fi

if [ $# -gt 1 ] ; then
  case "$1" in
    -d) lastcmd="cut -d/ -f3 | sort | uniq"
        shift
        ;;
    -r) basedomain="http://$(echo $2 | cut -d/ -f3)/"
        lastcmd="grep \"^$basedomain\" | sed \"s|$basedomain||g\" | sort |\
	     uniq"
        shift
        ;;
    -a) basedomain="http://$(echo $2 | cut -d/ -f3)/"
        lastcmd="grep -v \"^$basedomain\" | sort | uniq"
        shift
        ;;
     *) echo "$0: unknown option specified: $1" >&2; exit 1
  esac
else
  lastcmd="sort | uniq"
fi

#El Lynx al ser un navegador que funciona completamente en modo texto y se utiliza a través 
#de la línea de comandos, este software puede ser una buena solución para sistemas basados ​​en 
#consola, o incluso en servidores en caso de que sea necesario consultar la Internet.

lynx -dump "$1" | \
sed -n '/^References$/,$p' | \
  grep -E '[[:digit:]]+\.' | \
  awk '{print $2}' | \
  cut -d\? -f1 | \
eval $lastcmd

exit 0

56

#!/bin/bash
baseURL="http://www.city-data.com/zips"

/bin/echo -n "ZIP code $1 is in "

curl -s -dump "$baseURL/$1.html" | \
   grep -i '<title>' | \
   cut -d\( -f2 | cut -d\) -f1

exit 0

#!/bin/bash
if [ $# -ne 1 ]; then
  echo "Usage: $0 <zipcode>"
  exit 1
fi

apikey="b0304b43b2e7cd23" 

weather=`curl -s \
    "https://api.wunderground.com/api/$apikey/conditions/q/$1.xml"`
state=`xmllint --xpath \
     //response/current_observation/display_location/full/text\(\) \
     <(echo $weather)`
zip=`xmllint --xpath \
     //response/current_observation/display_location/zip/text\(\) \
     <(echo $weather)`
current=`xmllint --xpath \
     //response/current_observation/temp_f/text\(\) \
     <(echo $weather)`
condition=`xmllint --xpath \
     //response/current_observation/weather/text\(\) \
     <(echo $weather)`

echo $state" ("$zip") : Current temp "$current"F and "$condition" outside." 

