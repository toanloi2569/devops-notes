# Installation

### Installation
**Sử dụng pipx**  
Cần cài đặt [pipx](https://pipx.pypa.io/stable/installation/) trước, sau đó cài đặt poetry với lệnh sau
```
$ pipx install poetry
```

### Enable tab completion for Bash, Fish, or Zsh
**Oh My Zsh**  
```
$ mkdir $ZSH_CUSTOM/plugins/poetry
$ poetry completions zsh > $ZSH_CUSTOM/plugins/poetry/_poetry
```  

Thêm plugin vào `~/.zshrc`
```
plugins(
    poetry
    ...
    )
```

