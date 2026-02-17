---
title: .gitignore in Tar and Rsync
lang: en
date: 2025-08-13 17:44:40
updated: 2025-08-18 15:32:52
copyright_author: Maltune
copyright_author_href: https://keep.me.anonymo.us/
tags:
---


Sometimes we need to archive or transfer Git repos without Git's remote feature, and it will be convenient if we could reuse the `.gitignore` files to exclude unneeded files and directories. Luckily, two of the popular tools in doing so, Tar and Rsync, provide feature to retrieve excluding rules from files. However, there are some differences between Git's, Tar's and Rsync's understanding of excluding files. Here are the problems I encountered in messing with them.

# Tar
GNU Tar supports the option `--exclude-vcs-ignores`, which respects `.cvsignore`, `.gitignore`, `.bzrignore` and `.hgignore`, `.gitignore` in my case. It provides an approximation of `.gitignore`, however:

- It does not support double star (`**/foo`)
- It does not support leading and trailing slashes (`/foo` and `foo/`)
- It does not support negation (`!foo`)

and more. So use it with care when `.gitignore` is complex. Always dry-run it before any real-world operation is a good parctice. Since Tar doesn't have an explicit `--dry-run` option, we can use `-cvf /dev/null` to see what it will pack in creation and `-tf <archive>` to list the content of the archive in extraction.

# Rsync
Rsync's option `-f/--filter` controls its filter rules, and `-f ':- .gitignore'` (no space between `:` and `-`) tells Rsync to treat the content of `.gitignore` file in each directory as its excluding rules. That is to say, this works only if the rules in `.gitignore` happen to be interpreted the same way by both Git and Rsync. Here are some main differences between their interpretations:

- Rsync's shell-style globbing is slightly different from Git
- Leading slashes in Rsync anchors the matching from where the `.gitignore` that contains the rule is found, not repo root like Git
- Negation doesn't work, apparently, as Rsync uses `+` and `-` for including and excluding

Since Rsync's filter rules and `.gitignore` are completely different, **use `.gitignore` as Rsync filters only if its rules are *really* simple and you *really* need a quick dirty way to do that**, otherwise write a seperate `rsync-filters.txt` instead.

Also, Rsync provides a convenient option `-F`, whose first and second appearance are equivalent to `-f ': /.rsync-filter'` and `-f '- .rsync-filter'`, respectively. As for packing, we can Rsync the directory to somewhere else, e.g. `/tmp`, then add `-C /tmp` to Tar's argument list.
