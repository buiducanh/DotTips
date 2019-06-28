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
