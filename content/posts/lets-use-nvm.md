---
title: "Let's use NVM"
date: "2015-06-14"
description: "Use NVM (Node Version Manager) to gain better controls of Node JS versions."
categories:
  - "tool"
  - "nodejs"
---

![Node Version Manager](/img/nvm.png)

<br />


My colleague just came to me with a troubled face. He couldn't figure out why his gulp script failed to start up local server where everyone else is able to do so. After a short debugging session, I found that the culprit is Node JS version. Certainly, our gulp scripts have a conflict with the latest Node JS. Now, my colleague has to downgrade his Node JS installation, or does he have to? :)

In this fast-moving industry, maintaining specific versions for your libraries or tools seems to be a chore. That's why there are so many xyz managers, e.g. pip, apt-get, bower, npm, jspm, and homebrew, etc. For managing versions of Node JS, there is NVM - Node Version Manager. I highly recommend using it to have better controls of which version of Node I use.

If you already installed Node via other installers, it's better to completely remove the existing Node and NPM. To do that, I stole the following command from this stackoverflow thread - [How do I completely uninstall Node.js, and reinstall from beginning (Mac OS X)](http://stackoverflow.com/questions/11177954/how-do-i-completely-uninstall-node-js-and-reinstall-from-beginning-mac-os-x). Here are the commands:

~~~bash
lsbom -f -l -s -pf /var/db/receipts/org.nodejs.pkg.bom | while read f; do  sudo rm /usr/local/${f}; done

sudo rm -rf /usr/local/lib/node /usr/local/lib/node_modules /var/db/receipts/org.nodejs.*
~~~

That should remove all Node JS and NPM from Mac OS X. Now let's install NVM. As of June 2015, the following might be the easiest command to install NVM:

~~~bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.25.4/install.sh | bash
~~~

(Note: It's always safe to check if [NVM](https://github.com/creationix/nvm) updated their installation command.)

The install.sh script will insert a few lines to ~/.bashrc, however, Mac OS X for some reason won't source ~/.bashrc ([see details](http://apple.stackexchange.com/questions/119711/why-mac-os-x-dont-source-bashrc)). So we have to copy the inserted lines to ~/.bash_profile instead:

~~~bash
# Node version manager
export NVM_DIR="/Users/your_username/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm​​​
~~~

Change your_username to your home dir name. After sourcing ~/.bash_profile, NVM will be loaded and ready to use.

~~~bash
source ~/.bash_profile
~~~

Let's install Node JS, for example, version 0.10.38:

~~~bash
nvm install 0.10.38
nvm use 0.10.38
~~~

Type `node -v` to check if the correct version is installed and in use.

Node 0.10.38 is installed. However, it will be a bit tedious to type `nvm use 0.10.38` to switch to that version every time a terminal session starts. To fix that, I assign 'default' as an alias to v0.10.38:

~~~bash
nvm alias default 0.10.38
~~~

Versions tied to the 'default' alias will be the default active version.

You are free to install many versions of Node JS with NVM. For instance, here we install the stable version (0.12.4 as of June 2015):

~~~bash
nvm install stable
~~~

To list all installed Node Js, and see which one is active, use the `nvm ls` command:

![nvm ls](/img/nvm-ls.png)

The command reports that I have v0.10.38 and v0.12.4 installed. The current active version is 0.10.38. I have four aliases - default, node, sable, and iojs. Aliases can refer to other aliases. In this example, node refers stable, and stable refers 0.12.

<br />


## Conclusion

Managing versions is crucial but tedious. The task should be automated. NVM helps us manage multiple Node Js versions without a sweat. It becomes much easy to work in a working version, and try something new in the latest version. All I have to do is `nvm use default` and `nvm use stable`.

And a side benefit is that I don't have to `sudo npm install` any more. Just `npm install`.

<br />
