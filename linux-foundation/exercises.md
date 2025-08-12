# Module 1: Linux Fundamentals
This is about navigating the system and handling files.
## Exercise 1.1: Filesystem Navigation

* Find your current directory using `pwd`.
* List the contents of your home directory (`ls`). Now list them in a long format showing permissions and ownership 
  (`ls -l`). Now include hidden files (`ls -la`).
* Create a directory structure: `mkdir -p ~/lfca-practice/scripts`
* Navigate into the new scripts directory (`cd ~/lfca-practice/scripts`).
* Navigate back to your home directory using an absolute path, then a relative path (`cd ~`, then `cd ../..`).

## Exercise 1.2: File Manipulation

* Create an empty file called `notes.txt` in `~/lfca-practice` (`touch`).
* Copy `notes.txt` to `notes_backup.txt` (`cp`).
* Move `notes_backup.txt` into the scripts directory (`mv`).
* Create a symbolic link in `~/lfca-practice` that points to `~/lfca-practice/scripts/notes_backup.txt` (`ln -s`).
* Remove the original `notes.txt` file (`rm`).
* Remove the scripts directory and all its contents (`rm -r` or `rm -rf`). Be careful with this command!

## Exercise 1.3: Text Processing

* Use `echo "Hello Linux World" > greeting.txt` to create a file with content.
* View the file's content with `cat`.
* Count the lines, words, and characters in the file using `wc`.
* Search for the word "Linux" in the file using `grep`.
* Use the `find` command to locate all files in your home directory ending with `.txt`.
* Pipe the output of `ls -l /etc` into `less` to view it page by page.
* Pipe the output of `ls -l /etc` into `grep cron` to find all files/directories related to cron.

# Module 2: System Administration
This covers managing software, services, and processes.
## Exercise 2.1: Software Management

* Update your package list:
  * `sudo apt update` (this is for Ubuntu).
* Search for a package called `neofetch`:
  * `apt search neofetch`
* Install it:
  * `sudo apt install neofetch`
* Run `neofetch`.
* Remove it with `sudo apt remove neofetch`.

## Exercise 2.2: Process Management

* List all current running processes with `ps aux`.
* Find the Process ID (PID) of the cron service:
  * `pgrep cron`
* Run a command that will take a while
  1. `sleep 100`
  2. `sleep 100 &`
* **Observe:**
  * What does the terminal indicate?
  * Try `ps -ef --forest`
  * Try `pstree -p`, where `p` expose the process IDs

* Find the PID of `sleep 300 &` sleep command and terminate it using `kill`.

# Module 3: Security
Managing users, groups, and permissions.
## Exercise 3.1 User and Group Management
* Create a new user named `jdoe`: 
  * `sudo useradd -m jdoe`. The `-m` creates a home directory.
* Set a password for this new user (`sudo passwd jdoe`).
* Create a new group called `developers`:
  * `sudo groupadd developers`
* Add the user `jdoe` to the developers group:
  * `sudo usermod -aG developers jdoe`
* Verify the user's groups with `id jdoe`.
* Delete the user `jdoe` and their home directory:
  * `sudo userdel -r jdoe`.
## Exercise 3.2 Permissions
* Create a file `project.data`.
* Use `chmod` to set permissions so that the owner can read/write/execute, the group can read/execute, and others can 
  only read.
  * **Hint:** `chmod 754 project.data`
* Create a directory `project-files`.
* Change the ownership of this directory to the `root` user and the `developers` group:
  * `sudo chown root:developers project-files`

# Module 4: Storage
Managing disks and filesystems. (This may require adding a new virtual disk to your VM).
## Exercise 4.1: Partitions and Filesystems
* List all block devices (`lsblk`).
* Assuming you've added a new virtual disk (e.g., `/dev/sdb`), use `sudo fdisk /dev/sdb` to create a new primary 
  partition.
* Format the new partition with an ext4 filesystem (`sudo mkfs.ext4 /dev/sdb1`).
* Create a directory `/mnt/data` (`sudo mkdir /mnt/data`).
* Mount the new filesystem at `/mnt/data` (`sudo mount /dev/sdb1 /mnt/data`).
* Verify it's mounted with `df -h`.
* Edit `/etc/fstab` to make the mount permanent. You will need to get the UUID of the partition with `blkid`.

# Module 5: Networking
Configuring and troubleshooting network connections.

## Exercise 5.1: Network Inspection

* Find your machine's IP address (`ip a` or `ip addr show`).
* View your machine's routing table (`ip route`).
* Check your DNS resolution settings by viewing `/etc/resolv.conf`.
* Use `ping google.com` to test internet connectivity.
* Use `dig google.com` or host google.com to test DNS lookups.

## Exercise 5.2: Basic Firewall

* On Ubuntu (`ufw`): 
  * Check the firewall status (`sudo ufw status`).
  * Allow incoming SSH connections (`sudo ufw allow ssh`).
  * Enable the firewall (`sudo ufw enable`).




# Module 6: Automation with Scripting (NEW)
This module introduces basic scripting to automate the various tasks.

## Exercise 6.1: Bash Scripting - User Onboarding

**Purpose:**

To combine user management and file manipulation into an automated script.

**Task:**
1. Create a file named `create_user.sh`.
2. Write a Bash script that accepts one argument: a username.
3. The script should first check if the user already exists. If so, it should print an error message and exit.
4. If the user does not exist, it should create the user (`useradd -m`).
5. After creating the user, it should create a `~/welcome.txt` file in the new user's home directory with a message like 
  "Welcome [username]! Your account was created on [date]."
6. Make the script executable:
  
   `chmod +x create_user.sh` and run it with a test username (e.g., `./create_user.sh testuser`).

## Exercise 6.2: Python Scripting - Disk Usage Monitor

**Purpose:**

To introduce Python for system monitoring, which can handle more complex logic than simple shell scripts.

**Task:**

1. Create a file named `check_disk.py`.
2. Write a Python script that checks the disk usage of the root filesystem (`/`).
3. You can achieve this using Python's shutil library (`shutil.disk_usage('/')`).
4. Define a usage threshold (e.g., 85%).
5. If the current disk usage percentage is above the threshold, the script should print a clear warning message like 
   "WARNING: Root disk usage is at [XX%] which is above the 85% threshold."
6. If it's below the threshold, it should print "OK: Disk usage is normal."
7. Run the script using `python3 check_disk.py`.

## Exercise 6.3: Integrating Scripts with the Shell using Functions and Aliases

**Purpose:**

To learn how to make script functions available directly in the terminal, making them feel like native commands.

**Task:**

1. Create a file named `my_helpers.sh`. Inside this file, define a Bash function called `gcu()` (for "git commit and 
   update") that performs three commands: `git add .`, `git commit -m "$1"`, and `git push`. The `$1` will allow a 
   commit message to be passed to the function.
2. Edit your shell's startup file (e.g., `~/.bashrc` or `~/.zshrc`). Add a line at the end to `source` your 
   `my_helpers.sh` script. This loads the function into your shell's memory.
3. Below the source line in your startup file, add an alias: `alias goup="gcu"`.
4. Reload your shell's configuration (by running `source ~/.bashrc` or by opening a new terminal).
5. Now, you should be able to run `goup "My test commit"` directly from your command line as if it were a built-in
   command.

