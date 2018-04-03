[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/tarampampam/nod32-update-mirror/master/LICENSE) ![Language](https://img.shields.io/badge/language-bash-yellowgreen.svg)

# ESET Nod32 Update Mirror

> английская версия [доступен здесь](https://github.com/tarampampam/nod32-update-mirror/blob/master/README.md).

The script for creating a mirror of the anti-virus updates database "Eset Nod32". For its full-fledged operation it will be required::

  - **`bash`** *(Tested on versions `4.1.11(2)`, `4.2.24(1)` and `4.2.45(1)`)* or `cygwin`;
  - **`curl`** *(Tested on versions `7.29.0`, `7.37.0`)* and **`wget`** *(Tested on versions , and `1.14`, `1.15` and `1.18`)*;
  - **`unrar`** *(Tested on versions `3.39`, `4.0` and `5.00 beta 3`, for work with official mirrors)*;
  - `sed`, `awk` and some other "standard" applications;
  - **`nginx`** *(тested on versions `1.6.3`, `1.9.12` and `1.10.1` without any exotic modules)*.

Startup parameters and additional functions, see running the script with the flag `--help`.

![Console screenshot](https://cloud.githubusercontent.com/assets/7326800/16709324/ee055c38-4626-11e6-832e-17f40576d8c2.png)

## <i class="icon-file"></i>Features

 - Does not require any exotic or resource-intensive applications from the system, economically refers to system resources;
 - Works successfully with different versions of Eset Nod32 antivirus software *(for checking the "working" directories of different versions of Eset Nod32, checks `User-Agent` and redirects are used by means `nginx`)*;
 - Can automatically search for and use *(keeping their list up to date)* free update keys (**ATTENTION, THIS FUNCTIONAL IS ONLY FOR INTRODUCING AND TESTING THE WORK! USE ONLY LEGALLY PURCHASED KEYS!**)
 - It is possible to place the update database both in the root directory of the domain and in an arbitrary sub-directory *(you will need to rewrite the paths in the configuration file yourself `nginx`)*;
 - If you specify non-official update servers *(they can be specified up to 10 pcs.)* And an error occurs in the process from the first specified server, the update will occur from the second one, differently from the third one, and so on;
 - Implemented the ability to download updates only for certain software products, platforms, languages ​​and versions of Eset Nod32;
 - Supports debug mode to quickly identify sources of possible problems;
 - Writes a detailed log;
 - It is possible to specify the speed limits and delays in downloading update files;
 - At the completion of the update, the version of the update database and the update date are written to separate files *(file names are configured)*;
 - Only downloaded the updated files.


## <i class="icon-download"></i>Installation

For example, consider installing a clean distribution of **CentOS 7** *(from other distributions will only differ package manager, the location of some configs and installation method `unrar`)*.

- Go to the home directory:
```shell
$ cd ~
```
 - We put the necessary packages:
```shell
$ sudo yum -y install epel-release 
$ sudo yum -y install nano wget curl git nginx # Only after installing epel-release
$ wget -O unrar.rpm http://pkgs.repoforge.org/unrar/unrar-5.0.3-1.el6.rf.$(arch).rpm
$ sudo rpm -Uvh ./unrar.rpm && rm -f ./unrar.rpm
```
- We clone this repository:
```shell
$ git clone https://github.com/tarampampam/nod32-update-mirror.git
```
- We will move the directory with the web interface in `/usr/share/nginx/`, and afterwards it will be used as a mirror file store *(accessible via the web, of course)*:
```shell
$ mv ./nod32-update-mirror/webroot /usr/share/nginx/nod32mirror
```
- Let's proceed to the configuration `nginx`, for which we take an example of the configuration and correct it:
```shell
$ sudo cp ./nod32-update-mirror/nginx.server.conf /etc/nginx/conf.d/nod32mirror.conf
$ nano /etc/nginx/conf.d/nod32mirror.conf
# We expose:
#   listen 80;
#   server_name %domain_name% complete; or comment this line, if necessary to access the mirror was just the IP address of the server
#   root /usr/share/nginx/nod32mirror;
```
- If `server_name` it was not specified above, then you must delete or comment out the virtual server `nginx` "by default", for which we perform:
```shell
$ sudo nano /etc/nginx/nginx.conf
# Remove or comment on the 'server' section, otherwise the server will respond with a 'stub'
```
- After that we test the configuration, and if everything is good - then we say to `nginx` reread our configs:
```shell
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo nginx -s reload
```
- We open the specified `server_name` *(or its IP address)* in the browser, check the correct display of the web interface;
> Optionally:
> You can download your Eset Nod32 antivirus distributions to a directory `/usr/share/nginx/nod32mirror/download` and fix the file `/usr/share/nginx/nod32mirror/webface/index.html`, by specifying the `nod32button.settings.download.links` distribution file names in the files, and by setting `nod32button.settings.download.enabled` to `true`. Thus, visitors to the mirror will be able not only to receive updates from it, but also to download the necessary distributions *(or already preconfigured to that very mirror)* when they visit.

- Create a directory in which the script will be located, and move its directory into it:
```shell
$ NOD_SCRIPTS_DEST=~/scripts
$ mkdir -p "$NOD_SCRIPTS_DEST"
$ mv ./nod32-update-mirror/nod32-mirror "$NOD_SCRIPTS_DEST"
```
- Let's give the scripts the right to execute:
```shell
$ find "$NOD_SCRIPTS_DEST" -type f -name '*.sh' -exec chmod +x {} \;
```
- We'll create a copy of the config, and fix it for our needs *(it's necessary to work with the copy in order that during the next update of the script we did not have to rewrite its settings)*:
```shell
$ cp $NOD_SCRIPTS_DEST/nod32-mirror/settings.conf $NOD_SCRIPTS_DEST/nod32-mirror/conf.d/default.conf
$ nano $NOD_SCRIPTS_DEST/nod32-mirror/conf.d/default.conf
# Set:
#   export NOD32MIRROR_USE_FREE_KEY=1;
#   export NOD32MIRROR_MIRROR_DIR="/usr/share/nginx/nod32mirror";
#   export NOD32MIRROR_LOG_PATH="$HOME/nod32mirror.log";
```
- **An indication in the configuration is to use a free key - just to verify the operability!** After all the checks, please, **purchase a license key** and use only it, for what point it in `NOD32MIRROR_SERVER_0` with the value set `NOD32MIRROR_USE_FREE_KEY` to `0`.
- We perform a trial run:
```shell
$ $NOD_SCRIPTS_DEST/nod32-mirror/nod32-mirror.sh
# If all goes well, then:
$NOD_SCRIPTS_DEST/nod32-mirror/nod32-mirror.sh --get-key
# If the testing period is somewhat delayed and one or all of the keys will lose relevance - the script will automatically find new ones. If at this step, too, everything is fine, then:
$ $NOD_SCRIPTS_DEST/nod32-mirror/nod32-mirror.sh --update
```
- After that, the update should run successfully. Upon completion, you should check the correctness of the update for Eset Nod32 antivirus software that is configured for the newly created update mirror.
- Remove unnecessary:
```shell
$ rm -Rf ./nod32-update-mirror/
```

> To automate the update, add the following line to the crowns:
> ```shell
> 0 */3 * * * nice -n 15 bash ~/scripts/nod32-mirror/nod32-mirror.sh --update
> ```
> Warning: You must specify the full path to the script.

### <i class="icon-chat"></i>It does not work for me!

If you have any problems with the launch - you can always [tell us about it](https://github.com/tarampampam/nod32-update-mirror/issues/new).
The message **обязательно** apply the contents of the configuration file, the version of the software (`cat /proc/version`, `bash --version`, `wget -V`, `curl -V`), and the output of the script **in the form of not changed**!

## <i class="icon-cog"></i>Setting up the script

All settings are specified in the file `settings.conf`. Each option is accompanied by a detailed description and an example of use. Please be careful when configuring it.

You can also place your configurations in a directory `./nod32-mirror/conf.d` with arbitrary names, but with an extension `*.conf`.

## <i class="icon-cog"></i>Configuring the Web Interface

![Web-interface screenshot](https://cloud.githubusercontent.com/assets/7326800/16712120/bca39a8c-4695-11e6-970c-477c5e4dd081.png)

The web interface is a page executed in a minimalist style, which is displayed **instead of the** list of mirror files of the update. It is possible to configure it in such a way that when clicking on the logo, the required distribution will be downloaded *(if any one is specified, regardless of the indication of its bit depth)*, or a choice is given to the user *(choose according to the bit depth of its operating system)*. Also, by default, the output of the last date of mirror update and the version of the anti-virus database *(when hovering over the logo and having the appropriate settings)* is configured.
The settings are made by editing the file `./webroot/webface/index.html`.

### <i class="icon-upload"></i>History of changes

The history of changes is available a [this link](https://github.com/tarampampam/nod32-update-mirror/blob/master/CHANGESLOG.md).

### <i class="icon-upload"></i>References
 - [Post to blog](http://blog.kplus.pro/?p=2378)
 - [Post on habra](http://habrahabr.ru/post/232163/)
 - [editor readme.md](https://stackedit.io/)

### <i class="icon-refresh"></i>MIT License

> Copyright (c) 2014-2016 &lt;[github.com/tarampampam](http://github.com/tarampampam/)&gt;

> This license allows persons who receive a copy of this software and related documentation (hereinafter referred to as «Software»), to use the Software free of charge, without limitation, including unlimited right to use, copy, modify, add, publish, distribute, sublicense and / or sale of copies of the Software, as well as to persons to whom this Software is provided, provided that the following conditions are met:

> The above copyright notice and these terms and conditions must be included in all copies or significant portions of this Software.

> THIS SOFTWARE IS PROVIDED «AS IS», WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR CLAIMS FOR DAMAGES, LOSSES, OR OTHER REQUIREMENTS UNDER APPLICABLE CONTRACT, TORT OR OTHERWISE, ARISING FROM HAVING THE RESULT OF OR RELATED TO THE SOFTWARE OR THE USE OF THE SOFTWARE OR OTHER DEALINGS IN THE SOFTWARE.
