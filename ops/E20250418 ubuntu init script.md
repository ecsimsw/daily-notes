<img width="477" alt="스크린샷 2025-04-18 오전 1 23 37" src="https://github.com/user-attachments/assets/8ff8eac1-1816-4179-b9f4-da4a62c0dc79" />

``` bash
#!/bin/bash

set -e

echo "Ubuntu init script by ecsimsw."
echo

if [[ $EUID -ne 0 ]]; then
    echo "ERROR :: You need to be root to run this script"
    exit 1
fi

read -p "user name (enter for skip) : " NEWUSER
read -p "java version (enter for skip) : " JAVAVER

if [[ -n "$NEWUSER" ]]; then
    if id "$NEWUSER" &>/dev/null; then
        echo "ERROR :: user already exists"
        exit 1
    fi
    useradd -m -s /bin/bash "$NEWUSER"
    usermod -aG sudo "$NEWUSER"
    usermod -aG docker "$NEWUSER"
    chsh -s "$(which zsh)" "$NEWUSER"
    echo "$NEWUSER created, added to docker group, and zsh set as default shell."
fi

apt-get update > /dev/null 2>&1

apt-get install -y zsh > /dev/null 2>&1
echo "zsh installed"
apt-get install -y net-tools > /dev/null 2>&1
echo "net-tools installed"
apt-get install -y curl > /dev/null 2>&1
echo "curl installed"
apt-get install -y wget > /dev/null 2>&1
echo "wget installed"
apt-get install -y ufw > /dev/null 2>&1
echo "ufw installed"
apt-get install -y vim > /dev/null 2>&1
echo "vim installed"
apt-get install -y unzip > /dev/null 2>&1
echo "unzip installed"
apt-get install -y tree > /dev/null 2>&1
echo "tree installed"
apt-get install -y git > /dev/null 2>&1
echo "git installed"
apt-get install -y htop > /dev/null 2>&1
echo "htop installed"
apt-get install -y ncdu > /dev/null 2>&1
echo "ncdu installed"

if ! command -v aws > /dev/null 2>&1; then
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip > /dev/null 2>&1
    unzip -q /tmp/awscliv2.zip -d /tmp
    /tmp/aws/install > /dev/null 2>&1
    rm -rf /tmp/aws /tmp/awscliv2.zip
    if command -v aws > /dev/null 2>&1; then
        echo "aws cli installed"
    else
        echo "ERROR :: aws cli install failed"
    fi
else
    echo "aws cli installed"
fi

if [[ -n "$JAVAVER" ]]; then
    apt-get install -y "openjdk-${JAVAVER}-jdk" > /dev/null 2>&1
    echo "java installed"
    JAVA_PATH=$(update-java-alternatives -l | grep "java-${JAVAVER}-openjdk" | awk '{print $3}')
    if [[ -z "$JAVA_PATH" ]]; then
        JAVA_PATH=$(dirname $(dirname $(readlink -f $(which javac))))
    fi
    if [[ -n "$NEWUSER" ]]; then
        echo "export JAVA_HOME=$JAVA_PATH" >> /home/"$NEWUSER"/.zshrc
        echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /home/"$NEWUSER"/.zshrc
        chown "$NEWUSER":"$NEWUSER" /home/"$NEWUSER"/.zshrc
        echo "java env added to .zshrc"
    fi
fi

apt-get install -y ca-certificates > /dev/null 2>&1
echo "ca-certificates installed"
apt-get install -y gnupg > /dev/null 2>&1
echo "gnupg installed"
apt-get install -y lsb-release > /dev/null 2>&1
echo "lsb-release installed"
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg > /dev/null 2>&1
echo "docker gpg key added"

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update > /dev/null 2>&1

apt-get install -y docker-ce > /dev/null 2>&1
echo "docker-ce installed"
apt-get install -y docker-ce-cli > /dev/null 2>&1
echo "docker-ce-cli installed"
apt-get install -y containerd.io > /dev/null 2>&1
echo "containerd.io installed"
apt-get install -y docker-compose-plugin > /dev/null 2>&1
echo "docker-compose-plugin installed"
apt-get install -y docker-compose > /dev/null 2>&1
echo "docker-compose installed"

if [[ -n "$NEWUSER" ]]; then
    sudo -u "$NEWUSER" bash <<'EOF'
if [ ! -d ~/.zsh/zsh-autosuggestions ]; then
    git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions > /dev/null 2>&1
fi
if ! grep -q "zsh-autosuggestions.zsh" ~/.zshrc; then
    echo '
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
' >> ~/.zshrc
fi
EOF
fi

CURRENT_USER="${SUDO_USER:-$(whoami)}"
usermod -aG docker "$CURRENT_USER"
chsh -s "$(which zsh)" "$CURRENT_USER"
sudo -u "$CURRENT_USER" bash <<'EOF'
if [ ! -d ~/.zsh/zsh-autosuggestions ]; then
    git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions > /dev/null 2>&1
fi
if ! grep -q "zsh-autosuggestions.zsh" ~/.zshrc; then
    echo '
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
' >> ~/.zshrc
fi
EOF
echo "$CURRENT_USER is now in docker group and zsh is set as default shell."

echo
echo "finished"
```
