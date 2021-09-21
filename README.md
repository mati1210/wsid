# wsid

wsid (web show image dir), inspired by [woof](https://github.com/simon-budig/woof), runs a webserver that generates a HTML with every image/video on a directory
see example [here.](https://youtu.be/23L44tVQZZE)

python3.7+
## install
### arch/manjaro
[wsid](https://aur.archlinux.org/wsid) and [wsid-git](https://aur.archlinux.org/wsid-git) are avaliable on the AUR.
### pip
```
pip install 'wsid[gui]'
```
if that doesn't work, omit `[gui]` to install without the gui
### else
get latest release from [the releases page](https://github.com/matheuz1210/wsid/releases) and copy wsid to $PATH
## usage
just
```
wsid <path to dir>
```
or
```
wsid
```
to run the gui
