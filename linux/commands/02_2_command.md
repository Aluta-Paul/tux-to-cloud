# Learning Linux — Mini Note: Copy, Remove, Rename

Quick side note before the next big topic. Three commands, all dangerous in their own way, all things I'll type a thousand times. `cp`, `rm`, `mv` — copy, remove, and move/rename.

If you're a recruiter skimming this: this is me showing my work, not pretending to be a sysadmin. Yet. 🙂

---

## `cp` — copy

```bash
cp notes.txt notes.bak              # copy a file
cp notes.txt ~/cloud-lab/reports/   # copy into a folder
cp a.txt b.txt c.txt ~/backup/      # several files at once
cp -r ~/cloud-lab ~/cloud-lab-copy  # -r = recursive, needed for folders
```

Two flags worth knowing from day one:

- **`-r`** — recursive. **Without it, `cp` refuses to copy a folder.** Every beginner hits this: `cp: -r not specified; omitting directory`. It's not broken; it's being careful.
- **`-i`** — interactive, asks before overwriting.

That second one matters because **`cp` silently overwrites by default.** No prompt, no warning, the old file is just gone. `cp -i` (or `cp -n` to refuse overwrites outright) is the seatbelt.

```bash
cp -i important.txt important.txt.bak   # asks first
cp -n important.txt important.txt.bak   # never overwrites, no questions
cp -v file.txt ~/backup/                # -v = verbose, shows what it did
```

`-v` is underrated. When copying a tree, I love it cause it tells me exactly what landed where instead of leaving me guessing.

---

## `rm` — remove

```bash
rm notes.txt                  # delete a file
rm a.txt b.txt c.txt          # several at once
rm -r old-folder/             # -r = recursive, needed for folders
rm -f stubborn.txt            # -f = force, no prompts, ignore missing
rm -i *.txt                   # -i = ask before EACH deletion
```

**`rm` is the most dangerous command in this note, and it's not close.**

- There is **no trash can**. `rm` unlinks the file immediately. No undo, no recycle bin, no "are you sure."
- `rm -rf folder/` is the famous one — recursive *and* forced, so it deletes everything inside without a single prompt.
- **`rm -rf /` will destroy a system.** , but `rm -rf /*` and a hundred variations slip past it. From redits and X i have learned never type `-rf` with a wildcard unless you've stared at the path for five seconds first, even me i am yet to run this command let me look for a lab online and test.

My habits, formed by nearly making each of these mistakes:

```bash
ls folder/           # look FIRST, then delete
rm -i folder/*       # ask per file when I'm unsure
rm -r folder/        # only after I've seen what's in there.. mostly after i confirmed the file i created just to dlete realy lives there.
```


---

## `mv` — move, and rename

Here's the thing that surprised me: **there is no `rename` command for this.** Renaming *is* moving — you move a file to a new name in the same folder.

```bash
mv old.txt new.txt              # rename (same folder, new name)
mv notes.txt ~/cloud-lab/       # move into another folder
mv ~/cloud-lab/app ~/archive/   # move a whole folder, no -r needed
mv *.log ~/logs/                # move everything matching a pattern
```

Two key differences from `cp`:

- **No `-r` required for folders.** `mv` moves the whole thing regardless. It doesn't copy anything — it just repoints the name (within the same filesystem it's instant, even for huge folders).
- **It overwrites silently too.** `mv a.txt b.txt` destroys `b.txt` without asking. Same seatbelt applies:

```bash
mv -i old.txt new.txt      # ask before overwriting
mv -n old.txt new.txt      # never overwrite
mv -v old.txt new.txt      # show what happened
```

---

## The one rule that covers all three

**`cp`, `rm`, and `mv` all overwrite or destroy silently by default.**

`cp` overwrites the destination. `mv` overwrites the destination. `rm` overwrites nothing — it just deletes.

So the habit that saves me every time: **run `ls` on the target first.** Two seconds of looking beats an hour of recovering.

```bash
ls ~/backup/          # what's already there?
cp notes.txt ~/backup/
```


---

## Alsmost forgot Cheat sheet 

| Command | What it does |
|---|---|
| `cp file dest` | copy a file |
| `cp -r dir dest` | copy a folder (`-r` required) |
| `cp -i` / `-n` | ask before overwrite / never overwrite |
| `rm file` | delete a file — no trash, no undo |
| `rm -r dir` | delete a folder and everything in it |
| `rm -rf dir` | force, no prompts — the dangerous one |
| `rm -i` | ask before each deletion |
| `mv old new` | rename |
| `mv file dir/` | move into a folder |
| `mv dir/ ~/archive/` | move a whole folder, no `-r` needed |

---

*Small note, part of a living document. Corrections welcome — that's how I get better.*