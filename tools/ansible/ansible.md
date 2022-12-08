# Ansible Cheat-Sheet
## Install Homebrew

``
```bash
  echo "-> Installing Homebrew and Ansible..."

  curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh | bash

  test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"

  test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

  test -r ~/.bash_profile && echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.bash_profile

  test -r ~/.zshrc && echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.zshrc
  
  echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.profile
  
  test -r ~/.bash_profile && source ~/.bash_profile
  
  test -r ~/.zshrc && source ~/.zshrc
  
  test -r ~/.profile && source ~/.profile
```


### Install Ansible

```sh
brew install ansible
```


### Common Kubernetes collections

```sh
echo "-> Setting up Ansible Galaxy Collections..."

ansible-galaxy collection install community.general
ansible-galaxy collection install community.docker
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install cloud.common
ansible-galaxy collection install kubernetes.core
python3 -m pip install pyyaml
python3 -m pip install kubernetes
python3 -m pip install docker
```