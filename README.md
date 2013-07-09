scriptreplay_ng
===============

Installation
-------------

 * Install "scriptreplay" and "recordsession" to /usr/local/sbin
 * Add /usr/local/sbin to $PATH of the user
 * Add the follwing lines via "visudo"
```
<user> ALL=(ALL) NOPASSWD: /usr/local/sbin/scriptreplay
<user> ALL=(ALL) NOPASSWD: /usr/local/sbin/recordsession
```


Usage
-----

 * Start session
   ```
sudo recordsession
   ```
 * Replay session
   ```
sudo scriptreplay -t /var/log/recordshell//2013-07-08/2013-07-08_17-39-41-27336/timing.gz /var/log/recordshell//2013-07-08/2013-07-08_17-39-41-27336/typescript.gz
   ```

Documentation 
-------------

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
