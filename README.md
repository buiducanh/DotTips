# See Processes with headers
```
 ps -ef | grep -E "\sTIME\s|tmux"
 ```

# so scripts can use aliases after sourcing this file
```
shopt -s expand_aliases
```

```
function getPIDHoldingPort() {
  port="$1"
  retval=$(sudo lsof -n -i :$port | grep -iE "LISTEN|PID" | tail -n +2 | sort -u | awk '{print $2}')
  echo "$retval"
}
```
```
function killWait() {
  while kill -0 $1 &>/dev/null; do
    sleep 1
  done
}

function getDirOf() {
  tmp="$1"
  echo "${tmp%/*}"
}

# kill all processes that match this name
function killAll() {
    local pid
    local preview
    preview=$(ps aux | grep -iE "$1" | grep -vE "$2")
    pid=$(echo "$preview" | awk '{print $2}')
    echo "Killing these processes...:"
    echo "$preview"
    read -p "Continue (y/n)? " choice
    case "$choice" in
      y|Y ) ;;
      n|N ) echo "User aborted"; return 1;;
      * ) echo "invalid"; return 1;;
    esac

    if [ "x$pid" != "x" ]
    then
        echo "$pid" | xargs kill
    fi
    echo "Finished killing processes"
    echo "Results of: ps aux | grep -iE \"$1\" | grep -v \"$2\""
    ps aux | grep -iE "$1" | grep -v "$2"
    return 0
}
```

# Read arg, and read arg array
```
  local OPTIND
  getopts "h:" OPTION
  echo INPUT: $*, OPTION: $OPTION, OPTARG: $OPTARG
  IFS=',' read -ra HOSTS <<< "$OPTARG"
  for i in "${HOSTS[@]}"; do
```

# Wait for port to be available with timeout
```
function waitForPort() {
  echo "Wait until service can serve request at port $1"
  trap "return 1" SIGINT SIGTERM
  local status
  timeout 600 bash -c -- \
    'source ~/.bash_aliases; \
    port=""; \
    while [[ -z "${port// }" ]]; do \
        port=$(getPIDHoldingPort "$1"); \
        echo -n "."; \
        i=$(( ++i )); \
        [ $(( i % 10 )) == 0 ] && echo -n "${i}s"
        sleep 0.8s ; \
    done' || status="$?";
  echo
  [[ "$status" = "124" ]] && echo "Timed out while waiting for service" && return 1
  echo "service ready to serve"
  return 0
}
```

# Get background command id
```
      bgcommand &
      bgpid=$!
      trap "cleanUp $bgpid" SIGTERM SIGINT
```

# Parallel
```
      parallel ::: "echo \'run command\';run command" "echo \'run command2\';run command2"

```

# Toggle Comments
```
function toggleLines() {
  sed -i "$1"' s/^##*//w '"$HOME/.sedchangelog" "$2"
  if [ ! -s "$HOME/.sedchangelog" ]; then
    sed -i "$1"' s/^\s*/# /w '"$HOME/.sedchangelog" "$2"
  fi
}
```

# Android serial
```
alias setserial="setAndroidSerial"

function setAndroidSerial() {
  num_devices=$(adb devices | wc -l)
  INDEX=$1
  if [[ -z "$INDEX" ]]; then
    INDEX=0
  fi
  if [[ num_devices -le $((2 + $INDEX)) ]]; then
    echo "No devices attached at index $INDEX"
    return 0
  fi
  echo "Device at index $INDEX chosen"
  linenum=$(($INDEX + 2))
  serial=$(adb devices | sed "$linenum""q;d" | awk '{ print $1 }')
  echo "Setting ANDROID_SERIAL to $serial"

  export ANDROID_SERIAL=$serial

}
```

# Say mac command
```
alias saycommand='say -o ~/command.wav --data-format=I16@16000 --channels=1'
```

