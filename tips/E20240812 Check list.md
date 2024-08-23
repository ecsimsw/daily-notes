## Check list
- chrome
- homebrew
- docker
- clipy
- itsycal
- autojump
- autosuggestions
- intellij / jumper
- spectacle
- dozer

### zshrc
```
export PS1="%1~ %# "
export PATH=/opt/homebrew/bin:$PATH

[ -f /opt/homebrew/etc/profile.d/autojump.sh ] && . /opt/homebrew/etc/profile.d/autojump.sh

plugins=(zsh-autosuggestions)
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh

export PATH="$HOME/.jenv/bin:$PATH"
eval "$(jenv init -)"
export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"
```
