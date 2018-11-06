# check-process-memory-usage
A plugin for Nagios (and other compatible monitoring services) to check and monitor the main memory usage of one or more unix process. A single process can be located by pid or a pidfile or a single or multiple processes by filtering the process list using pid, gid, file name, and command line.

## About
I had an actual use case where I needed to monitor the main memory usage [(resident set size)](https://en.wikipedia.org/wiki/Resident_set_size) of a group of httpd-processes running under different users and locations.

Looking around for a solution it seemed that this requirement was more or less abundant. So, I have written this plugin to solve my needs and maybe somebody else's needs.

## Installation
Copy the check_process_memory_usage file to your Nagios plugins directory, e.g.:

    git clone https://github.com/aalborgunilib/check-process-memory-usage
    cd check-process-memory-usage
    sudo cp plugins/check_process_memory_usage /usr/lib64/nagios/plugins/
    sudo chmod 755 /usr/lib64/nagios/plugins/check_process_memory_usage

Install Perl dependencies for the plugin via e.g. [cpanm](https://metacpan.org/pod/App::cpanminus). (Please stay within the check-process-memory-usage directory):

    curl -L https://cpanmin.us | perl - --sudo App::cpanminus
    cpanm --sudo --installdeps .

*Compilation of the Perl dependencies may require the installation of software development tools on the server (gcc).*

Now, check to see if the plugin is working:

    plugins/check-process-memory-usage 
    
You should get something in the line of:

    PROCESS_MEMORY_USAGE OK - RSS 192M - SIZE: 8.8G | 'resident set size'=196348KB;; 'virtual memory size'=9201472KB;;

The reported metrics are resident set size and virtual memory size. The first being representative of the non-swapped physical memory that a process is using.

To check for a specific process you can try:

    plugins/check_process_memory_usage --verbose --uid root --cmndline /usr/sbin/sshd

You should get something like this:

    UID        GID        PID    FNAME                    CMNDLINE
    root       root       1085   sshd                     /usr/sbin/sshd -D
    PROCESS_MEMORY_USAGE OK - RSS 4.3M - SIZE: 111M | 'resident set size'=4308KB;; 'virtual memory size'=112812KB;;

The UID root and CONDLINE part /usr/sbin/sshd should be coloured red on an ANSI terminal to show how the filters matched the process list (when using --verbose).

## Usage

    Usage: check_process_memory_usage
        [ -c|--critical=<critical threshold in kB> ]
        [ -w|--warning=<warning threshold in kB> ]
        [ -n|--no_filter_match ]
        [ -f|--fname=<file name filter> ]
        [ -C|--cmndline=<command line filter> ]
        [ -u|--uid=<uid or user name filter> ]
        [ -g|--gid=<gid or group name filter> ]
        [ -p|--pid=<pid> ]
        [ -P|--pidfile=<pidfile to get pid> ]
        [ -t|--timeout=<timeout> ]
        [ -v|--verbose ]

The `-c|--critical` and `-w|--warning` defines the standard Nagios service check thresholds (in kB). The critical and warning thresholds can be omitted e.g. if you would like to just graph the memory usage. The thresholds are matched against RSS (resident set size) memory.

`-n|--no_filter_match` will let the plugin exit with an OK even if none of the filters matched up against a process. This will avoid the plugin of sending an UNKNOWN status in case the process has died or has not yet been started (many will probably monitor this by using another plugin). Metrics are still reported in either case as being 0KB.

`-f|--fname` will filter the process list by the exact base name of the process's executable file. E.g. `--fname=sshd` will match sshd processes but `--fname=ssh` will not.

`-C|--cmndline` will filter the process list by matching against the command with all its arguments as a string. The match is done as a single index match. E.g. `--cmndline=ssh` will match the commandline `/usr/sbin/sshd -D`. Also, *the plugin will automatically filter out itself from the process list* as the `cmndline`-option will always return a selfmatch.

`-u|--uid` will filter the process list by matching against the uid (or user name). Mapping from uid to user name is done automatically if the uid is not all numeric.

`-g|--gid` as uid but with group id (or group name).

Filters can be combined to narrow down a process or group of processes. E.g. `--fname=httpd --cmndline=config/proxy.conf` to filter httpd processes running with a certain configuration file.

`-p|--pid` will select the process with the given pid.

`-P|--pidfile` as pid but will read the pid from the pidfile.

`-t|--timeout` is the plugin timeout. If timeout is reached, the check will bail out and issue an UNKNOWN state.

`-v|--verbose` outputs a process list (much like `ps -eo user,group,pid,fname,command`) matching the filters and color the matches for you to easilly check that the filters you specified are working to your preferences.

## Copyright and license

Copyright (c) 2018 Kasper Løvschall and Aalborg University Library

This software is free software; you can redistribute it and/or modify it under the same terms as Perl itself. See [Perl Licensing](http://dev.perl.org/licenses/).
