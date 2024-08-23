## zsh config (ecsimsw-server)

### Install zsh
```
sudo apt install zsh
chsh -s /usr/bin/zsh
```

### ~/.zshrc
```
PS1='%n@%m %~$ '

source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh

[[ -s /home/ecsimsw/.autojump/etc/profile.d/autojump.sh ]] && source /home/ecsimsw/.autojump/etc/profile.d/autojump.sh
autoload -U compinit && compinit -u

HISTFILE=~/.histfile
HISTSIZE=1000
SAVEHIST=1000
setopt appendhistory
```
