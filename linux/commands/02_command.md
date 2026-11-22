# Learning Linux — Level 2: The `x` Bit, Building a Lab, and the Magic of Links

New lesson, new file. Level 1 was about users, groups, and permissions in theory. This one is about actually *touching* the filesystem — making folders, filling them, and discovering that half of what looks like "different things" on a Linux box is really one thing wearing several names.

If you're a recruiter skimming this: this is me showing my work, not pretending to be a sysadmin. Yet. 🙂

---

## The thing that blew my mind first

Did you know that **if you don't have the execute (`x`) permission on a folder, you cannot open it?** Not "can't see what's inside" — can't *enter* it at all. Try to `cd` in and you get `Permission denied`. Even if you know the exact path to a file inside, you can't reach it. The door is locked and the `x` bit is the key.

On a **file**, `x` means "run this as a program." On a **folder**, `x` means "walk through this door." Same bit, two jobs. And a folder actually needs all three doing three different things:

- **`r` (read)** — you can *list* the names inside (`ls` works).
- **`x` (execute)** — you can *enter and pass through* it (`cd` works, and files inside become reachable).
- **`w` (write)** — you can *create and delete* entries inside it.

Just some wonderfully weird combinations that I texted:

- `r` but no `x`: you can see the names in a folder, but you can't go in or open anything. Like reading a menu through a locked window.
- `x` but no `r`: you can't list the folder — but if you already know a file's name, you can still walk in and use it. A corridor with no lights where you already know the way.

Something tells me in the real world this is a biggie. It's the reason `/root` is locked down to `700` (owner only, no `x` for anyone else), and why home directories on shared machines are often `750`. **one folder can hide an entire subtree from the rest of the machine.** I am yet to Confirm biggie Becasue I am yet to work with really massive folders. Maybe today we learn how to make huge folders and files?😎✔.

And yes — this is why `ls -la` is now permanently in my muscle memory. 

---

## `ls` — and the flag that changed how I see folders

Plain `ls` lists what's in a directory. But the flag I didn't know I needed is uppercase **`-R`**, for **recursive**. It lists the current folder, then every folder inside it, then every folder inside *those*, all the way down. It's how you see the shape of a whole project in one shot instead of `cd`-ing twenty times.

And here's a warning that cost me five minutes of confusion: **uppercase `-R` and lowercase `-r` are not the same thing.**

- `ls -R` = **R**ecursive. Descends into subfolders.
- `ls -r` = **r**everse. Just flips the sort order.

One letter of case, two completely different superpowers. Which is exactly how I found out what was in my lab folder:

```bash
admin_paul@DESKTOP-TMVN5BI:~/cloud-lab$ ls -r
reports  backup  app
```

There they are, in reverse: `app`, `backup`, `reports`. That's my lab — `cloud-lab` is where I have all the fun. Now let's build the thing.

---

## `mkdir` — and building a whole lab in one line

The basic move is obvious:

```bash
mkdir cloud-lab
```

But the real trick is `-p`, which stands for **parents**. It does two things: it creates any missing parent folders along the way, and it **doesn't complain if the folder already exists** Very Important. 
Also:
That second part is why `-p` is the safe default — running the same command twice just... works.

Then comes the move that made me sit up straight: **brace expansion**:

```bash
mkdir -p ~/cloud-lab/{folderA,folderB,folderC}
```

One command, three folders. And before `mkdir` even runs, the shell expands that into:

```bash
mkdir -p ~/cloud-lab/folderA ~/cloud-lab/folderB ~/cloud-lab/folderC
```

So brace expansion isn't a `mkdir` feature — **it's a shell feature**GOOGLE what SHELL vs TERMINAL, and it works with `touch`, `cp`, `rm`, basically anything that takes filenames. 
And because `-p` creates parents, you can nest braces to build an entire tree in one shot:

```bash
mkdir -p ~/cloud-lab/{app/{config,data,logs},backup,reports}
```

