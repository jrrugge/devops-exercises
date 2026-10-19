# My DevOps Journey

- **Started:** _YYYY-MM-DD_
- **GitHub:** @jrrugge

> Log what you learn as you go. Commit + push to your fork after each session.

---

## 01 · Linux Fundamentals

- [ ] 01 - The Terminal

<img width="1067" height="557" alt="image" src="https://github.com/user-attachments/assets/38b46f53-d753-4c37-8e8e-f2c2afb65375" />

- [ ] 02 - The Shell
 
<img width="771" height="352" alt="image" src="https://github.com/user-attachments/assets/783881e0-8656-4bce-84f7-9a925ae22cd8" />

Commands: 
1.1. Save the list of installed shells:
cat /etc/shells

cat /etc/shells > ~/sandbox/shells.txt
That file is /etc/shells. The line above copies it into your sandbox.

2.1. Make zsh the shell you get every time you log in:
sudo chsh -s "$(which zsh)" $USER

Put your settings in ~/.zshrc. Covers the alias, the theme, the team aliases, and the git plugin:

cat > ~/.zshrc << 'EOF'
alias ll='ls -lah'
alias gs='git status'
alias k='kubectl get pods'
alias dc='docker compose'
ZSH_THEME="agnoster"
plugins=(git)
EOF

ll is a shortcut for ls -lah. gs, k, and dc are the team shortcuts. ZSH_THEME picks a theme. plugins=(git) turns on the git plugin.

- [ ] 03 - Linux Commands

<img width="1117" height="330" alt="image" src="https://github.com/user-attachments/assets/e7118ca9-def6-4e4c-994f-4153b746e0be" />

history
1. find all the files flag
ls -a (--all)
ls -a ~/sandbox/files > ~/sandbox/hidden.txt

1.2 date
date +%F > ~/sandbox/today.txt

2 search man page descriptions by keyword
apropos "space usage" > ~/sandbox/apropos-space.txt

locate the program
which du > ~/sandbox/where-is-du.txt

man ls
ls -l ~/sandbox/files > ~/sandbox/long.txt #show permissions, owner, sixe per file

append the toolkit to ~/.zhrc
cat >> ~/.zhrc <<'EOF'
alias gl='git log --oneline'
alias dps='docker ps'
alias kgp='kubectl get pods'
EOF

- [ ] 04 - The Linux File System

<img width="782" height="275" alt="image" src="https://github.com/user-attachments/assets/ae7d1e54-f222-4f2f-9f78-c2f8f530cc32" />
  
- [ ] 05 - Working with Files
- [ ] 06 - Working with Directories
- [ ] 07 - Users & Groups
- [ ] 08 - File Permissions

### Notes & takeaways
-

### Commands I want to remember
```bash

```

### Gotchas / things that confused me
-

---

<!-- Copy the block above for each new course you start. -->
