I wrote this guide in the effort of documenting the steps I personally took to build a flexible python environment on MacOS 11 (Big Sur). This is in the pursuit of exploring network automation technologies such as Ansible, but is in no way limited to network engineers. In fact, I wrote this guide after sifting through many articles detailing ways that Python developers set up their own environments. I found the parts that made the most sense to me and have consolidated and re-styled them here for future reference.

MacOS 11 actually comes with Python2.7 and Python3 by default, for use by the system, located in /usr/bin. From here on out we will refer to these as the *system* python environment. We do not want to tamper with this system python environment, so let's get started building a *dev* python environment.

## 1. Xcode
First, we need to install Xcode. Xcode is a basic local development suite provided by Apple, and is available on the App Store. Unless you develop MacOS/IOS applications or want the included IDE, you can get away with installing only the command line tools like so:

```
xcode-select --install
```

## 2. Homebrew
Homebrew has been referred to as "The Missing Package Manager of MacOS". I slightly disagree as there is the included App Store, but unfortunately the App Store does not track, install, or update common dev dependencies that are not included in Xcode. This is where Homebrew comes in.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

The command above will install a tool named **brew** which can manage an impressive amount of apps and tools, from simple *nix tools such as **wget** and **tree** to full-blown applications like FireFox. More information on the available brew packages found [here](https://formulae.brew.sh/formula/)

For peace of mind, verify your new homebrew install with:
```
brew doctor
```

## 3. Installing PyEnv & dependencies
PyEnv will allow us to install multiple versions of python and manage a "shim" in our $PATH to easily swap between them.

First we need to install some dependencies:
```
‌brew install bzip2 openssl readline sqlite3 xz zlib
```

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

Next we install pyenv: 

```
brew install pyenv
```

With pyenv installed, we can begin to install all of the various versions of python that our projects require. If you're just starting out, you can start with the latest version. 

To find the list of available packages, use: 

```
pyenv install --list
```

In my case, I decided to install versions 3.8.6 and 3.9.0 using the following commands:
* `pyenv install 3.8.6`
* `pyenv install 3.9.0`

You can verify the versions installed via pyenv with: 

```
pyenv versions
```

The asterisk to the left of the version signifies which version is currently the default.

```
~ % pyenv versions
* system (set by /Users/cj/.pyenv/version)
  3.8.6
  3.9.0
```

> Keep in mind that best practice is to run `pyenv rehash` after each individual version installation. 

## 4. Shell profile
In order to begin using the various versions of python we've installed with pyenv, we must add the following paths to our shell profile:

```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zprofile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zprofile
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zprofile
```

The config above is specific to *zsh*, the default shell of MacOS. For bash users, you can swap out *.zprofile* for *.bashprofile*.

In order for these changes to effect you need to restart your shell.

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

Keep in mind that you can also import and call any of the installed versions of python by manually specifying their version names, as seen in the following output:

```
~/documents/git/netwrknerd/fantastic-couscous % pyenv local 3.8.6 3.9.0
~/documents/git/netwrknerd/fantastic-couscous % python3.9 --version  
Python 3.9.0
~/documents/git/netwrknerd/fantastic-couscous % python3.8 --version 
Python 3.8.6
```

## venv (virtualenv)

While pyenv will maintain separate dependencies per version of python it installs, it does not separate dependencies per project. Using **venv** we will be able to create a *virtual python environment* per application+version pair. 

Let's begin by installing the pyenv-virtualenv extension:

```
brew install pyenv-virtualenv
```

Next we create a virtual environment for Ansible using python version 3.8.6 and activate it for this shell session:

```
~ % pyenv global 3.8.6
~ % python --version 
Python 3.8.6
~ % pyenv versions
  system
* 3.8.6 (set by /Users/cj/.pyenv/version)
  3.9.0
  3.9.0/envs/ansible-3.9.0
  ansible-3.9.0
~ % pyenv virtualenv Ansible-3.8.6
Looking in links: /var/folders/bv/wzm6mqwx2wnbq0cl7yxrcm3m0000gn/T/tmp7vbn0965
Requirement already satisfied: setuptools in /Users/cj/.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (49.2.1)
Requirement already satisfied: pip in /Users/cj/.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (20.2.1)
~ % pyenv activate Ansible-3.8.6
(Ansible-3.8.6) ~ % 
```

> Please be aware that I had previously created 3.9.0 prior to capturing the output for this guide.

From the output above you can see we:
* Set the global python version to 3.8.6
* Verified the current python version in our path
* Created a virtual python environment named ansible-3.8.6
* Activated *ansible-3.8.6* in our current shell

Now that we've created a venv, it's best to upgrade pip before installing any dependencies:

```
(Ansible-3.8.6) ~ % python -m pip install pip --upgrade
Collecting pip
  Using cached pip-20.2.4-py2.py3-none-any.whl (1.5 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 20.2.1
    Uninstalling pip-20.2.1:
      Successfully uninstalled pip-20.2.1
Successfully installed pip-20.2.4
```


## Ansible
> Note: Since MacOS 11 I have had to add the following to my path 
> (otherwise building the cryptography wheel fails), but you can
> simply > copy and paste this into your current shell:
> ```
> export LDFLAGS="-L/usr/local/opt/openssl/lib"
> export CPPFLAGS="-I/usr/local/opt/openssl/include"
> export PKG_CONFIG_PATH="/usr/local/opt/openssl/lib/pkgconfig"
> ```

Finally it is time to install Ansible in our new shiny, isolated, virtual environment:

```
(Ansible-3.8.6) ~ % python -m pip install ansible                               
Processing ./Library/Caches/pip/wheels/d4/aa/67/ab673e276d0f02bb35ac71ba7491b8456d56b3c7fe4322e7cb/ansible-2.10.3-py3-none-any.whl
Processing ./Library/Caches/pip/wheels/cb/f5/61/a226fbdb73687e5e8988c7485cc004e557e6b881565060ba62/ansible_base-2.10.3-py3-none-any.whl
Collecting cryptography
  Using cached cryptography-3.2.1.tar.gz (540 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
    Preparing wheel metadata ... done
Processing ./Library/Caches/pip/wheels/13/90/db/290ab3a34f2ef0b5a0f89235dc2d40fea83e77de84ed2dc05c/PyYAML-5.3.1-cp38-cp38-macosx_11_0_x86_64.whl
Collecting jinja2
  Using cached Jinja2-2.11.2-py2.py3-none-any.whl (125 kB)
Collecting packaging
  Using cached packaging-20.4-py2.py3-none-any.whl (37 kB)
Processing ./Library/Caches/pip/wheels/74/c4/b2/301f50054a0b5635fc25567992701a66ccb924f38a85e1133c/cffi-1.14.4-cp38-cp38-macosx_11_0_x86_64.whl
Collecting six>=1.4.1
  Using cached six-1.15.0-py2.py3-none-any.whl (10 kB)
Processing ./Library/Caches/pip/wheels/0c/61/d6/4db4f4c28254856e82305fdb1f752ed7f8482e54c384d8cb0e/MarkupSafe-1.1.1-cp38-cp38-macosx_11_0_x86_64.whl
Collecting pyparsing>=2.0.2
  Using cached pyparsing-2.4.7-py2.py3-none-any.whl (67 kB)
Collecting pycparser
  Using cached pycparser-2.20-py2.py3-none-any.whl (112 kB)
Building wheels for collected packages: cryptography
  Building wheel for cryptography (PEP 517) ... done
  Created wheel for cryptography: filename=cryptography-3.2.1-cp38-cp38-macosx_11_0_x86_64.whl size=411671 sha256=90200e47180a1799889772e5d852e44c5f8575c1d5e535e04e58e684adb6e7b9
  Stored in directory: /Users/cj/Library/Caches/pip/wheels/b6/b8/df/c8777d287c13b23b3caaca1491e1ae1fe9a050d664e1d5a203
Successfully built cryptography
Installing collected packages: pycparser, cffi, six, cryptography, PyYAML, MarkupSafe, jinja2, pyparsing, packaging, ansible-base, ansible
Successfully installed MarkupSafe-1.1.1 PyYAML-5.3.1 ansible-2.10.3 ansible-base-2.10.3 cffi-1.14.4 cryptography-3.2.1 jinja2-2.11.2 packaging-20.4 pycparser-2.20 pyparsing-2.4.7 six-1.15.0
```

Now let's install Juniper's PyEz (required for juniper-junos Ansible role)

```
(Ansible-3.8.6) ~ % python -m pip install junos-eznc
Collecting junos-eznc
  Using cached junos_eznc-2.5.4-py2.py3-none-any.whl (195 kB)
Requirement already satisfied: six in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from junos-eznc) (1.15.0)
Collecting netaddr
  Using cached netaddr-0.8.0-py2.py3-none-any.whl (1.9 MB)
Requirement already satisfied: pyparsing in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from junos-eznc) (2.4.7)
Requirement already satisfied: PyYAML>=5.1 in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from junos-eznc) (5.3.1)
Collecting ncclient>=0.6.3
  Using cached ncclient-0.6.9.tar.gz (118 kB)
Collecting scp>=0.7.0
  Using cached scp-0.13.3-py2.py3-none-any.whl (8.2 kB)
Collecting yamlordereddictloader
  Using cached yamlordereddictloader-0.4.0.tar.gz (3.3 kB)
Requirement already satisfied: jinja2>=2.7.1 in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from junos-eznc) (2.11.2)
Collecting lxml>=3.2.4
  Using cached lxml-4.6.1.tar.gz (3.2 MB)
Collecting pyserial
  Using cached pyserial-3.5-py2.py3-none-any.whl (90 kB)
Collecting transitions
  Using cached transitions-0.8.5-py2.py3-none-any.whl (78 kB)
Collecting paramiko>=1.15.2
  Using cached paramiko-2.7.2-py2.py3-none-any.whl (206 kB)
Requirement already satisfied: setuptools>0.6 in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from ncclient>=0.6.3->junos-eznc) (49.2.1)
Requirement already satisfied: MarkupSafe>=0.23 in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from jinja2>=2.7.1->junos-eznc) (1.1.1)
Collecting pynacl>=1.0.1
  Using cached PyNaCl-1.4.0.tar.gz (3.4 MB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
    Preparing wheel metadata ... done
Collecting bcrypt>=3.1.3
  Using cached bcrypt-3.2.0.tar.gz (42 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
    Preparing wheel metadata ... done
Requirement already satisfied: cryptography>=2.5 in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from paramiko>=1.15.2->junos-eznc) (3.2.1)
Requirement already satisfied: cffi>=1.4.1 in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from pynacl>=1.0.1->paramiko>=1.15.2->junos-eznc) (1.14.4)
Requirement already satisfied: pycparser in ./.pyenv/versions/3.8.6/envs/Ansible-3.8.6/lib/python3.8/site-packages (from cffi>=1.4.1->pynacl>=1.0.1->paramiko>=1.15.2->junos-eznc) (2.20)
Building wheels for collected packages: ncclient, yamlordereddictloader, lxml, pynacl, bcrypt
  Building wheel for ncclient (setup.py) ... done
  Created wheel for ncclient: filename=ncclient-0.6.9-py2.py3-none-any.whl size=78200 sha256=22fc397bcc5e3003afa3dd2847dee81dae52d0af696f2214a63cd920bffd1b4b
  Stored in directory: /Users/cj/Library/Caches/pip/wheels/4e/07/51/62032d2dfa8b15def5d7b5f41f355b1c4dec833c1f50d558eb
  Building wheel for yamlordereddictloader (setup.py) ... done
  Created wheel for yamlordereddictloader: filename=yamlordereddictloader-0.4.0-py3-none-any.whl size=4051 sha256=c34830be88baeeccf4d258cc27ef191bd68800c9b40d1d55aa081b5bd037ca20
  Stored in directory: /Users/cj/Library/Caches/pip/wheels/50/9a/6f/9cb3312fd9cd01ea93c3fdc1dbee95f5fa0133125d4c7cb09a
  Building wheel for lxml (setup.py) ... done
  Created wheel for lxml: filename=lxml-4.6.1-cp38-cp38-macosx_11_0_x86_64.whl size=1660477 sha256=0c980ad202a3e2cbc0fd213c3a53fdbcd6963cc7ad8541cd9b8f7ea5c34e17b1
  Stored in directory: /Users/cj/Library/Caches/pip/wheels/ec/09/f7/3fd7a70d4bf66890898e4e7fc71ecd5107017554cac3b604e9
  Building wheel for pynacl (PEP 517) ... done
  Created wheel for pynacl: filename=PyNaCl-1.4.0-cp38-cp38-macosx_11_0_x86_64.whl size=429584 sha256=41ecbe66458586a0ece4ba37b95e60ec8b5dfd47a75128c15879c7e1b358c916
  Stored in directory: /Users/cj/Library/Caches/pip/wheels/10/23/c0/f42041fbad88c47b268b6119919130b407b48e24b41a349684
  Building wheel for bcrypt (PEP 517) ... done
  Created wheel for bcrypt: filename=bcrypt-3.2.0-cp38-cp38-macosx_11_0_x86_64.whl size=32335 sha256=e981a1389aeb4e0017096193a0e9d0171d9c46724511b3cf7b0ff6ccee73d41d
  Stored in directory: /Users/cj/Library/Caches/pip/wheels/af/42/cb/78425eb7d565a75b710a82f213c19f7100b873af40ddb372fc
Successfully built ncclient yamlordereddictloader lxml pynacl bcrypt
Installing collected packages: netaddr, pynacl, bcrypt, paramiko, lxml, ncclient, scp, yamlordereddictloader, pyserial, transitions, junos-eznc
Successfully installed bcrypt-3.2.0 junos-eznc-2.5.4 lxml-4.6.1 ncclient-0.6.9 netaddr-0.8.0 paramiko-2.7.2 pynacl-1.4.0 pyserial-3.5 scp-0.13.3 transitions-0.8.5 yamlordereddictloader-0.4.0
```

> At this point you'll most likely want to re-init your shell for peace of mind. Don't forget to activate your venv *ansible-3.8.6*!

Now let's install the *'juniper-junos* Ansible role using ansible-galaxy:

```
(Ansible-3.8.6) ~ % ansible-galaxy install juniper.junos
```

> Please note that galaxy roles are installed at the root of your user directory and are *not* isolated per virtual python environment. These roles will be shared between all of your Ansible projects.

Last but definitely not least, you will need to add the following as a variable to either your playbooks or as a group_var:

`ansible_python_interpreter: ~/.pyenv/shims/python`

# Summary
That's it! You have successfully installed python versions *3.8.6* and *3.9.0* in isolated, virtual environments free to be mucked up as you develop away. You can be rest assured that whatever you do while testing ansible-playbooks and installing other python modules, you will not break your system's python or create a mess that can't cleaned up easily. Hope this helped!
