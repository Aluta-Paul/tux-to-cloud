# Learning Linux — Level 1: Moving Around, Users, Permissions, and the Magic of `x`

I'm learning Linux, and this is my running log — one file, updated as I go, so I actually remember what I learn instead of collecting commands I forget by Friday. What started as "how do I get around the filesystem" turned out to be the same question as "who am I, and what am I allowed to touch?" — asked in about six different ways.

If you're a recruiter skimming this: this is me showing my work, not pretending to be a sysadmin. Yet. 🙂

---

## Part 1 — `cd`, and the reset button

`cd` is "change directory," and it's the command I use the most, because I'm always somewhere I didn't mean to be.

```bash
cd /var/log      # go to an absolute path
cd projects      # go into a folder called projects
cd ..            # UP one level — the two dots mean "parent folder"
cd ../..         # up two levels, because why stop at one
```

`cd ..` is my "oops, wrong folder" button. It's the Linux version of the back arrow in a browser.

And then there's the one I actually love:

```bash
cd ~
```

The tilde `~` means **my home directory** — `/home/yourname` on most setups. So `cd ~` takes me straight back to my own corner of the system, wherever I've wandered off to.

I think of `cd ~` as **the reset button**. Lost in a maze of folders? Not sure where you are? `cd ~`, and you're home. Safe. Back to your own user. It doesn't undo anything — it resets *you*, your location, back to a place you recognize.

```bash
cd          # lazy version — also goes home
cd -        # bonus: jump back to the PREVIOUS folder, very handy
```

That phrase — *back to your own user* — is the part that kept nagging at me. Which user? **Whose** home am I going to? That question turned out to be the whole rest of this document.

---

## Part 2 — Whose home am I going to? (the `root` user)

On a Linux system, **everything is tied to a user**. Every file has an owner, every process runs as somebody, every command you type is done *by* somebody. So `cd ~` is really saying "take me to the home folder of whichever user I'm currently logged in as."

`whoami` tells me who that is:

```bash
whoami
# → yourname        (a normal human)
```

And then there's the big one. Linux has one super-user called **root**. Root can do anything — read any file, change any setting, delete anything, including things it probably shouldn't. Root isn't a person; root is a power level.

So naturally you'd want to know what it's like:

```bash
sudo whoami
# → root            (the boss)
```

Those two commands side by side are basically the entire Linux security model in miniature: **I am a normal user with limits, and root is the exception I have to ask permission to become.** You don't log in as root for day-to-day work — you log in as yourself and escalate only when you genuinely need to. That's why `sudo` exists, and why it asks for your password.

---

## Part 3 — Making users

If the system is built around users, I need to know how to make them. There are two commands, and they are not the same:

```bash
sudo adduser alice     # friendly and interactive: sets up a home dir, asks questions
sudo useradd alice     # low-level and silent: creates the user and nothing else
```

On Ubuntu/Debian I reach for `adduser` because it holds my hand — it creates `/home/alice`, prompts for a password, sets sensible defaults. `useradd` is what the system itself uses underneath: powerful, but it won't do a single thing you didn't explicitly ask for.

Checking that users exist:

```bash
cat /etc/passwd        # the classic "list of users" file — readable by everyone
id alice               # alice's user ID, group ID, and groups
```

---

## Part 4 — Passwords

A user without a password is a door without a lock, so:

```bash
sudo passwd alice      # set or change alice's password (as an admin)
passwd                 # change MY OWN password
```

Things I learned the hard way:

- `passwd` on its own changes **your** password. `sudo passwd someone` changes **theirs**. One word of difference, completely different meaning.
- Passwords are never stored in plain text. They live hashed in `/etc/shadow`, which normal users can't even read — that's why changing someone else's password needs `sudo`.
- `sudo passwd -l alice` **locks** an account (drops a `!` in front of the hash) without deleting anything, and `-u` unlocks it. Locking is the polite way to disable an account without destroying its files or history.

```bash
sudo passwd -l alice   # lock
sudo passwd -u alice   # unlock
```

