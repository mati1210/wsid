#!/usr/bin/env python
# SPDX-License-Identifier: GPL-2.0-or-later
# SPDX-FileCopyrightText: matheuz1210 <matheuz1210@gmail.com>
# created 21Jul2021, 00:50AM -0300

import os, sys
import http.server, socket, urllib.parse, html
import zipfile, io


__version__ = '1.1'

# extensions for supported images/videos
IMGEXT = ('.jpg', '.jpeg', '.jfif', '.png',
          '.avif', '.jxl', '.gif', '.webp')
VIDEXT = ('.mp4', '.mkv', '.webm')

HTML = {

    'header': """<!doctype html><html>
<head><meta charset='UTF-8'><meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>img,video{max-width:100%;height:auto}</style></head><body id='top'>
<a href='#top' style='position:fixed;bottom:1em;right:1em;background-color:#FFF'>top</a>\n""",

    'footer': '\n</body></html>',

    'queries': """<hr><b>Sort by</b><br>
<a href='?s=sm&p={p}&i={i}'>Size (smallest first)</a> |
<a href='?s=old&p={p}&i={i}'>Last modified (oldest first)</a> |
<a href='?s=az&p={p}&i={i}'>Name (A to Z)</a><br>
<a href='?s=big&p={p}&i={i}'>Size (biggest first)</a> |
<a href='?s=new&p={p}&i={i}'>Last modified (newest first)</a> |
<a href='?s=za&p={p}&i={i}'>Name (Z to A)</a>
<br><br><a href='?z'>download as zip</a><br><br>
<a href='?s={s}&p={pp}&i={i}'>Previous</a><b> Page {p} </b><a href='?s={s}&p={pn}&i={i}'>Next</a>
<br><br><b>Images per page</b><br>
<a href='?s={s}&p={p}&i=20'>20</a> | <a href='?s={s}&p={p}&i=80'>80</a> |
<a href='?s={s}&p={p}&i=100'>100</a> | <a href='?s={s}&p={p}&i=200'>200</a><hr>\n""",
}


class query:
    imgcount, page, sort, zipf = 20, 0, 'new', False  # default query values


# file sort
def fsort(files, sort, cache={}):
    reverse = False

    # memoization kinda
    try:
        if cache['lfiles'] == files and cache['lsort'] == sort:
            return cache['sorted']
    except KeyError:
        pass
    cache['lfiles'], cache['lsort'] = files, sort

    # sort
    if sort == 'az':     # a-z
        cache['sorted'] = sorted(files)
        return cache['sorted']

    elif sort == 'za':   # z-a
        cache['sorted'] = sorted(files, reverse=True)
        return cache['sorted']

    elif sort == 'sm':   # size (smallest first)
        sort = 6

    elif sort == 'big':  # size (biggest first)
        sort = 6
        reverse = True

    elif sort == 'old':  # last modified (oldest first)
        sort = 8

    else:                # last modified (newest first)
        sort = 8
        reverse = True

    cache['sorted'] = sorted(files, key=lambda x: os.stat(x)[sort], reverse=reverse)
    return cache['sorted']


def generate_zip(files):
    fd = io.BytesIO()

    with zipfile.ZipFile(fd, 'a') as zf:
        [zf.write(f) for f in files if f.lower().endswith(IMGEXT + VIDEXT)]

    return fd.getbuffer()


def generate_html(files):
    htmltags = []

    for f in files:
        flower = f.lower()
        url = urllib.parse.quote(f)
        e = html.escape(f)

        if flower.endswith(IMGEXT):
            htmltags.append(f'<a href="{url}" download><img src="{url}" alt="{e}"></a>')
        elif flower.endswith(VIDEXT):
            htmltags.append(f'<video src="{url}" controls>"{e}"</video>')
        elif not os.path.isdir(f):
            htmltags.append(f'''<br><p>file {e} is not an compatible image/video.
would you like to <a href="{url}"> download it?</a></p><br>''')

    queries = HTML['queries'].format(p=query.page, pp=query.page - 1, pn=query.page + 1,
                                     s=query.sort, i=query.imgcount)

    return bytes(HTML['header'] + queries + '\n'.join(htmltags) + queries + HTML['footer'],
                 'utf-8')


class coolHandler(http.server.SimpleHTTPRequestHandler):

    server_version = 'wsid/' + __version__

    def list_directory(self, path):
        files = os.listdir()

        # handle queries
        if '?' in self.path:
            queries = urllib.parse.parse_qs(self.path.split('?', 1)[1], True)

            if 'z' in queries:
                query.zipf = True
            else:
                try:
                    query.imgcount = int(queries['i'][0])
                    query.page = int(queries['p'][0])
                    query.sort = queries['s'][0]
                except KeyError:
                    pass

                if query.page < 0: query.page = 0

        # sends the content
        self.send_response(200)

        if query.zipf:
            self.send_header('Content-type', 'application/zip')
            wfile = generate_zip(files)
        else:
            index = query.page * query.imgcount
            if len(files) < index: index = 0
            files = fsort(files, query.sort)[index:index + query.imgcount]

            self.send_header('Content-type', 'text/html')
            wfile = generate_html(files)

        self.send_header('Content-Length', str(len(wfile)))
        self.end_headers()
        self.wfile.write(wfile)

    def do_GET(self):
        try:
            return super().do_GET()
        except (BrokenPipeError, ConnectionResetError, ConnectionAbortedError):
            print(f'{self.path} was interrupted', file=sys.stderr)


# for ipv6 support
class coolServer(http.server.ThreadingHTTPServer):
    address_family = socket.AF_INET6

    def server_bind(self):
        self.socket.setsockopt(socket.IPPROTO_IPV6, socket.IPV6_V6ONLY, 0)
        return super().server_bind()


# get local ip
def ip():
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
        s.connect(('8.8.8.8', 80))
        return s.getsockname()[0]


def main():
    # try to get directory from cli, otherwise try to run gui
    try:
        root = sys.argv[1]
    except IndexError:
        try:
            from PyQt5.QtWidgets import QApplication, QFileDialog
        except ModuleNotFoundError:
            print('directory not specified and PyQt5 not installed.', file=sys.stderr)
            sys.exit(1)

        gui = QApplication(sys.argv)
        root = QFileDialog.getExistingDirectory()
        del(gui)

    try:
        os.chdir(root)
    except (NotADirectoryError, FileNotFoundError):
        print(f'"{root}" not found or is not a directory', file=sys.stderr)
        sys.exit(1)

    # run server
    port = 80
    try:
        httpd = coolServer(('::', port), coolHandler)
    except PermissionError:
        port = 8080
        httpd = coolServer(('::', port), coolHandler)

    print(f'http://{ip()}:{port}')

    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        httpd.server_close()


if __name__ == '__main__':
    main()
