So I had a problem the other day where I wanted to count the number of running processes associated with the running instance of a script - until now I had only been using /sbin/pidof to find the pids running a script by name, but if there are other instances of the same script running, this would give an incorrect result.

If instead of using `pidof`, I use `ps` with the `--ppid` filter, which shows only pids that have the given pid (or pids, if you specify more than one) as its parent.
As it turns out, when you run subroutines from within a Bash script, the parent pid (ppid) of each sub-process is always that of the main process executed, even if your subprocesses spawn subprocesses of their own, the ppid will still be that of the first process.
<pre>#!/bin/bash
function ThreadCount {
    `ps --no-headers -o pid --ppid $$ | wc -l`
}
ct=ThreadCount</pre>
The above code lists the number of processes which have a parent pid that matches the currently executing process pid, and counts them. We must remember to omit the headers from the `ps`output, otherwise our final count will include the header line.

`$$` is the variable in which the parent PID for the currently running process is stored.

`wc -l` counts the number of lines in STDOUT, piped from the `ps` command.
