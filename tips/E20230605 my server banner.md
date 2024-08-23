## my server banner

### /etc/update-motd.d

vi /etc/update-motd.d/00-header
```
#!/bin/sh
[ -r /etc/lsb-release ] && . /etc/lsb-release

if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
	# Fall back to using the very slow lsb_release utility
	DISTRIB_DESCRIPTION=$(lsb_release -s -d)
fi
echo " "
echo "   ___  ___ ___(_) _ __ __     __      __"
echo "  / _ \/ __/ __| | '_   _ \/ __\ \ /\ / /"
echo " |  __/ (__\__ \ | | | | | \__ \\ V  V / "
echo "  \___|\___|___/_|_| |_| |_|___/ \_/\_/  "
echo " "
printf "Welcome to ecsimsw server :)\n"
printf "%s (%s %s %s)\n" "$DISTRIB_DESCRIPTION" "$(uname -o)" "$(uname -r)" "$(uname -m)"
echo "CPU `LC_ALL=C top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}'`% RAM `free -m | awk '/Mem:/ { printf("%3.1f%%", $3/$2*100) }'` HDD `df -h / | awk '/\// {print $(NF-1)}'`"
echo ""
```

### remove all the other files in /etc/update-motd.d
except '00-header'    

### turn off 'last login option' 
vi /etc/ssh/sshd_config

```
PrintLastLog no
```
### and restart ssh
```
service ssh restart
```

### result
<img width="422" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/efeb2e66-961a-4add-baa8-96a50864fc60">