---

## Part 5 — Groups

Groups are how Linux hands permissions to *several* people at once instead of one at a time. Every user has a **primary group** and can belong to as many **supplementary groups** as you like.

```bash
sudo groupadd developers           # create a group
sudo usermod -aG developers alice  # add alice to it
```

Those flags matter: **`-a`** means *append* (don't remove her from her other groups) and **`-G`** means *supplementary groups*. Forget the `-a` and you overwrite her entire group list. Classic beginner mistake — ask me how I know.

```bash
groups alice       # which groups is alice in?
id alice           # the fuller picture
```

So now I have users, passwords, and groups. But what are they all *for*?

---

## Part 6 — And this is where permissions come in

**Everything in Linux comes down to permissions.** Users and groups aren't admin busywork — they exist so the system can answer one question about every single file:

> *Who is allowed to read this, write to this, or execute this?*

Every file has three permission slots — **owner**, **group**, and **others** — and each slot can carry **read (r)**, **write (w)**, and **execute (x)**. That's where strings like `rwxr-xr--` come from. Once that clicked, the whole users-and-groups chapter stopped feeling like paperwork and started feeling like the actual point of the system.

---

## Part 7 — Seeing it all: `ls` and `ls -la`

`ls` lists what's in a directory:

```bash
ls
```

But plain `ls` hides everything interesting — the hidden files (anything starting with a dot) and all the permission information. So:

```bash
ls -la
```

Two flags doing the heavy lifting:

- **`-l`** = long format: permissions, owner, group, size, date, for each entry.
- **`-a`** = all: includes hidden files like `.bashrc`, plus the special `.` (this folder) and `..` (parent folder — the exact same `..` from `cd ..`!).

The output looks like this:

```bash
$ ls -la
drwxr-xr-x  5 alice developers 4096 Nov 22 10:14 .
drwxr-xr-x 20 root  root       4096 Nov 22 09:58 ..
-rw-r--r--  1 alice developers  220 Nov 20 18:02 .bash_logout
-rw-r--r--  1 alice developers 3771 Nov 20 18:02 .bashrc
drwxr-xr-x  2 alice developers 4096 Nov 22 10:14 notes
```

Reading one line, left to right:

| Piece | Means |
|---|---|
| `d` or `-` | directory or regular file |
| `rwxr-xr-x` | owner can read/write/execute; group can read/execute; others can read/execute |
| `alice` | the owner |
| `developers` | the group |
| `4096` | size in bytes |
| `Nov 22 10:14` | last modified |
| `notes` | the name |

And there at the top are `.` and `..` — the same folders `cd ..` has been using all along. They're not special commands, just entries in the directory, and the permission system treats them exactly like everything else.

---

## Part 8 — Did you know: no `x`, no entry

Here's the thing that genuinely delighted me, and something tells me in the real world this is a biggie.

**If you don't have the execute (`x`) permission on a folder, you cannot open it.** Not "can't see what's inside" — can't *enter* it at all. `cd` into it and you get `Permission denied`. You can't even reach a file inside it if you know the exact path.

On a **file**, `x` means "run this as a program." On a **folder**, `x` means "walk through this door." And a folder needs all three bits doing three different jobs:

- **`r` (read)** — you can *list* the names inside (`ls` works).
- **`x` (execute)** — you can *enter and pass through* it (`cd` works, and files inside become reachable).
- **`w` (write)** — you can *create and delete* entries inside it.

Which gives you some wonderfully weird combinations:

- `r` but no `x`: you can see the names in a folder but you can't go in or open anything. Like reading a menu through a locked window.
- `x` but no `r`: you can't list the folder — but if you already know a file's name, you can still walk in and use it. Like a corridor with no lights but you know the way.

Why this matters in the real world: **`x` on a folder is the gatekeeper for everything inside it.** If you can't traverse the directory, the permissions on the files inside are irrelevant — you never get to them. That's exactly why `/root` is locked down to `700` (owner only, no `x` for anyone else) and why home directories on multi-user systems are often `750`. One bit on one folder can hide an entire subtree from the rest of the machine. Biggie confirmed.

And yes — this is also why `ls -la` exists in my life now. I want to *see* the bits before I trip over them.

---

## Part 9 — `ls` again, but deeper: `-R` (and `-r`)

The flag I didn't know I needed is uppercase **`-R`**, for **recursive**. It lists the current folder, then every folder inside it, then every folder inside those, all the way down. It's how you see the shape of a whole project in one shot instead of `cd`-ing twenty times.

While I'm here, a warning that cost me five minutes of confusion: uppercase `-R` and lowercase `-r` are **not** the same thing.

- `ls -R` = **R**ecursive. Descends into subfolders.
- `ls -r` = **r**everse. Just flips the sort order (normally alphabetical, so app → backup → reports becomes reports → backup → app).

One letter of case, two completely different superpowers. Which is exactly how I found out what was in my lab folder:

```bash
admin_paul@DESKTOP-TMVN5BI:~/cloud-lab$ ls -r
reports  backup  app
```

There they are, in reverse: `app`, `backup`, `reports`. That's my lab — this is where I have all the fun. Now let's build the thing.

---

## Part 10 — `mkdir`, and building a whole lab in one line

The basic move is obvious:

```bash
mkdir cloud-lab
```

But the real trick is `-p`, which stands for **parents**. It does two things: it creates any missing parent folders along the way, and it **doesn't complain if the folder already exists**. That second part is why `-p` is the safe default — running the same command twice just... works.

Then comes the move that made me sit up straight. The shell lets you write **brace expansion**:

```bash
mkdir -p ~/cloud-lab/{folderA,folderB,folderC}
```

One command, three folders, and the shell itself did the work. Before `mkdir` even runs, bash expands that into:

```bash
mkdir -p ~/cloud-lab/folderA ~/cloud-lab/folderB ~/cloud-lab/folderC
```

So brace expansion isn't a `mkdir` feature — **it's a shell feature**, and it works with `touch`, `cp`, `rm`, basically anything that takes filenames. That's the part worth internalizing. And because `-p` makes parents, you can nest braces to build an entire tree in one shot:

```bash
mkdir -p ~/cloud-lab/{app/{config,data,logs},backup,reports}
```

That single line creates `cloud-lab`, then `app` inside it, then `config`, `data`, and `logs` inside `app`, plus `backup` and `reports` next to it. Six folders, one command, no `cd`, no fuss.

One gotcha, learned painfully: **no spaces after the commas.** `{folderA, folderB}` with a space creates a folder literally named `␣folderB` — leading space and all — because you told it to. The space is part of the name. Braces are literal-minded.

---

## Part 11 — `touch`: filling the folders

Now I had a problem. My folders existed but they were **empty**, and empty folders aren't a lab. I had a challenge: I want *things* in those folders.

Enter `touch`. Two jobs, one command:

- If the file **doesn't exist**, it creates it — empty, zero bytes.
- If the file **does exist**, it does nothing except update its "last modified" timestamp.

That second behaviour is where the name comes from: it *touches* the file to make it look freshly edited.

```bash
touch notes.txt                          # one file
touch notes.txt ideas.txt todo.txt       # several at once
touch ~/cloud-lab/reports/{notes.txt,server-status.txt}   # brace expansion again
```

So I filled the lab, folder by folder:

```bash
touch ~/cloud-lab/app/config/app.conf
touch ~/cloud-lab/app/data/users.txt
touch ~/cloud-lab/app/logs/words.txt
touch ~/cloud-lab/app/logs/{access/access.log,error/error.log,system/{app.log,login.log,security.log}}
touch ~/cloud-lab/reports/{notes.txt,server-status.txt}
```

Then two files I'll explain in the next section, because they're the interesting ones:

```bash
cp ~/cloud-lab/app/config/app.conf ~/cloud-lab/backup/app.conf.bak.restored
ln ~/cloud-lab/reports/server-status.txt ~/cloud-lab/backup/server-status.txt.hard
```

And just in passing — here's what the lab looks like now, with `ls -R` doing its recursive thing:

```bash
admin_paul@DESKTOP-TMVN5BI:~/cloud-lab$ ls -R
.:
app  backup  reports

./app:
config  data  logs

./app/config:
app.conf

./app/data:
users.txt

./app/logs:
access  error  system  words.txt

./app/logs/access:
access.log

./app/logs/error:
error.log

./app/logs/system:
app.log  login.log  security.log

./backup:
app.conf.bak.restored  server-status.txt.hard

./reports:
notes.txt  server-status.txt
admin_paul@DESKTOP-TMVN5BI:~/cloud-lab$
```

Nine folders, a dozen files, all from a handful of lines. Notice the two odd names in `./backup` — `.bak.restored` and `.hard`? Those were experiments in the next section.

---

## Part 12 — Links: shortcuts vs two doors to the same house

Two files can relate to each other in two completely different ways on Linux, and confusing them is a rite of passage.

### Symbolic link (`ln -s`) — the Windows shortcut

```bash
ln -s /path/to/target  linkname
```

A symlink is a tiny file whose entire content is "go look over there instead." It's a signpost. If the target moves or gets deleted, the signpost keeps pointing at nothing — you get a **dangling link**, and `ls -l` shows it in angry red.

Think of it as a shortcut on a Windows desktop: double-click it and you're at the real file, but delete the real file and the shortcut is just a dead arrow.

### Hard link (`ln`) — two doors, one house

```bash
ln /path/to/target  linkname
```

A hard link is **not** a shortcut. It's a second *name* for the exact same file on disk. Both names point at the same data — same inode (the number the filesystem uses to identify the file), same size, same contents. There's no "original" and "copy"; there's one file with two front doors.

Which means:

- Edit through either name → both show the change, because it's one file.
- Delete one name → the file is still there via the other. The data only vanishes when the **last** name is gone.
- A hard link can't dangle. There's no target to lose.

The house analogy is the one that stuck for me: **a symlink is a signpost pointing at a house — move the house and the sign is useless. A hard link is the same house with two front doors on two different streets — knock on either, you're in the same living room, and closing one door doesn't demolish anything.**

### Seeing the difference

```bash
ls -li ~/cloud-lab/reports/server-status.txt ~/cloud-lab/backup/server-status.txt.hard
```

`-i` shows the inode number. Both names report the **same** inode — proof they're one file. A symlink, meanwhile, would show up in `ls -l` starting with `l` and an arrow: `linkname -> /path/to/target`.

### Why this matters beyond my backup folder

Once you know about links, you start seeing them **everywhere** on a Linux system. A huge amount of what looks like "different things" is really one function with several names:

- `sh` is usually a symlink to `bash` — different name, same program.
- Many systems have a dozen commands that are just links to one shared binary (that's how `busybox` fits 300 tools into one file).
- Package updates often "move" a command by simply repointing a symlink at a new version, leaving the old one in place for rollback.
- And inside `/usr/bin` you'll find chains of links where the real executable lives somewhere else entirely.

Different names, one function underneath. That's the whole idea, and it's baked into the operating system itself — not just a trick for saving disk space.

---

## Cheat sheet (for the skimmers)

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
| `ls -R` | list recursively, every subfolder |
| `ls -r` | list in reverse order (not the same as `-R`!) |
| `mkdir -p a/b/c` | make nested folders, no complaints if they exist |
| `mkdir -p dir/{x,y,z}` | brace expansion — three folders, one command |
| `touch file` | create an empty file (or update its timestamp) |
| `ln target link` | hard link — two names, one file |
| `ln -s target link` | symbolic link — a shortcut |
| `ls -li` | see inode numbers, to prove a hard link |

---

## Where I'm headed next

`chmod`, `chown`, and `sudo` in more depth — because now that I understand what the `x` bit does on a folder and why permissions are the real gatekeepers, I want to learn how to actually *change* them instead of just reading them.

*This is a living document. It gets updated as I learn. Corrections welcome — that's how I get better.*