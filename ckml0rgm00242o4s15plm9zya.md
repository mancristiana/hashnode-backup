## Moving a Git Repository

Every once in a while you might need to move a Git repository while preserving the history. 

## TL;DR

Here is how you can move or duplicate a Git repository from an initial repository 
`https://github.com/username/old.git` to a new repository `https://github.com/username/new.git`

```
git clone https://github.com/username/old.git
git remote rm origin
git remote add origin https://github.com/username/new.git
git push -u origin main
``` 

## Background

My friend Leila built a website for  [*Fearless into Tech*](https://fearlessintotech.com)  community. She shared the code of the website on her GitHub profile. 

Since more members in the community wished to contribute to the website, we created an organization on GitHub. That way anyone in the community could create issues, write code and improve the website. We then needed to move the repository from Leila's personal GitHub profile to the new organization while preserving the commit history to reflect her contributions. 

Here is how you can move a Git repository:


## 1. 
Clone the repository you want to move by running  

```
git clone https://github.com/username/old.git
``` 

## 2.
Check remote repositories it is connected to by running

```
git remote -v
``` 
Output example showing a remote called origin which points to the original git repository. 

```txt
origin https://github.com/username/old.git (fetch)
origin https://github.com/username/old.git (push)
``` 

## 3.
Remove the remote called origin by running 

```
git remote rm origin
``` 
Check that the list of remotes was empty by running `git remote -v`

## 4.
Add a new remote and call it origin

```
git remote add origin https://github.com/username/new.git
``` 

## 5.
Push to the main branch of the new repository
```
git push -u origin main
```

Let me know if there is an easier way to do this :) 