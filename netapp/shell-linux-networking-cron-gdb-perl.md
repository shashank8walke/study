# Shell, Linux, Networking, Cron, GDB, Perl

---

## Shell Scripting (Bash)

### Basics

```bash
#!/bin/bash

var="Shashank"                 # no spaces around =
echo "Name is ${var}"
curr_date=$(date)              # $() runs a command and captures output
```

### Arguments

```bash
./abc.sh net-123 500
script_name=$0    # script path
volume_id=$1      # first arg
size=$2           # second arg
num_args=$#       # count of args
all_args=$@       # all args as separate words
```

### Named flags with `getopts`

```bash
while getopts "n:s:h" flag; do
    case $flag in
        n) name=$OPTARG ;;
        s) size=$OPTARG ;;
        h) echo "Usage: $0 -n <name> -s <size>"; exit 0 ;;
        *) echo "Unknown flag"; exit 1 ;;
    esac
done
echo "$name ${size}GB"
```

`n:` means `-n` takes an argument; `h` (no colon) is a flag only.

### Arithmetic

```bash
sum=$((2 + 3))    # integer only; for float use bc
result=$(echo "3.14 * 2" | bc)
```

### Conditionals

```bash
if [ -f "/etc/config.yaml" ]; then   # -f = file exists
    echo "found"
elif [ -d "/etc/config" ]; then      # -d = directory exists
    echo "dir"
else
    echo "missing"
fi
```

| Test | Meaning |
|---|---|
| `-f file` | File exists and is regular |
| `-d dir` | Directory exists |
| `-z "$var"` | String is empty |
| `-n "$var"` | String is non-empty |
| `-r file` | File is readable |

Comparison operators: `-gt -ge -lt -le -eq -ne` (for integers)

> Always quote variables inside `[ ]`: `[ -z "$var" ]` not `[ -z $var ]`

### Loops

```bash
for file in $(ls /mnt); do
    echo "$file"
done

cnt=0
while [ $cnt -lt 5 ]; do
    echo $cnt
    ((cnt++))
done
```

### Functions

```bash
try_function() {
    local name=$1       # local scopes variable to function
    echo "Hello $name"
}

try_function "Shashank"
```

---

## Text Processing Tools

### grep — find matching lines

```bash
grep "error" file.txt
grep -i "error" file.txt       # case-insensitive
grep -r "error" ./logs/        # recursive
grep -v "debug" file.txt       # invert (lines NOT matching)
grep -n "error" file.txt       # show line numbers
grep "^START" file.txt         # lines starting with START
grep "END$" file.txt           # lines ending with END
grep "error|warning|info" file.txt   # multiple patterns (use -E or egrep)
```

### awk — column/field processing

```bash
awk '{print $1}' file          # print first column (whitespace delimited)
awk '{print $NF}' file         # last column (NF = number of fields)
awk -F: '{print $1}' /etc/passwd   # split by : delimiter
awk '/error/ {print $0}' log.txt   # print lines containing "error"
awk '$3 == "offline" {print $1, $2}'   # conditional print
```

### sed — stream editor (find/replace/delete)

```bash
sed 's/foo/bar/' file.txt          # replace first occurrence per line
sed 's/foo/bar/g' file.txt         # replace all (global)
sed -i 's/foo/bar/g' file.txt      # edit file in-place
sed '/error/d' file.txt            # delete lines matching "error"
sed '1,5d' log.txt                 # delete lines 1 through 5
```

### cut — extract columns by position

```bash
cut -d: -f1 /etc/passwd            # field 1, delimiter :
cut -d: -f1,6 /etc/passwd          # fields 1 and 6
cut -d: -f2-4 file.txt             # fields 2, 3, 4
cut -c1-10 file.txt                # characters 1 to 10
```

### sort / uniq

```bash
sort data.txt | uniq -u            # lines that appear exactly once
sort -n numbers.txt                # numeric sort
sort -r file.txt                   # reverse
```

### strings — extract printable text from binary files

