+++
description = "Learn how to sign your commits with Keybase.io PGP key without typing a passphrase on each commit and get your commits verified on Github"
tags = ["git", "pgp", "github", "security", "linux"]
categories = [
  "Sys Admin",
  "Linux"
]
date = "2016-09-18T22:29:45Z"
title = "Setting up GPG, GIT with keybase.io"
social_image = "https://s-media-cache-ak0.pinimg.com/originals/93/b7/0f/93b70f721d3e341e93ddea2853815df3.png"
+++

Back in April, {{<ext-link "Github started to support signed commits and tags with PGP" "https://github.com/blog/2144-gpg-signature-verification">}}.

I've always wanted that {{<ext-link "PGP (Pretty Good Privacy)" "https://en.wikipedia.org/wiki/Pretty_Good_Privacy">}} were widely spread because, for example I would love to easily send confidential emails and avoid repudiation in some others that I receive from third parties, however the lack of support for email clients not just desktop client, web clients and mobile clients which today are that necessary makes very hard that people adopt it, so using it in our daily life is mostly an utopia.

After Github has added support for it, it has been for me a good excuse to start to use PGP in my daily life, it isn't email, but well, I use Github, as many other geeks, almost in daily basis, or if it isn't Github, it's git, so if it's well integrated then, I don't mind to use in any repo either although it isn't hosted on Github.

The key point of PGP is, obviously, to have a pair of keys; I had one long time ago, that I ended without using it, after I don't know how many years later, which was around 2 years ago, I got a new PGP key pair with {{<ext-link "keybase" "https://keybase.io">}}, but being hones the account was almost unactive until last April.

Keybase.io provides a PGP cloud service which create and host keys attached to people; it provides some mechanisms to identify people through third party social services which today are used by tons of people. Of course, the concept of hosting your private key obligate to the users in trust in their security, something that's very debatable, but being honest, that same thing happens with the digital certificates and in my case, for the usage that I give to the key, it isn't big concern.

To setup my Keybase key with Github, in order to sign my commits and be marked as verified, I followed the blog awesome {{<ext-link "blog post Github GPG + Keybase PGP of Ahad Nassri" "https://www.ahmadnassri.com/blog/github-gpg-keybase-pgp/">}}


## Using gnupg-agent for the sake of your fingers

A very important thing for me and avoid to disable the signing options of my git configuration is no to have to type the passphrase of my GPG key on each commit. For this purpose, we have an agent, as we have for the `ssh`.

In my case, I use an Ubuntu Linux for all my development stuff, so here I describe to install and configure `gpg-agent` for a Ubuntu Linux, if you use another distro then you may have to make some changes and if you use another OS then, you probably end up changing everything but I hope that you can get, at least, the idea in how to do so.

### Install it

Installing the agent is as easy how a simple `apt-get install gnupg-agent` command execution.

You can run the agent manually each time that you login or do the same adding that command to your `.bashrc` file or the equivalent `rc` file that your shell execute on a new session.

In my case I use {{<ext-link "Z shell (zsh)" "https://en.wikipedia.org/wiki/Z_shell">}} and {{<ext-link "Oh My ZSH!" "http://ohmyz.sh/">}} and I use the {{<ext-link "plugin that it provides for running the `gnupg-agent`" "https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/plugins/gpg-agent/gpg-agent.plugin.zsh">}}, which you can run adding to the `plugins` variable list `gpg-agent`, for example mine, looks like

```sh
plugins=(ssh-agent zsh-syntax-highlighting gpg-agent)
```

**if you're using gnpg-agent version 2** then you don't need to modify your `rc` file because you don't need to execute the agent, it will be executed automatically at OS the startup time as any other service; check it with `gpg-agent --version`


### Configure it

To configure the agent you have to create a configuration file; by default is located on your *user home director* in the directory `.gnupg/gpg-agent.conf`, take a look to the {{<ext-link "options summary section of the documentation web site" "https://www.gnupg.org/documentation/manuals/gnupg/Agent-Options.html#option%20%2d%2doptions">}} to see all the available options.

Mine only contains a few options which I wanted to tweak

```
# Set the default cache time to 1 day.
default-cache-ttl       86400
default-cache-ttl-ssh   86400

# Set the max cache time to 30 days.
max-cache-ttl           2592000
max-cache-ttl-ssh       2592000
```

Now it's time to add our *pgp keys*, due the purpose of this post, I'll add my Keybase Key, but you could whatever *pgp key* you own, repeating the process for each key.

Export your private key through the Keybase command line or through the website and save it somewhere, I'll assume that the key file is in `~/my-key` for the rest of this post.

List the current keys, which we shouldn't have any, but the same command will initialize the key store database on the first call `gpg --list-keys`, you should see something like

```
~ ➜  gpg --list-keys
gpg: /home/vagrant/.gnupg/trustdb.gpg: trustdb created
```

Import the key `gpg --import ~/my-key`, in my case I got

```
~ ➜  gpg --import my-key
gpg: key B5CB5AD5: secret key imported
gpg: key B5CB5AD5: public key "keybase.io/ifraixedes <ifraixedes@keybase.io>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

And now if you list the keys (`gpg --list-keys`), you should see something like

```
~ ➜  gpg --list-keys
/home/vagrant/.gnupg/pubring.gpg
--------------------------------
pub   4096R/B5CB5AD5 2015-04-15
uid                  keybase.io/ifraixedes <ifraixedes@keybase.io>
sub   2048R/97F96DB7 2015-04-15 [expires: 2023-04-13]
sub   2048R/1C9B754A 2015-04-15 [expires: 2023-04-13]
```

## Configure GIT for signing and use a default key

In my case, I'm happy to sign all the commits with the same key, so I don't have the need to constantly specify a key per project so I made all this configurations as global, if you need to configure them per project then remove the `--global` options of the commands that I listed below.

Let's specify the key to use with `git config --global user.signingkey ifraixedes@keybase.io` or use the key shorthand `git config --global user.signingkey B5CB5AD5`

Then, let's sign all the commits by default, so we don't have to specify `-S` to `git commit` each time, `git config --global commit.gpgsign true`

With these two options, we are set if the agent is version 1, but **if the agent that you're running in your machine is the version 2**, then you have to add one configuration parameter more to git, otherwise you'll see a message, on each commit, saying something like `agent is not running in this session`; this happens because GIT is using by the default the `gpg` binary in your path, and for the version 2 you have to use the `gpg2`.

First make sure that you install the *gpg v2* with `sudo apt-get install gnupg2` and thereafter, tell git to use *gpg2* with `git config --global gpg.program gpg2`

## Wrapping up

At this point, the best thing before giving it a try is to exit your session and login again, than trying to run the `gpg-agent` by hand, because you'll try that everything is running as expected and it's easier.


Happy signing!

---
Useful links:

* {{<ext-link "Github GPG + Keybase PGP" "https://www.ahmadnassri.com/blog/github-gpg-keybase-pgp/">}}
* {{<ext-link "Github help for PGP" "https://help.github.com/categories/gpg/">}}
* {{<ext-link "Good summary docs of GnuPG" "https://wiki.archlinux.org/index.php/GnuPG">}}
