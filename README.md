
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

Auditshell submits the typescript and the timings of a patched util-linux/script binary to syslog which prevents modification by regular terminal users.
The logged information can also be forwarded to secured logging servers using standard syslog logfile distribution.


## Installation of "auditshell"

The following instructions describe the procedure how to install a audit shell in combination with
the scriptreplay utility.

 * Install tools
  
    ```bash
    cp scriptreplay helpers/auditshell helpers/auditshell_create_sessionfiles /usr/local/bin/
    chown root:root /usr/local/bin/{scriptreplay,auditshell,auditshell_create_sessionfiles}
    chmod 755 /usr/local/bin/{scriptreplay,auditshell,auditshell_create_sessionfiles}
    ```
 * Install Build dependencies
 
   ```bash
   apt-get install libtoolize libtool autopoint pkg-config make gcc
   zypper install libtool gettext-tools pkg-config make gcc autoconf automake
   ```
 * Patch and install custom "script" implementation
 
   ```bash
   cd helpers/
   wget https://www.kernel.org/pub/linux/utils/util-linux/v2.23/util-linux-2.23.tar.gz
   tar zxvf util-linux-2.23.tar.gz
   cd util-linux-2.23/
   patch -p1 < ../auditshell_script.patch
   ./configure --without-ncurses --disable-nls
   make
   cp script /usr/local/bin/
   chown root:root /usr/local/bin/script
   chmod 755 /usr/local/bin/script
   ```
 * Syslog configuration:
    * Disable string escaping on system which are using rsyslogd (i.e. Ubuntu systems with rsyslogd)
    * Redirect the auditshell logs to another logfile using syslog configuration 
 * Change shell of user
 
   ```bash
   chsh -s /usr/local/bin/auditshell <user>
   ```
 * Prevent regular users to change their shell
   * Remove all shells from /etc/shells
   * Remove all perrmissions from the the /usr/bin/chsh binary
   * Use the FAKE_SHELL variable in /etc/login.defs

## Watch auditshell sessions

 * Start session, and execute commands
 * Extract session files
 
   ```bash
   /usr/local/bin/auditshell_create_sessionfiles /var/log/messages /tmp/foo
   ```
 * Replay session

   ```bash
   scriptreplay -t /tmp/foo/2013-09-11_18-47-45.user1.11931.timing \
       /tmp/foo/2013-09-11_18-47-45.user1.11931.typescript
   ```

## Logging configuration

### Syslog-NG Configuration


 * Edit /etc/syslog-ng/syslog-ng.conf
      ```
      # define audit shell filter
      filter f_auditshell { match('^auditshell'); };
      # enhance existing messages filter by f_auditshell to ignore messages matched by f_auditshell
      filter f_messages   { not facility(news, mail) and not filter(f_iptables) and not filter(f_auditshell); };

      # define a log-sink for auditshell
      destination auditshell {
        file ("/var/log/auditshell/$YEAR-$MONTH/$FACILITY-$YEAR-$MONTH-$DAY"
         owner(root) group(root) perm(0600) dir_perm(0700) create_dirs(yes)
        );
      };
      log { source(src); filter(f_auditshell); destination(auditshell); };
      ```
 * Restart Syslogd
      ```
      /etc/init.d/syslog restart
      ```
