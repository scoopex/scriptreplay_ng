
scriptreplay_ng
===============

Scriptreplay can be used to replay recorded session recorded by the linux/unix "script" tool.
This project also provides tools to setup auditable shell sessions.

# Usage

  * Installation: Add the tools "scriptreplay" and "record-script-session" to your PATH environment
    ```bash
    cd ~ 
    git clone git@github.com:scoopex/scriptreplay_ng.git
    echo '$HOME/scriptreplay_ng:$PATH' >> ~/.bashrc
    exec bash
    ```    

  * Record session
    ```bash
    record-script-session
    record-script-session "<sessioname>"
    ```

  * Replay session
     ```bash
    scriptreplay -t timing typescript
    scriptreplay -t ~/.script/2014-07-21/2014-07-21_09-42-46-crontab/timing.gz ~/.script/2014-07-21/2014-07-21_09-42-46-crontab/typescript.gz 
    ```
# Manpage

```
NAME
    scriptreplay - play back typescript of terminal session

SYNOPSIS
    scriptreplay -h|--help

    scriptreplay [-a|--accelerate <num>] [-t|--timing <timingfile>]
    <typescript>

DESCRIPTION
    scriptreplay replays a typescript of a terminal session; optionally,
    using timing data to ensure realistic typing and output delays.

    The timing data consists of two fields, separated by a space. The first
    field indicates how much time elapsed since the previous output. The
    second field indicates how many characters were output this time.

    *typescript* is the path to the typescript file. If the file
    *typescript*.timing exists then it is automatically used as timing data
    file. Use parameter -t or --timing to specify an alternative timing data
    file.

    This version of scriptreplay supports reading of compressed *typescript*
    files. If *timingfile* is not specified, scriptreplay tries to open a
    timing data file that uses the same compression algorithm as
    *typescript*. The decompression method is determined by examining the
    file extension of the *typescript* file. Recognized file extensions of
    compressed *typescript* files are: "bz2", "gz", "lz" or "lzma".

  Controlling the playback
    *   "-" or "d" decreases display speed.

    *   "+" or "i" increases display speed.

    *   "s" or "p" pauses the playback; and "c" continues again.

    *   "f" or "q" stops the playback and exits scriptreplay.

    Pressing any other key jumps to the next output (useful if there is no
    output activity due to a long delay).

OPTIONS
    -a, --accelerate *num*
            Accelerates timing by factor *num*. *num* must be greater than
            0. A *num* value less than 1 slows down the playback speed; and
            a value greater than 1 increases the playback speed.

    -t, --timing *timingfile*
            Specify the file path to the timing data file.

EXAMPLES
  Create a new typescript with timing data
     user@caladan:~$ script -t typescript 2>typescript.timing
     Script started, file is typescript
     user@caladan:~$ ls
       ...
     user@caladan:~$ exit
     Script done, file is typescript

  Replay a typescript
     user@arrakis:~$ scriptreplay typescript
     user@caladan:~$ ls
       ...
     user@caladan:~$ exit

     scriptreplay: typescript time (normal):    14 seconds ( 0 minutes)
     scriptreplay: typescript time (accel) :     1 seconds ( 0 minutes)

NOTES
    The playback might not work properly if the typescript contains output
    from applications that have been recorded with different termio settings
    and/or terminal window sizes.

COPYRIGHT
    This program is in the public domain.

AUTHORS
    Joey Hess <joey@kitenet.net>

    Marc Schoechlin <ms@256bit.org>

    Hendrik Brueckner <hb-perl@256bit.org>

SEE ALSO
    script(1), bzcat(1), zcat(1), lzcat(1)
```

# Auditshell

Auditshell replaces bash by audited replacement which writes scriptreplay data to /var/log/auditshell. 
The apparmor profile prevents audited users to read, change or delete the recorded sessions and to change thei shell.

## Installation of "auditshell"

The following instructions describe the procedure how to install a audit shell in combination with
the scriptreplay utility.

 * Install the tools and the apparmor configuration
   ```
   cp scriptreplay helpers/auditshell /usr/local/bin/
   cp helpers/usr.local.bin.auditshell /etc/apparmor.d/usr.local.bin.auditshell
   chmod 755 /usr/local/bin/auditshell /usr/local/bin/scriptreplay
   chown root:root /usr/local/bin/auditshell /usr/local/bin/scriptreplay
   ```
 * Create a directory for auditfiles
   ```
   mkdir /var/log/auditshell/
   chmod 1777 /var/log/auditshell/
   ```
 * Enable apparmor profle
   ```
   aa-enforce /usr/local/bin/auditshell
   ```
 * Change shell of user which should be audited
   ```
   chsh -s /usr/local/bin/auditshell <user>
   ```

## Watch auditshell sessions

 * Identify a session 
   ```
   cd /var/log/auditshell
   ls -1d *
   ```
 * View a session
   ```
   scriptreplay -t 2017-10-21_10-19-17.marc.2159/timing* -s 2017-10-21_10-19-17.marc.2159/typescript*
   ```
