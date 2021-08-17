<h2>Processes</h2>
We create "processes" when we interact with Linux. A process is just a numbered instance of a running program. The <i>ps</i> command displays a list of all processes.

Many processes can run concurrently on modern systems. The OS quickly swithces between various processes running on the CPU. Multicore CPU's can acutually execute many processes at the same time. Each core quickly switches between various processes. To long listing of all processes on the system, use:

```bash
ps -ef 
```

To see the user ID, process ID, parent process ID, CPU usage, and command name of a process, use:

```bash
ps -e --format uid,pid,ppid,%cpu,cmd 
```

<i>top</i> shows the sorted list of active processes:

```bash
top
```

<h2>Foreground and background</h2>

Jobs can either be in the foreground or the background. Thus far, we have run commands at the prompt and waited for them to complete. We call this running in the “foreground.”

Use the “&” operator, to run programs in the “background”:

```bash
sleep 1000 &
```

Now you can see which processes are running in the background:

```bash
jobs
```

To bring a background program to foreground, use:

```bash
fg 1
```

<h2>Terminate processes</h2>

To kill a process, use the ‘kill’ command with the five-digit process id:

```bash
kill 54356
```

Use a -9 option to cause a process to end suddenly (and with a greater likelihood of success):

```bash
kill -9 54356
```

Available signals:

| Signal | Value |  Description |
| --- | --- | --- |
| SIGHUP | (1) | Hangup |
| SIGINT | (2) | Interrupt from keyboard |
| SIGKILL | (9) | Kill signal |
| SIGTERM |  (15) | Termination signal |

Properly killing processes:
1. Send a SIGINT.
2. Send a SIGTERM.
3. Send a SIGKILL.

<h2>pkill vs killall</h2>

Both pkill and killall offer distinct choices. Killall provides an option to match processes based on their age, whereas pkill contains a flag to exclusively kill processes on a certain tty. Neither is superior; they simply specialize in different areas.

```bash
pkill -SIGTERM -f chromium
killall -15 chromium
```