# Learning Linux — The Basics (My Notes)

Hi! I'm currently learning Linux, and instead of just collecting commands I forget by Friday, I'm writing down what I actually understand. This notes comes 1-2 weeks after wen i officially learned a concept and played around with it. 
This is the first level: moving around, understanding who you are on the system, and why Linux cares so much about *who is allowed to do what*.

If you're a recruiter skimming this: this is me showing my work, not pretending to be a sysadmin. Yet. 🙂
And if you are also trying to learn LINUX see wat new command i may have here✌

---

## 1. Moving around: `cd` (change directory)

`cd` is the command I use the most. It's literally "change directory" — where am I going next?

```bash
cd /var/log      # go to an absolute path
cd projects      # go into a folder called projects (relative)
cd ..            # go UP one level (the two dots mean "parent folder")
cd ../..         # up two levels, because why stop at one
```

### And then there's cd ~ — the reset button
```bash
cd ~
```
The tilde ~ means my home directory. So cd ~ takes me straight back to my own little corner of the system — /home/yourname on most setups.

I think of cd ~ as the reset button. Lost in a maze of folders? Not sure where you are? cd ~ and you're home. Safe. Back to your own user. It doesn't undo anything, but it resets your location bringin you back to a place you recognize.

```bash
cd ~        # take me home
cd          # same thing, just lazier (also goes home)
cd -        # bonus: jump back to the PREVIOUS folder. Very handy.
```

## 2. But wait... whose home am I going to?
Here's the thing that clicked for me: cd ~ takes me home. But that little word "me" is doing a lot of work — because on a Linux system, everything is tied to a user.

Which brings me to the big one.

### The `root` user
Linux has one super-user called root. Root can do anything — read any file, change any setting, delete anything. Root is not a person; root is a power level.

Every normal user (like me) has limits.

Root has none.

So you don't log in as root for day-to-day stuff. You use a normal account and only escalate when you really need to (with sudo, which I'll get to in a later note).

`whoami` tells me which user I currently am

```bash
whoami
# → yourname        (a normal human)

sudo whoami
# → root            (the boss)

```
## 3. Creating users
Since Linux is built around users, I need to know how to make them. There are two commands, and after playin with them, they're not the same:

```bash
sudo adduser alice     # friendly, interactive: asks questions, sets up home dir
sudo useradd alice     # low-level, silent: creates the user and nothing else

### Checking users exist:
cat /etc/passwd        # the classic "list of users" file (readable by everyone)
id alice               # shows alice's user ID, group ID, and groups

### Managing passwords
sudo passwd alice      # set or change alice's password (as root/admin)
passwd                 # change MY OWN password

```
A few things I learned the hard way:

`passwd` on its own changes your password. `sudo passwd someone` changes theirs.

Passwords aren't stored in plain text — they live hashed in `/etc/shadow`, which normal users can't even read. That's why you need `sudo`.

`sudo passwd -l alice` locks an account (puts a ! in front of the hash) without deleting it. -u unlocks it. I think in real world its the polite way to disable an account.

```bash
sudo passwd -l alice   # lock
sudo passwd -u alice   # unlock
```
### 4. Groups
Groups are how Linux lets you give permissions to several people at once instead of one by one. Every user has a primary group and can belong to many supplementary groups.

```bash
sudo groupadd developers          # create a group
sudo usermod -aG developers alice # add alice to it

```
The -aG flags matter: -a means append (don't remove her from other groups) and -G means supplementary groups. If I forget the -a, I overwrite her group list.  Dont ask me how i had to master -a flag, in summary if ALICE was a real user we probably would ave visted R several times.

```bash
## Alwas check
groups alice       # which groups is alice in?
id alice           # the fuller picture
```

### 5 ...and this is where permissions come in

So now I have users, passwords, and groups. But what are they all for?


Everything in Linux comes down to permissions. Users and groups aren't there for decoration: they exist so the system can answer one question about every single file:

`Who is allowed to read this, write to this, or execute this?`

Every file has three permission slots — owner, group, and others — and each slot can have read (r), write (w), and execute (x). That's why you'll see things like `rwxr-xr--` next to files. 

### 6 Seeing it all: ls and ls -la
ls lists what's in a directory:

```bash
ls
```
But plain ls hides the interesting stuff — especially hidden files (anything starting with a dot) and all the permission info. So:

```bash
ls -la

# -l = long format. Shows permissions, owner, group, size, and date for each entry.

# -a = all. Includes hidden files, like .bashrc and the special . (this folder) and .. (parent folder — the same .. from cd ..!).

$ ls -la
drwxr-xr-x  5 alice developers 4096 Nov 22 10:14 .
drwxr-xr-x 20 root  root       4096 Nov 22 09:58 ..
-rw-r--r--  1 alice developers  220 Nov 20 18:02 .bash_logout
-rw-r--r--  1 alice developers 3771 Nov 20 18:02 .bashrc
drwxr-xr-x  2 alice developers 4096 Nov 22 10:14 notes

##########--------------------
```
| Piece | Means |
|---|---|
| `d` or `-` | directory or regular file |
| `rwxr-xr-x` | owner can read/write/execute; group can read/execute; others can read/execute |
| `alice` | the owner |
| `developers` | the group |
| `4096` | size in bytes |
| `Nov 21 10:14` | last modified |


## 7 Todays Cheat Sheet

| Command | What it does |
|---|---|
| `cd ..` | up one folder |
| `cd ~` | home — my reset button |
| `cd -` | back to the previous folder |
| `whoami` | who am I? |
| `sudo whoami` | ...and who am I with power? |
| `adduser alice` | create a user, the friendly way |
| `passwd alice` | set a user's password |
| `passwd -l alice` | lock an account |
| `groupadd devs` | make a group |
| `usermod -aG devs alice` | add a user to a group |
| `groups alice` | which groups is a user in? |
| `ls -la` | list everything, with permissions |
| `ls -R` | BONUS: i love to see everything using this  -R Flag |