# fzf with other command
Prerequisites
```
PS1='[\u@\h \D{%T} \W]\n$ '
```
```
#!/usr/local/fbcode/gcc-5-glibc-2.23/bin/python3.6

import subprocess
import itertools
import re
import sys
import tempfile


PROMPT_CMDLINE = r'^\$ '
PROMPT_STATUSLINE = r'^\[abui@[^ ]+ \d+:\d+\:\d+ .+]'

# Get a paste process ready to go in case we actually do generate a paste
paste_proc = subprocess.Popen(['arc', 'paste'], stdin=subprocess.PIPE,
                              stderr=open('/dev/null', 'w'),
                              encoding='utf-8')

# Get the whole tmux scrollback
panecap = subprocess.run(['tmux', 'capture-pane', '-p', '-S', '-'],
                         stdout=subprocess.PIPE, check=True, encoding='utf-8')

scrollback = panecap.stdout.splitlines()

cmds = []
collect = []
for line in scrollback:
    if re.match(PROMPT_CMDLINE, line):
        collect = [line]
    elif re.match(PROMPT_STATUSLINE, line):
        if len(collect) > 0:
            cmds.append(collect)
        collect = []
    elif len(collect) > 0:
        collect.append(line)

if len(cmds) != 0:
    fzflines = ['{}\t{}'.format(*c) for c in
                zip(itertools.count(), (ls[0] for ls in cmds))]
    # print('Choices:')
    # print(fzflines)
    with tempfile.TemporaryDirectory() as td:
        for idx, cmd in zip(itertools.count(), cmds):
            with open(td + '/{}'.format(idx), 'w') as cf:
                cf.write('\n'.join(cmd))
        fzf = subprocess.Popen(
            ['fzf', '--tac', '--multi', '+s',
             '--preview=cat {}/{}'.format(td, '{1}')],
            stdin=subprocess.PIPE,
            stdout=subprocess.PIPE)
        cmdi, _ = fzf.communicate(input=('\n'.join(fzflines)).encode('utf-8'))
        if fzf.returncode != 0:
            sys.exit(0)
    paste = []
    for line in cmdi.decode('utf-8').splitlines():
        idx = int(line.split('\t')[0])
        print('Pasting output of \'{}\''.format(cmds[idx][0]))
        paste += cmds[idx] + ['']

    paste_proc.stdin.write('\n'.join(paste))
    paste_proc.stdin.close()
    paste_proc.wait()
```

# Alert
```
# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
```

# Color tmux local
```
if [[ -n $TMUX ]]; then
  tmux rename-window "[[ local ]]"
  echo -e "\033]6;1;bg;red;brightness;250\a"
  echo -e "\033]6;1;bg;green;brightness;128\a"
  echo -e "\033]6;1;bg;blue;brightness;114\a"
fi
```

# Color remote tmux
```
if [[ $HOSTNAME = $oTherHOST ]]; then
  if [[ $TMUX ]]; then
    tmux rename-window "[[ other ]]"
  fi
  echo -e "\033]6;1;bg;red;brightness;227\a"
  echo -e "\033]6;1;bg;green;brightness;143\a"
  echo -e "\033]6;1;bg;blue;brightness;10\a"
fi

if [[ $HOSTNAME = $other1HOST ]]; then
  if [[ $TMUX ]]; then
    tmux rename-window "[[ ohter1 ]]"
  fi
  echo -e "\033]6;1;bg;red;brightness;57\a"
  echo -e "\033]6;1;bg;green;brightness;197\a"
  echo -e "\033]6;1;bg;blue;brightness;77\a"
fi
```
# History related
```
# avoid duplicates
export HISTCONTROL=ignoredups:erasedups
# When the shell exits, append to the history file instead of overwriting it
shopt -s histappend
HISTSIZE=130000 HISTFILESIZE=-1
# After each command, append to the history file and reread it
export PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}history -a; history -c; history -r"
```
# Sed prints matched line with original line
```
xargs sed -ne 's/\(.\+\)networkStatus_\(.\+\)/\1networkStatus_\2\n\1networkStatusComponent_\2/p' < list_of_files.txt
```
