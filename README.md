yo-mean-vagrant
===============

## Synopsis

A *working* Vagrant (https://www.vagrantup.com/) box to quickly deploy a new MEAN stack (http://en.wikipedia.org/wiki/MEAN) for full-stack development with Yeoman (http://yeoman.io).

Deployed client: Trusty64 (ubuntu/trusty64)

Requirements: Vagrant & VirtualBox (https://www.virtualbox.org/) installed

Tested Host Platforms: OS X, Windows

## Why?

I couldn't find a single *working* vagrant box for setting up a Yeoman MEAN stack. Either permissions problems, or ruby problems, etc. Yeah, crazy right?

Also, If you're using Windows as a host, a little bit more work is needed due to long path names (see https://github.com/mitchellh/vagrant/issues/713) as npm dependencies tend to nest quite deeply.

## Installation

1. Download & Install Virtual Box (https://www.virtualbox.org/wiki/Downloads)

2. Download & Install Vagrant (https://www.vagrantup.com/downloads.html)

3. Create the directory where you will store (and eventually map) your code base on the host machine.

  I.e. under windows
  `> mkdir C:\mycode`
  or OS X
  `> mkdir ~/mycode`
  (By default, the mapped path inside the client is "/var/www/mean", you can change this in the Vagrant file if you like.)

4. Download & unzip the code from this git (https://github.com/sekmun/yo-mean-vagrant) into a new directory (e.g., vagrant/mean)

5. Edit "./Vagrantfile"
  * set your sync folder by editing this line 
  
  `config.vm.synced_folder "C:\\mycode", "/var/www/mean", create: true`

6. Start vagrant
  * `> vagrant up`

7. Go make a cup of coffee... then

  `> vagrant ssh`

  `$ cd /var/www/mean`

  `$ mkdir /var/www/mean/yourProjectName && cd /var/www/mean/yourProjectName`
  
8. If you are using Windows, do the additional **Windows Workaround** step below. If you don't do it under Windows, the next steps will fail.

  If you've done everything to this point, it's time to create the scaffolded app.

  `$ yo angular-fullstack`

  Go through and answer all the questions, Javascript, HTML, Sass, uiRouter, Bootstrap, UI Bootstrap, Mongoose
  This will take about couple of minutes to generate a scaffolded angular-fullstack application. 
  Once this is completed, you may want to edit the generated `/var/www/mean/yourProjectName/Gruntfile.js`

  > If you want to change the IP address of where the machine is served, edit your Gruntfile.js. Changing the IP to 0.0.0.0 should allow the site to be accessed from all IPs.

  `$ grunt serve`

  At this point if everything is working you should see that the [Running "watch" task] is active.
  
9. Fire up your browser on the host machine and point it to

  `http://localhost:9000`
  
  or
  
  `http://10.0.3.2:9000`
  
You should see the scaffolded Yeoman MEAN application. If you bring up another browser you can interact with the data on both browser windows and see the two-way real time binding work.

### Windows Workaround

Thanks to this https://github.com/mitchellh/vagrant/issues/713 discussion which helped create a workaround for this.

So the problem is this: Windows *can* actually hold path names longer than 264 characters by using the unicode library, but VirtualBox for Windows doesn't seem to implement this library in mapping the sync folders. Owing to extremely long path names in npm modules, the sync folder mapping will thus fail (at least for yo angular-fullstack) when you do a npm install.

To fix this, the hack is to *relocate* the `./node_modules` directory (this being the only offender of long paths) to a hard path inside the linux client (linux - which doesn't complain about long paths), then symlink from /var/www/mean/node_modules (or whatever your path is) Then npm will install these libraries inside the client (where you will run `grunt serve` anyway). 
The downside is that you can't "see" these files on the Windows box, but since you're just interested in port 9000 and you are coding in the `client` and `server` directories (I hope you're not hacking dependencies!) all's well that ends well. Happy days.

1. Open a Windows command prompt

2. type `fsutil behavior set SymlinkEvaluation L2L:1 R2R:1 L2R:1 R2L:1 `

3. In the guest machine (after `vagrant ssh`)

  1. Create the "hard" location of your files, inside your client's home directory (this is where the "real" files will live):

    `mkdir -p ~/yourProjectName/node_modules`
  
  2. Symlink your project to the above location:
    
    `ln -s ~/yourProjectName/node_modules /var/www/mean/yourProjectName/node_modules`

You can now `install npm` packages in your project directory without worrying about path length.

Again, if you ever lose or destroy your vagrant client you'll have to create the "hard" directory and do another `npm install` to get these file dependencies back.

## Issues

This box opens port 9000 (express) and 35729 (livereload) if you want to access the client via http://localhost. Make sure you're not running anything on these ports on the host.

I encountered some problems with the livereload port 35729 being used on the host machine, so `vagrant up` would hang at "Booting VM". To overcome this, I removed the port mapping in Vagrantfile.

It means you can't access the client via `http://localhost:9000`, but you can still use `http://10.0.3.2:9000`

## License

The MIT License (MIT)

Copyright (c) 2015 Sek-Mun

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
