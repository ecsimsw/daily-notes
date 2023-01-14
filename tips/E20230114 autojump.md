## autojump

https://github.com/wting/autojump

#### brew install 

```
brew install autojump
```

#### Add the following line to your ~/.bash_profile or ~/.zshrc file:

```
[ -f $(brew --prefix)/etc/profile.d/autojump.sh ] && . $(brew --prefix)/etc/profile.d/autojump.sh
```

#### if it not works

vi ~/.zshrc

```
plugins=(autojump)
```
