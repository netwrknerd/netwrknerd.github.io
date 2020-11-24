---
title: "The correct way to manage Python"
date: 2020-11-24T01:02:00-04:00
categories:
  - blog
tags:
  - network
  - network-automation
---

# The correct way to manage Python

I wrote this guide in the effort of documenting the steps I personally took to build a flexible python environment on MacOS 11 (Big Sur). This is in the pursuit of exploring network automation technologies such as Ansible, but is in no way limited to network engineers. In fact, I wrote this guide after sifting through many articles detailing ways that Python developers set up their own environments. I found the parts that made the most sense to me and have consolidated and re-styled them here for future reference.

MacOS 11 actually comes with Python2.7 and Python3 by default, for use by the system, located in /usr/bin. From here on out we will refer to these as the *system* python environment. We do not want to tamper with this system python environment, so let's get started building a *dev* python environment.

## 1. Xcode
First, we need to install Xcode. Xcode is a basic local development suite provided by Apple, and is available on the App Store. Unless you develop MacOS/IOS applications or want the included IDE, you can get away with installing only the command line tools.

`xcode-select --install`

## 2. Homebrew
Homebrew has been referred to as "The Missing Package Manager of MacOS". I slightly disagree as there is the included App Store, but unfortunately the App Store does not track, install, or update common dev dependencies that are not included in Xcode. This is where Homebrew comes in.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

The command above will install a tool named **brew** which can manage an impressive amount of apps and tools, from simple *nix tools such as **wget** and **tree** to full-blown applications like FireFox. More information on the available brew packages found [here](https://formulae.brew.sh/formula/)

For peace of mind, verify your new homebrew install with:
` brew doctor `

## 3. Installing PyEnv & dependencies
First we need to install some dependencies:
`‌brew install bzip2 openssl readline sqlite3 xz zlib`

> Note: I found the following output during this installation:
> ```
>For compilers to find zlib you may need to set:
>  export LDFLAGS="-L/usr/local/opt/zlib/lib"
>  export CPPFLAGS="-I/usr/local/opt/zlib/include"
>
>For pkg-config to find zlib you may need to set:
>  export PKG_CONFIG_PATH="/usr/local/opt/zlib/lib/pkgconfig"
>```
> I was not able to successfully install python versions via pyenv without setting the attributes listed above. YMMV.

Next we install pyenv: `brew install pyenv`

With pyenv installed, we can begin to install all of the various versions of python that our projects require. If you're just starting out, you can start with the latest version. To find the list of available packages, use: `pyenv install --list`

In my case, I decided to install versions 3.8.6 and 3.9.0 using the following commands:
* `pyenv install 3.8.6`
* `pyenv install 3.9.0`

You can verify the versions installed via pyenv with: `pyenv versions`

The asterisk to the left of the version signifies which version is currently the default.

```
~ % pyenv versions
* system (set by /Users/cj/.pyenv/version)
  3.8.6
  3.9.0
```

Keep in mind that best practice is to run `pyenv rehash` after each individual version installation. More information in this can be found here: 

## 4. Configure shell profile
In order to begin using the various versions of python we've installed with pyenv, we must add the following paths to our shell profile:

```
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zprofile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zprofile
$ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zprofile
```

The config above is specific to *zsh*, the default shell of MacOS. For bash users, you can swap out *.zprofile* for *.bashprofile*.

In order for these changes to effect you need to restart your shell. This can be accomplished by either opening a new terminal or using `exec $SHELL`

## 5. Using PyEnv

With our paths set correctly in our shell profile, we need to define how we want to use the versions of python we've installed with pyenv. There are three levels of priority, with the last being the highest priority when evaluated:

1. Global — sets the Python version(s) for all shells and directories where no version is otherwise set
2. Local — sets the Python version(s) in a directory and its subdirectories (if a local version is not set in the subdirectories)
3. Shell — sets the Python version(s) for the current shell using an environment variable (PYENV_VERSION)

For example, in order to set the latest version of python as my global default, I would do the following:

```
~ % pyenv global 3.9.0
~ % python --version  
Python 3.9.0
```

Also, here's an example of setting the python version of a specific git repository folder:

```
~/documents/git/netwrknerd/fantastic-couscous % pyenv local 3.8.6
~/documents/git/netwrknerd/fantastic-couscous % python --version
Python 3.8.6
```

The version of python is tracked by the creation of a file named ".python-version" as you can see here:

```
~/documents/git/netwrknerd/fantastic-couscous % ls -al
total 96
drwxr-xr-x@ 17 cj  staff   544 Nov 21 18:56 .
drwxr-xr-x@  7 cj  staff   224 Nov 21 18:56 ..
drwxr-xr-x@ 13 cj  staff   416 Nov 21 16:40 .git
-rw-r--r--@  1 cj  staff  1827 Nov 19 18:43 .gitignore
-rw-r--r--   1 cj  staff     6 Nov 21 18:56 .python-version
-rw-r--r--@  1 cj  staff  1074 Nov 19 18:32 LICENSE
-rw-r--r--@  1 cj  staff    85 Nov 19 18:32 README.md
drwxr-xr-x@  6 cj  staff   192 Nov 20 03:19 group_vars
drwxr-xr-x@  3 cj  staff    96 Nov 20 03:19 inventory
~/documents/git/netwrknerd/fantastic-couscous % cat .python-version 
3.8.6
```

Keep in mind that you can also import and call any of the installed version of python by manually specifying their version names, as seen in the following output:

```
~/documents/git/netwrknerd/fantastic-couscous % pyenv local 3.8.6 3.9.0
~/documents/git/netwrknerd/fantastic-couscous % python3.9 --version  
Python 3.9.0
~/documents/git/netwrknerd/fantastic-couscous % python3.8 --version 
Python 3.8.6
```