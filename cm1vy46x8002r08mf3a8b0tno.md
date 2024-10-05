---
title: "Automating System Administration Tasks with Bash Scripting"
datePublished: Sat Oct 05 2024 09:22:26 GMT+0000 (Coordinated Universal Time)
cuid: cm1vy46x8002r08mf3a8b0tno
slug: automating-system-administration-tasks-with-bash-scripting
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1728120089318/2ad77c33-6aa9-43de-ab84-37e565ff6ca3.avif

---

In the realm of software development and system administration, efficiency is key. Recently, I embarked on a journey to learn Bash scripting, and it has transformed the way I approach everyday tasks. This blog post outlines how Bash scripting has been useful to me, the challenges I faced, and the tasks I automated to streamline my workflow.

## Why Bash Scripting?

Bash scripting is a powerful tool for automating repetitive tasks, managing system operations, and enhancing productivity. With the ability to execute a series of commands in a terminal, I quickly realized that scripting could save me hours of manual work. By learning Bash scripting, I gained a deeper understanding of the Linux command line and the potential for automation in my daily tasks.

## Key Benefits of Bash Scripting

### 1\. **Automation of Repetitive Tasks**

One of the primary benefits of Bash scripting is automation. Before learning Bash, I often found myself performing repetitive tasks, such as checking disk usage or installing software, manually. This was not only time-consuming but also prone to human error. By creating scripts, I could automate these tasks and run them with a single command.

#### Example: Automating Software Installation

One of the first scripts I wrote was for automating the installation of Prometheus, a monitoring tool I frequently use. Here’s how it helped me:

```plaintext
#!/bin/bash
###############
# Description:
# This script is used to automate the installation of Prometheus

echo "Installing Prometheus"
echo "Checking the binary"
if [ -e /home/ubuntu/prometheus-2.53.2.linux-amd64.tar.gz ]; then
    # -e is used to check whether the file exists or not
    echo "File exists"
    tar -zvxf /home/ubuntu/prometheus-2.53.2.linux-amd64.tar.gz
else
    echo "File does not exist"
    wget https://github.com/prometheus/prometheus/releases/download/v2.53.2/prometheus-2.53.2.linux-amd64.tar.gz
    tar -zvxf /home/ubuntu/prometheus-2.53.2.linux-amd64.tar.gz
fi
```

By running this script, I no longer had to manually check for the binary or download it; the script handled everything seamlessly.

### 2\. **Efficient Resource Management**

Managing system resources effectively is crucial, especially when working on multiple projects. I developed a script to find large files in a specified directory, helping me keep track of disk usage and identify any space hogs.

#### Example: Finding Large Files

This script finds and lists the largest files in a specified directory.

```plaintext
#!/bin/bash

############
# Description:
# This script is used to automate the finding of large files and listing them

echo "This script provides a list of big/large files in the provided path"

path="$1"
# $1 means whatever argument we pass after the shell script can be taken as input

echo $path

du -ah $path | sort -hr | head -n 5 > /tmp/filesize.txt

cat /tmp/filesize.txt
```

This script not only lists the largest files but also saves the output to a temporary file, making it easier for me to manage disk space proactively.

### 3\. **Improved Workflow and Efficiency**

Bash scripts have improved my overall workflow by reducing the time I spend on routine tasks. For instance, I created a script to delete old log files, which has helped me maintain a cleaner system without manual intervention.

#### Example-1: Deleting Old Log Files

```plaintext
#!/bin/bash

####################
# Description: 
# It is used to automate the finding and deletion of folders

for folder in $(find -type d); do
    # "-type d" defines the type as directory
    echo "The folder is $folder"
    if [ -d test ]; then
        # -d checks whether the directory exists or not
        echo "Folder exists"
        echo "Removing the folder"
        rm -rf test
    else
        echo "Folder does not exist"
    fi
done
```

By running this script, I can easily clean up log files older than 30 days, keeping my system clutter-free and ensuring that I don’t run out of disk space unexpectedly.

#### Example-2: Git Installation Check Script

With Git being an essential tool for version control, the **Git Installation Check Script** automates the installation process, ensuring that I always have this vital tool ready for use.

```plaintext
#!/bin/bash

# Description:
# This script is used to automate the installation of git


echo "Checking if git is already installed ?"

if git --version > /dev/null 2>&1; 
# The output of the git version is redireted to /dev/null and discarded to avoid printing to screen
# 2>&1 is used to redirect the standard error and discard it
then
        echo "Git is already installed the version is"
        git --version

else

        echo "Checking the type of Machine"

        if [ "$(uname)" == "Linux" ];
        then
                echo "Installing git on Linux"
                sudo apt install git -y

        elif [ "$(uname)" == "Darwin" ];
        then
                echo "Installing git on MAC"
                brew install git
                
        else
                echo "not a valid command"
        fi
fi
```

## Learning and Skill Development

Learning Bash scripting has not only enhanced my technical skills but also improved my problem-solving abilities. As I wrote scripts to automate tasks, I encountered various challenges that pushed me to learn more about the command line, file management, and system operations.

Through this process, I’ve become more comfortable with Linux, which is essential for my career in software development and cloud computing.

## Conclusion

Bash scripting has been a game-changer for me. It has allowed me to automate repetitive tasks, manage resources efficiently, and improve my workflow. The ability to write scripts has empowered me to take control of my environment, making my development process smoother and more productive.

If you’re considering learning Bash scripting, I highly recommend it. The skills you acquire will be invaluable, not only for automating tasks but also for understanding the underlying mechanics of the systems you work with.

Feel free to reach out if you have any questions or want to share your own experiences with Bash scripting!