```bash
strings data.bin | grep "=="       # find readable content in binary/compressed files
```

### xargs — pipe output as arguments to another command

```bash
ls /home/shash | xargs rm                         # delete all listed files
find . -name "*.yaml" | xargs -I{} cp {} /backup/ # copy each found file to /backup
```

---

## Linux

### File permissions

```
rwx rwx rwx
421 421 421
owner group other
```

```bash
chmod 600 key.pem          # owner read+write only (required for SSH keys)
chmod 755 script.sh        # owner rwx, group/other rx
chown user:group file      # change ownership
```

### Process management

```bash
ps aux                     # all processes (a=all users, u=user format, x=no tty)
ps -ef | grep java         # find process by name
top / htop                 # live process viewer
kill -15 PID               # SIGTERM: graceful shutdown (process can catch/cleanup)
kill -9 PID                # SIGKILL: force kill (cannot be caught by process)
jobs / fg / bg             # job control (background/foreground)
nohup cmd &                # run detached from terminal (survives logout)
```

> **Interview:** `SIGTERM` can be caught by the process for cleanup. `SIGKILL` cannot — the kernel forces termination.  
> **Zombie process:** A process that finished but its parent hasn't called `wait()` to collect the exit status. Shows as `Z` in `ps`. Wastes a PID but no CPU/memory.

### Disk

```bash
df -h                      # disk usage per filesystem (-h = human-readable)
du -sh dir/                # total size of a directory
lsblk                      # list block devices
mount / umount             # mount/unmount filesystems
```

`df` = filesystem-level view; `du` = directory-level view.

### File operations

```bash
find . -name "*.log" -type f -size +5M -user root   # complex search
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null  # suppress permission errors
ln -s target link          # create symlink
```

`2>/dev/null` redirects stderr (e.g. permission denied noise) to null — critical pattern for automation scripts.

### Inode concept

Every file has an inode containing metadata (permissions, owner, timestamps, block pointers). Path → inode → data blocks.  
- **Hard link:** shares the same inode (same file, two names)  
- **Symlink:** separate inode pointing to a path (breaks if target is deleted)

### Compression / archiving

```bash
tar -czf archive.tar.gz dir/    # create: c=create, z=gzip, f=filename
tar -xzf archive.tar.gz         # extract
gzip -d file.gz                 # decompress gzip
gunzip file.gz                  # same as above
bunzip2 file.bz2
```

### SSH

```bash
ssh username@host -p 2220
ssh -i sshkey.private user@host -p 2220   # key-based auth
```

Key must be `chmod 600` — SSH refuses keys readable by others. Used everywhere in cloud: EC2 key pairs, CI/CD pipeline access, Azure SSH keys.

### Other useful

```bash
base64 -d data.txt             # decode base64
cat file | tr 'A-Za-z' 'N-ZA-Mn-za-m'   # ROT13 decode
xxd file | head                # hex dump (inspect binary)
file data                      # detect file type
strings file | grep pattern    # extract printable text from binary
```

---

## Networking

### curl — HTTP transfers

```bash
curl https://api.example.com          # basic GET
curl -i url                           # response body + headers
curl -I url                           # headers only
curl -o output.txt url                # save to file
curl -u user:pwd url                  # basic auth
curl -H "Authorization: Bearer TOKEN" url
curl -X POST url -d '{"key":"val"}' -H "Content-Type: application/json"
```

### Network inspection

```bash
ip addr show                   # all interfaces and IPs (modern ifconfig)
ip route show                  # routing table
netstat -tulnp                 # listening ports: TCP+UDP, numeric, show PID
ss -tulnp                      # same but faster/modern (preferred over netstat)
```

Flags: `-t` TCP, `-u` UDP, `-l` listening only, `-n` numeric (skip DNS), `-p` show PID.

### Connectivity testing

```bash
ping host                      # ICMP reachability
traceroute host                # path + latency per hop (probe = test packet)
nc -zv hostname 443            # check if remote port is open (netcat)
nmap host                      # port scan
dig example.com                # DNS lookup
```

