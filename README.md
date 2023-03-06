# docker-tutorial
This repository is designed to provide a simple and effective way to run Valgrind, C, C++ code. If you're a 42 student, this repo is a must-have resource for you to check for memory leaks in  your C/C++ code and perfecting your skills ;)

To create a Linux virtual environment with Docker on a Mac:

1. Install Docker via the Managed Software Center.
2. Clone the [42toolbox](https://github.com/alexandregv/42toolbox.git) repository and run the init_docker.sh file which is provided in this repo too.
3. Check that Docker is running by checking the Docker icon in the top right corner. Note it may take some time.
4. Create a directory called "docker_image" and a Dockerfile within it.
5. In the Dockerfile, specify the instructions to build the Linux image by using the following:
```bash
# Sets the base image as the latest version of Arch Linux
FROM archlinux:latest    

# Updates the system's package databases and # Installing some packages (feel free to add packages that you need)
RUN pacman -Syy
RUN echo "Y" | pacman -Syu    
RUN echo "Y" | pacman -S gcc git make vim sudo gdb valgrind zsh glibc python3 python-pip unzip wget curl cmake

# Adds a new user "asioud" to the system and set a password for it
RUN useradd -m -G users -s /bin/bash asioud   
RUN echo "asioud:strongpwd" | chpasswd

# Grants administrative privileges to the "asioud" user
RUN echo 'asioud ALL=(ALL:ALL) ALL' | sudo EDITOR='tee -a' visudo   

# Installs Pygments and Norminette for the new user
RUN su - asioud -c "pip install --user pygments norminette"

# Sets the system time zone to Europe/Berlin by copying the time zone information to the /etc/localtime file.
RUN cp /usr/share/zoneinfo/Europe/Berlin /etc/localtime 

# Sets the PATH environment variable to include the $HOME/.local/bin directory. (colour-valgrind doesn't work without it)
RUN echo "export PATH=$PATH:$HOME/.local/bin" >> ~/.zshrc
```

6. In the terminal, navigate to the home directory and run `docker build -t arch:42 docker_image/` to build the Docker image.
7. Running the script to get inside the Linux container:
Details are not provided in the original text.
```bash
#!/bin/bash

# Get the name of the script
me=`basename "$0"`

# Strip the path to the home directory from the current working directory
NEWVAR=${PWD#/Users/}

# Get the absolute path to the script
SCRIPTPATH="$( cd -- "$(dirname "$0")" >/dev/null 2>&1 ; pwd -P )"

# Construct a variable for the home directory in the container
VAR="/home/${NEWVAR}"

# Check if an alias for the script exists in the .zshrc file
if grep -q "alias archlinux=" $HOME/.zshrc; then
    echo ""
else
	# If the alias doesn't exist, add it to the .zshrc file
	echo "alias archlinux=${SCRIPTPATH}/${me}" >> $HOME/.zshrc 
fi

# Run the Docker container
docker run -it \
  --dns 8.8.8.8 --dns 8.8.4.4 \
  --workdir=$VAR \
  -e "DEBUGINFOD_URLS=https://debuginfod.archlinux.org" \
  -e "OSTYPE=archlinux" \
  -e "SHELL=/bin/zsh" \
  -e "TERM=xterm-256color" \
  -e "HOSTNAME=archlinux" \
  --user=asioud \
  --privileged=true \
  --hostname=archlinux \
  -v $HOME:/home/asioud \
  arch:42
```
You are now logged in your docker Linux container and you've mounted the mac's home directory into the arch's linux home directory.
