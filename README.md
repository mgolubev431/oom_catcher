# Out of memory catcher script

This script starts a stress test of the server, and ends when OOM is triggered. The script is configured to check the syslog after each iteration to see if the first oom entry appears after the script starts.

This can be useful, for example, when building and testing alert rules in Grafana to catch OOM-killers in the syslog.
```
#!/bin/bash

# Get the current time in timestamp format
start_time=$(date +%s)

# Start an endless loop
while true
do
    # Check that a certain amount of time has passed since the script was launched (for example, 30 seconds)
    current_time=$(date +%s)
    elapsed_time=$((current_time - start_time))

    if [ $elapsed_time -ge 30 ]; then
        # Check the log for entries from OOM-killer
        if grep -q "oom-killer" /var/log/syslog; then
            echo "OOM killer is activated. Stop the script."
            break  # Break the loop if an entry from OOM-killer is found
        fi
    fi

    # Start the stress process
    stress --vm 1 --vm-bytes 128M --vm-hang 0 &

    sleep 3  # Wait 3 seconds before next iteration
done

# After exiting the loop, kill all processes launched by the script
pkill -f stress
```
