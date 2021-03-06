Configure git
=============
Depending on transport (`ssh` or `https`) you prefere to access github, configure your local git settings based on guide bellow.

## SSH vs HTTPS:
* Please note that the port 22 (`ssh` default) is quite often blocked on corporate networks - this might be a problem while connected to such network on business trip/visits. This is disadvantage for `ssh` when compared to the `https` transport, since `https` is running by default on 443 port which is blocked very rarely.

* Another advantage of `https` is that it uses github `tokens` for login credentials, which allows to set specific oauth scopes (access rights) per token if it is desired - please see the `SSH transport` section bellow for details.

## SSH transport
1. SSH keys setup - please follow [this github guide](https://help.github.com/articles/connecting-to-github-with-ssh/) to setup ssh key(s) for github.

2. Setup moniker for github url:
    
    **Either** run the following command:
    
    ```sh
    git config --global url."git@github.com:".insteadOf github:
    ```
    
    , **or** add the following section in to your `~/.gitconfig` file:
    
    ```ini
    [url "git@github.com:"]
        insteadOf = github:
    ```

## HTTPS transport
1. Github **token** generation/setup:
    
    The github token is equivalent to ssh key, but it allows to set [oauth scopes (access rights)](https://developer.github.com/apps/building-oauth-apps/scopes-for-oauth-apps/), and it also can be safelly stored in keychain (not as plain-text file). 

    Login to the https://github.com and then go to the https://github.com/settings/tokens page where you can generate new token(s). You can assign set of oauth scopes to generated token (enable all or just subset).

    **Copy-Paste and store generated token somewhere!**, you will need it later (in point 3. bellow). It is not possible to acquire the token value afterwords (once generation web page is refreshed) - new token needs to be generated when it is lost.

2. Setup moniker for github url:
    
    **Either** run the following command:
    ```sh
    git config --global url."https://github.com/".insteadOf github:
    ```
    , **or** add the followng section in to your `~/.gitconfig` file:
    ```ini
    [url "https://github.com/"]
        insteadOf = github:
    ```

3. Setup of the `credential helper` for your platform (**Non-mandatory** step, but provides **convenience**):

    The `credential helper` is keychain storage which eliminates necessity to provide login credentials for every single git server request (e.g. pull, fetch, push, clone, submodule update/init/sync). It is necessary to be provided just once, then it will be stored in keychain and used from there automatically when necessary.

    Please follow [this guide](https://stackoverflow.com/questions/5343068/is-there-a-way-to-skip-password-typing-when-using-https-on-github#5343146) to setup `credential helper` for your platform.
            
    * **Mac OS X** platform: Run the following command to set `credential helper` for this platform:
        ```sh
        git config --global credential.helper osxkeychain
        ```
        Then execute for example `git pull` command, which will ask for username and password, use your gihub username and copy-pasted token frim step 1. above as password. You won't be asked for username or password anymore once this is done the 1st time.
 
## Suggested additional git configuration/aliases<a name="suggested_additional_git_config"/>
We would like to share a few things how to improve your git setup. It should simplify your git workflow, and also make it more reliable (specially important if submodules are used in the repo).

Execute following sequence of commands to add new aliases to your user level git config:

```sh
git config --global alias.pullall '!f(){ git pull "$@" && git syncUpdate; }; f'

git config --global alias.syncUpdate '!f(){ git submodule sync --recursive && git submodule update --init --recursive; }; f'

git config --global alias.logg '!f(){ git log --graph --stat --decorate=full --color --word-diff=color -M -C --find-copies-harder -l100 --histogram "$@"; }; f'

git config --global alias.oldest-ancestor $'!zsh -c \'diff --old-line-format= --new-line-format= <(git rev-list --first-parent \"${1:-master}\") <(git rev-list --first-parent \"${2:-HEAD}\") | head -1\' -'
```

This creates 4 aliases for git (`git pullall`, `git syncUpdate`, `git logg`and `git oldest-ancestor`) in your `~/.gitconfig` file, where:

 * `git pullall <git-pull-params>`: this executes git pull, and then pulls all submodules (with recurve sync, init and update for all submodules). Actually, you should prefer to execute this alias  instead of trivial `git pull` in 99.999…% of cases.

 * `git syncUpdate`: this pulls all submodules (with recurve sync, init and update for all submodules),

 * `git logg <git-log-params>`: this executes git log with additiona tweaks which give you very nice version tree on commandline. The output includes all info about all branches, tags, and commit details per each commit (this is what you usually need),

 * `git oldest-ancestor <commit1> <commit2>`: this finds oldest common ancestor for two commits (it is very useful for finding very original commit which some branch has been branched off some other branch).