### How DNS resolution works

1. Check local cache → `/etc/hosts` → local resolver
2. Recursive resolver queries: root nameserver → TLD nameserver (`.com`) → authoritative nameserver
3. Returns IP, cached with TTL

### Common ports

| Port | Service |
|---|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 25 | SMTP |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 2181 | Zookeeper |
| 9092 | Kafka |

### Debugging: microservice can't reach database

```bash
ping db-host                   # 1. reachable at all?
nc -zv db-host 5432            # 2. port open?
dig db-host                    # 3. DNS resolving?
curl -v http://db-host:5432    # 4. any response?
# 5. check firewall rules / security groups
ss -tulnp                      # 6. on db server: is it actually listening?
```

### wget vs curl

- **wget:** best for downloading files, supports recursive download (`-r`)
- **curl:** better for API testing, scripting, supports more protocols

### traceroute notes

`* * *` means the router dropped/blocked ICMP probes — not necessarily a problem, just a silent router. Multiple IPs per hop = different paths taken by each probe packet (load balancing).

---

## cron

**Field order:** `Minute Hour Day-of-month Month Weekday`  
Memory trick: **M H D M W** — "My Hamster Does Math Well"

```
* * * * * command
│ │ │ │ └── Day of week (0=Sun, 1=Mon ... 6=Sat)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

```bash
0 9 * * 1 /path/to/script.sh     # every Monday at 9 AM
0 2 * * * /home/shashank/backup.sh  # daily at 2 AM
*/5 * * * * cmd                  # every 5 minutes
```

Shortcuts:

```
@daily    = 0 0 * * *
@weekly   = 0 0 * * 0
@reboot   = run at startup
```

```bash
crontab -e    # edit your crontab
crontab -l    # list
crontab -r    # remove all
```

`/etc/crontab` — system-wide crontab, has an extra `USER` field.

> Always use full paths in cron jobs (e.g. `/usr/bin/python3`) — cron runs with a minimal `PATH`.

---

## GDB (GNU Debugger)

```bash
gcc -g file.c -o prog    # compile with debug symbols
gdb prog                 # start debugger
gdb prog core            # debug from core dump (post-crash analysis)
```

### Core commands

| Category | Commands |
|---|---|
| Run | `run [args]`, `kill`, `quit` |
| Flow | `next` (step over), `step` (step into), `continue`, `finish` |
| Breakpoints | `break <line\|func>`, `tbreak` (one-time), `disable N`, `ignore N count` |
| Watchpoints | `watch var` (write), `rwatch var` (read), `awatch var` (both) |
| Inspect | `print var`, `set var=val`, `list`, `x/FMT addr` |
| Stack | `backtrace`, `frame N`, `info locals`, `info args` |
| Advanced | `info registers`, `disassemble func` |

- **`step` vs `next`:** `step` goes into function calls; `next` skips over them
- **Segfault:** compile with `-g`, run in gdb, it crashes → `backtrace` → walk frames → `info locals` to find the bad pointer
- **Infinite loop:** run → Ctrl-C to interrupt → `backtrace` to see where it's stuck
- **Unexpected variable modification:** use `watch` command
- **Already-crashed process:** use core dump → `gdb prog core`

---

## Perl

Perl is used at NetApp for config parsing, log analysis, and storage workflow scripting. Goal: read and understand existing scripts.

```perl
#!/usr/bin/perl
use strict;     # forces variable declaration with my
use warnings;   # warns about uninitialized values, typos
```

### Variables

```perl
my $name  = "Shashank";       # scalar: single value
my @arr   = (4, 5, 6);        # array: ordered list
my %map   = ("key" => 4);     # hash: key-value pairs (=> is just a comma)
```

```perl
print "Value is $name\n";     # double quotes: interpolates variables
print 'Value is $name\n';     # single quotes: literal (no interpolation)
```

### Arrays

```perl
push @arr, 6;       # add to end
pop @arr;           # remove from end
unshift @arr, 0;    # add to front
shift @arr;         # remove from front
my $len = scalar @arr;  # length
```

### Hashes

```perl
my %h = ("name" => "Shashank", "age" => 24);
print $h{name};                # access value
foreach my $key (keys %h) {
    print "$key => $h{$key}\n";
}
my @keys = keys %h;
my @vals = values %h;
```

### Control flow

```perl
# foreach with explicit variable
foreach my $item (@fruits) { print "$item\n"; }

