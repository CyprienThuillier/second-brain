### Processes

Each process has his own id called a `PID`.
- `ps` to list all the running processes
 - `ps aux` to see the processes run by other user or that don't run from a session

Use `top` (or `btop` for a prettier version) to see real-time statistics about the running processes.

![[btop.png]]
*BTOP interface - screenshot*

You can terminate a process using `kill [PID]`. Below are some of the signals that you can send to a process when it's killed :
- `SIGTERM` - Kill the process, but allow it to do some cleanup tasks beforehand
- `SIGKILL` - Kill the process - doesn't do any cleanup after the fact
- `SIGSTOP` - Stop/suspend a process

To interact with the **systemd** process/daemon, use the command `systemctl [option] [service]`

We can do 5 options with :
- `start
- `stop
- `enable
- `disable
- `status


### Backgrounding & Foregrounding

Processes can be run in the background or in the foreground. For example, `echo` is run in the foreground but you can run it in the background as you can see in the example below :

```
root@linux:~# echo "Hello!"
Hello!
root@linux:~# echo "Hello!" &
[1] 16889
```

- The `&` operator will run the command in the background so you can continue to execute your commands without having to wait for this command to finish first. 
	For example, you can run `sudo apt install [package] & echo "Downloading"` to execute the downloading in the background and run the echo.
- You can also use `Ctrl + Z` to run a command in the background.

To foreground a process backgrounded using either `Ctrl + Z` or the `&` operator, you can use `fg` to bring this back to focus. 


### Automations

You can automate the execution of commands using `cron`. To edit the crontab where you set the cron automations, you can use the command `crontab -e`.


