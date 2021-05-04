## Creating an Empty Commit

There are times when creating an empty commit in Git is useful. A common scenario is the need to trigger a continuous integration pipeline or adding an important commit message that might have some effect. For example, SemVer integrates with Git and uses commit messages to increase version numbers of your library or application semantically. 

This short post shows you how you can create an empty commit.

## Attempt to commit without any file changes

I needed to create an empty commit when using SemVer. Let me explain.
SemVer finds commit messages containing patterns like `+semver:minor` and determines the corresponding build and release number used in the continuous deployment process. While the best practice is adding this text on the commit with the minor or major change, there are cases where the change is broken up into multiple smaller commits. I personally experienced such a scenario or forgot to add the correct message bumping the version.

So I tried running this:
```
git commit -m "+semver:minor"
```

Git, however, did not let me commit, proudly declaring my branch was up to date and there was nothing to commit.

## Using the `--allow-empty` option
Git assumes that creating an empty commit might be a mistake, thus preventing it by default. You can bypass this using the [`--allow-empty`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---allow-empty) option.

```
git commit -m "+semver:minor" --allow-empty
```
With this approach, you can create an empty commit containing no files or any changes, only with your desired message.

## Using the `--allow-empty-message` option
When researching to write this post, I stumbled upon the [`--allow-empty-message`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---allow-empty-message) option. Similarly to the `--allow-empty` option, it is not meant for typical workflow. Omitting to write a descriptive short message about what has changed is not good practice and makes it harder to scan through the changes when looking at the commit history.

  
However, an empty commit with no message might be useful for triggering a Continuous Deployment & Integration (CD/CI) pipeline. 

```
git commit --allow-empty-message --no-edit
```
This snippet bypasses Git's requirement of adding a message. The [`--no-edit`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---no-edit) option keeps the selected empty message and prevents launching an editor, which by default prompts modifying empty commit messages.

## Summary
In rare cases, you might need to bypass Git's default requirements when making a commit. The `--allow-empty` and `--allow-empty-message` options can be useful to create empty commits to trigger some effect. The examples mentioned in this post were triggering a CI/CD pipeline or increasing the semantic version of your package or application.