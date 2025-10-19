# vagrant-suspend-resume
Suspend/resume all Vagrant boxes on system shutdown/startup


this is an update of https://www.ollegustafsson.com/vagrant-suspend-resume/

🧩 1. Overview

We’ll add:

A small state file: /var/lib/vagrant-boxes/state.json

During stop: store the list of boxes we suspended.

During start: resume only boxes listed in that file.



⚙️ 2. /usr/local/bin/vagrant-boxes


```bash
#!/bin/bash
# vagrant-boxes: suspend/resume all Vagrant boxes for all users
# -------------------------------------------------------------
# Logs to:  /var/log/vagrant-boxes.log
# State:    /var/lib/vagrant-boxes/state.json

LOGFILE="/var/log/vagrant-boxes.log"
STATE_DIR="/var/lib/vagrant-boxes"
STATE_FILE="$STATE_DIR/state.json"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

mkdir -p "$STATE_DIR"
chmod 755 "$STATE_DIR"

exec >>"$LOGFILE" 2>&1
echo "[$DATE] Action: $1"

validShells=$(grep -v "#" /etc/shells | tr '\n' '|')
userList=$(grep -E "$validShells" /etc/passwd | awk -F ':' '{ print $1 }')

case "$1" in
  start)
    echo "[$DATE] Starting/resuming Vagrant boxes from last shutdown..."
    if [ ! -s "$STATE_FILE" ]; then
      echo "[$DATE] No previous suspended boxes found. Nothing to resume."
      exit 0
    fi

    while IFS="|" read -r user vm; do
      if [ -d "$vm" ]; then
        echo "[$DATE] -> Resuming $vm for $user"
        cd "$vm" && su -c "timeout 120 vagrant up" $user
      fi
    done < "$STATE_FILE"

    # Clear state after successful startup
    > "$STATE_FILE"
    echo "[$DATE] Done starting boxes."
    ;;

  stop)
    echo "[$DATE] Suspending running Vagrant boxes..."
    > "$STATE_FILE" # reset
    for user in $userList; do
      su -c "vagrant global-status --prune 2>/dev/null" $user | grep running | awk '{print $5}' | while read -r vm; do
        [ -d "$vm" ] || continue
        echo "[$DATE] -> Suspending $vm for $user"
        cd "$vm" && su -c "timeout 120 vagrant suspend" $user
        echo "$user|$vm" >> "$STATE_FILE"
      done
    done
    echo "[$DATE] Done suspending boxes. Stored state in $STATE_FILE"
    ;;

  *)
    echo "Usage: $0 {start|stop}" >&2
    exit 1
    ;;
esac

exit 0
```

