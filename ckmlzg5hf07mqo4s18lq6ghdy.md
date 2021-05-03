## Multiple Git Accounts on One Computer

You might need to use multiple Git accounts on a single computer. For example, you might have personal, work, and university accounts.

Most Git tutorials show you how to configure your global username and email which will be used when you make commits. Typically you set this right after installing Git and is your default user name and email.

```
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR EMAIL"
```

## Setting User per Git Repository
You can set a custom user name and email for a specific Git repository by running this in the root of your Git repository:
```
git config user.name "YOUR NAME"
git config user.email "YOUR EMAIL"
``` 

## Setting User per Git Commit
In some cases, you might want to set a user name for a specific commit. You can configure a user name and email as part of the git commit command.
```
git -c "user.name=YOUR NAME" -c "user.email=YOUR EMAIL" commit -m"COMMIT MESSAGE"
```

## Setting User per Folder
I found [an interesting solution on Stack Overflow](https://stackoverflow.com/a/43884702/5679427) for setting custom user names and emails based on which folder your Git repository is in on your computer.

You need to update the global `~/.gitconfig` file. This file is specific to each user and is also the file where your global user and email are stored. Typically, on a Windows machine, this file is located at the root of your user folder (e.g. `C:\Users\CristianaMan\.gitconfig`). If you are unsure where this file is, you can run:

```
git config --list --show-origin
```
This command will show a list of all the configured Git properties and the corresponding file location. 

Once you identified the file, notice it contains the following:
```
[user]
	name = YOUR NAME
	email = YOUR EMAIL
```

You can replace the user configuration with a snippet like the one in the example below which uses [conditional includes](https://git-scm.com/docs/git-config#_includes). In this example, we would clone all personal repositories under a folder called `personal` and all work repositories under a folder called `work`.

In `.gitconfig` file:
```
[includeIf "gitdir:~/personal/"]
	path = .gitconfig-personal
[includeIf "gitdir:~/work/"]
	path = .gitconfig-work
```

> Note: On Windows you might need to add the whole path 
> (e.g. `[includeIf "gitdir:C:/Users/CristianaMan/Documents/personal"]`)

Create a `.gitconfig-personal` file in the same folder as your `.gitconfig` file and add
```
[user]
	name = YOUR NAME
	email = YOUR PERSONAL EMAIL
```

Create a `.gitconfig-work` file and add
```
[user]
	name = YOUR NAME
	email = YOUR WORK EMAIL
```

You can check if your user name and email are set correctly by cloning a project in the personal folder and in the root of that repository running `git config --list --show-origin`.