That single line creates `cloud-lab`, then `app` inside it, then `config`, `data`, and `logs` inside `app`, plus `backup` and `reports` next to it. Six folders, one command, no `cd`, no fuss.

One gotcha, learned painfully: **no spaces after the commas.** `{folderA, folderB}` with a space creates a folder literally named `␣folderB` — leading space and all — because you told it to GIGO. The space is part of the name. 

---

## The challenge: filling the folders

Now I had a problem. My folders existed but they were **empty**, and I want *things* in those folders.

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

## Links: shortcuts vs two doors to the same house

Two files can relate to each other in two completely different ways on Linux, and confusing them is a rite of passage.

### Symbolic link (`ln -s`) — the Windows shortcut

```bash
ln -s /path/to/target linkname
```

A symlink is a tiny file whose entire content is "go look over there instead." It's a signpost. If the target moves or gets deleted, the signpost keeps pointing at nothing — you get a **dangling link**, and `ls -l` shows it in angry red.

Think of a shortcut on a Windows desktop: double-click it and you're at the real file, but delete the real file and the shortcut is just a dead arrow.

### Hard link (`ln`) — two doors, one house

```bash
ln /path/to/target linkname
```

A hard link is **not** a shortcut. It's a second *name* for the exact same file on disk. Both names point at the same data — same inode (the number the filesystem uses to identify the file), same size, same contents. There's no "original" and "copy"; there's one file with two front doors.

Which means:

- Edit through either name → both show the change, because it's one file.
- Delete one name → the file is still there via the other. The data only vanishes when the **last** name is gone.
- A hard link can't dangle. There's no target to lose.

The house analogy is the one that stuck: **a symlink is a signpost pointing at a house — move the house and the sign is useless. A hard link is the same house with two front doors on two different streets — knock on either, you're in the same living room, and closing one door doesn't demolish anything.**

### Seeing the difference

```bash
ls -li ~/cloud-lab/reports/server-status.txt ~/cloud-lab/backup/server-status.txt.hard
```

`-i` shows the inode number. Both names report the **same** inode — proof they're one file. A symlink, meanwhile, would show up in `ls -l` starting with `l` and an arrow: `linkname -> /path/to/target`.

### Why this matters beyond my backup folder

Once you know about links, you start seeing them **everywhere** on a Linux system. A huge amount of what looks like "different things" is really one function with several names:

- `sh` is usually a symlink to `bash` — different name, same program.
- Many systems have a dozen commands that are just links to one shared binary (that's how `busybox` fits 300 tools into one file). What exactly is this>>> MMh lets take a step at a time or YT it.
- Package updates often "move" a command by simply repointing a symlink at a new version, leaving the old one in place for rollback.
- And inside `/usr/bin` you'll find chains of links where the real executable lives somewhere else entirely.

Different names, one function underneath. That's the whole idea, and it's baked into the operating system itself — not just a trick for saving disk space.

---

## Todays Cheat sheet (for the skimmers)

| Command | What it does |
|---|---|
| `ls -R` | list recursively, every subfolder |
| `ls -r` | list in reverse order (not the same as `-R`!) |
| `ls -li` | long listing with inode numbers |
| `mkdir -p a/b/c` | make nested folders, no complaints if they exist |
| `mkdir -p dir/{x,y,z}` | brace expansion — three folders, one command |
| `touch file` | create an empty file (or update its timestamp) |
| `touch a.txt b.txt c.txt` | create several files at once |
| `cp src dest` | copy a file |
| `ln target link` | hard link — two names, one file |
| `ln -s target link` | symbolic link — a shortcut |

---

## Where I'm headed next

`chmod` and `chown` and `grep,m find` I want to learn how to actually *change* permissions instead of just reading them and searching... do i need to explain why searching hehe.

*This is a living document. It gets updated as I learn. Corrections welcome — that's how I get better.*