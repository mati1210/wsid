# wsid 0.2

wsid (web show image dir), inspired by [woof](https://github.com/simon-budig/woof), runs a webserver that generates a HTML with every image/video on a directory
see example [here.](https://youtu.be/23L44tVQZZE)

currently requires python 3.9 (when wsid 1.0 will be released, that requirement will be raised to python 3.10)
## install
### arch/manjaro
[wsid](https://aur.archlinux.org/wsid) and [wsid-git](https://aur.archlinux.org/wsid-git) are avaliable on the AUR.
### pip
```
pip install wsid
```
also run 
```
pip install PyQt5
```
if you'd like to run the gui.
## usage
just
```
wsid <path to dir>
```
or
```
wsid
```
to run the gui (requires PyQt5)
