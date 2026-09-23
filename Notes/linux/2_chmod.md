The chmod (Change Mode) command is used in Linux to change the access permissions of files and directories. It determines who can read, write, or execute a file.
In Linux, permissions are split into three groups:

* u (User / Owner)
* g (Group)
* o (Others / Everyone else)

And there are three permission types:

* r (Read - view file or list directory contents)
* w (Write - modify or delete file)
* x (Execute - run a script/program or enter a directory)

------------------------------
## Method 1: Using Numbers (Absolute Mode)
This is the fastest and most common way to set permissions. Each permission type has a number score:

* 4 = Read (r)
* 2 = Write (w)
* 1 = Execute (x)
* 0 = No permission

You add the numbers together to get a 3-digit code for User, Group, and Others:

| Command | Numeric Code | What it does |
|---|---|---|
| chmod 777 file.txt | 7 (4+2+1), 7, 7 | Full permissions to everyone (Highly unsecure!) |
| chmod 755 script.sh | 7 (Owner), 5 (Group), 5 (Others) | Owner can do everything; others can only read and execute (Standard for scripts). |
| chmod 644 document.txt | 6 (4+2), 4, 4 | Owner can read and write; others can only read (Standard for regular files). |
| chmod 600 id_rsa | 6 (Owner), 0, 0 | Only the owner can read and write; blocked for everyone else (Standard for private SSH keys). |

------------------------------
## Method 2: Using Letters (Symbolic Mode)
This method is great for adding or removing a specific permission without changing the rest. You use + (add), - (remove), or = (set exactly).

* Make a script executable:
chmod +x script.sh (Gives execution permission to everyone)
* Remove write permission for "others":
chmod o-w report.pdf
* Give group read and write permission:
chmod g+rw notes.txt

## Pro Tip: Apply to folders recursively
If you want to change permissions for a folder and everything inside it, use the -R flag:
chmod -R 755 /path/to/directory
What file or directory are you trying to restrict or open up right now? Tell me who needs access, and I can give you the exact chmod command to use.

The order is strictly fixed and always follows this sequence from left to right:First Digit: User (The owner of the file)Second Digit: Group (Members of the file's assigned group)Third Digit: Others (Everyone else on the system)An easy way to remember the order is the acronym UGO (User, Group, Others).So when you run chmod 400:4 is for the User (Read only)0 is for the Group (No access)0 is for Others (No access)Does the UGO order make sense, or would you like to see how to check who the current owner and group actually are using the ls command?