# foreach with default $_ (implicit variable)
foreach (@fruits) { print "$_\n"; }

# C-style for
for (my $i = 0; $i < 5; $i++) { ... }

# while
while ($cnt < 10) { $cnt++; }

last;   # break
next;   # continue (skip to next iteration)
redo;   # restart current iteration without re-checking condition
```

### Subroutines

```perl
sub add {
    my ($a, $b) = @_;     # @_ is the magic argument array
    return $a + $b;
}
my $res = add(3, 4);
```

### File I/O

```perl
# Read
open(my $fh, '<', 'file.txt') or die "Cannot open: $!";  # $! = OS error message
while (my $line = <$fh>) {
    chomp $line;              # removes trailing newline
    print "$line\n";
}
close($fh);

my @all_lines = <$fh>;        # slurp entire file into array
chomp @all_lines;

# Write (overwrite)
open(my $out, '>', 'out.txt') or die "Cannot open: $!";
print $out "line 1\n";
close($out);

# Append
open(my $log, '>>', 'log.txt') or die $!;
print $log "New entry\n";
close($log);
```

File modes: `<` read, `>` write/overwrite, `>>` append, `+<` read+write.

`die` exits the script. `warn` prints warning but continues. Always include `$!` for the OS-level reason.

### Regex

```perl
if ($str =~ /pattern/) { }      # match
if ($str !~ /pattern/) { }      # no match
$str =~ s/old/new/g;            # global substitution
$str =~ s/^\s+|\s+$//g;         # trim leading/trailing whitespace
```

| Syntax | Meaning |
|---|---|
| `.` | any character except newline |
| `\d` | digit `[0-9]` |
| `\w` | word char `[a-zA-Z0-9_]` |
| `\s` | whitespace |
| `\D \W \S` | complement (non-digit, non-word, non-space) |
| `?` | 0 or 1 |
| `+` | 1 or more (greedy) |
| `*` | 0 or more (greedy) |
| `+?` | 1 or more (non-greedy) |
| `{n,m}` | between n and m |
| `^` | start of string/line |
| `$` | end of string/line |
| `\b` | word boundary |
| `[abc]` | one of a, b, c |
| `[^abc]` | not a, b, c |
| `(...)` | capture group → `$1`, `$2` |
| `(?:...)` | non-capturing group |
| `(?<name>...)` | named capture → `$+{name}` |

Modifiers: `/i` case-insensitive, `/g` global, `/m` multiline, `/s` dot matches newline.

### References (for complex data structures)

```perl
my $aref = \@arr;         # reference to array
my $href = \%h;           # reference to hash

# Array of hashes (common in scripts)
my @people = (
    { name => "Alice", age => 30 },
    { name => "Bob",   age => 25 },
);
foreach my $p (@people) {
    print "$p->{name} is $p->{age}\n";
}
```

### Backticks — run shell commands

```perl
my $output = `ls -la`;    # captures stdout of shell command
```

### Common interview questions

- **`chomp`** — removes trailing `\n` from string or array elements
- **`or die $!`** — `$!` gives OS error reason ("No such file or directory")
- **`die` vs `warn`** — `die` exits, `warn` continues
- **`while (<$fh>)`** — reads file line by line into `$_`
- **Skip blank lines** — `next if /^\s*$/`
- **`s/^\s+|\s+$//g`** — trims leading and trailing whitespace (`|` is alternation)
- **`+` vs `+?`** — greedy vs non-greedy (non-greedy matches as little as possible)
- **Capture groups** — `(...)` captures into `$1`, `$2`, etc.
