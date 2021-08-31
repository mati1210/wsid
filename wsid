#!/usr/bin/env python
# SPDX-License-Identifier: GPL-2.0-or-later
# SPDX-FileCopyrightText: matheuz1210 <matheuz1210@gmail.com>
# created 21Jul2021, 00:50AM -0300

import os
import sys
import http.server
import socket
import urllib.parse as urlparse
from html import escape
import zipfile
import io


__version__ = '0.3'

# extensions for supported images/videos
imgext = ('.jpg', '.jpeg', '.jfif', '.png', '.avif', '.jxl', '.gif', '.webp')
vidext = ('.mp4', '.mkv', '.webm')

html = (
    # start
    '''<!DOCTYPE html><html>
<head><meta charset='UTF-8'></head>
<body>
''',
    # end
    '''
</body></html>
''',
    # queries
    '''
<hr><b>Sort by</b><br>
<span>
<a href='?sort=6'>Size (smallest first)</a> |
<a href='?sort=8'>Last modified (oldest first)</a> |
<a href='?sort=10'>Name (A to Z)</a></span><br>
<span>
<a href='?sort=6&reverse=1'>Size (largest first)</a> |
<a href='?sort=8&reverse=1'>Last modified (recent first)</a> |
<a href='?sort=10&reverse=1'>Name (Z to A)</a></span><br>
<br><a href='?zip=1'>download as zip</a>
<hr>
''',
    # no files
    '''<h1>No supported file types</h1>
<p>this is where images/videos would show up. if there was some!<br><br>add
images/videos to the selected folder and refresh the page to see them</p>'''
)


def sort(files, num=8, reverse=False):
    # if num is an os.stat item, sort by stat, otherwise sort alphabetically
    if num < 10:
        fdict = {}
        for f in files:
            fdict |= {
                f: os.stat(f)[num]
            }
        return dict(sorted(fdict.items(), key=lambda x: x[1], reverse=reverse)).keys()
    else:
        files.sort(reverse=reverse)
        return files


def handle_queries(path, query):
    queries = urlparse.parse_qs(path.split('?', 1)[1])

    qlist = []
    for q in query:
        if q in queries:
            qlist.append(int(queries[q][0]))
        else:
            qlist.append(False)

    return qlist


class coolHandler(http.server.SimpleHTTPRequestHandler):

    def generate_zip(self, files):
        self.send_header('Content-type', 'application/zip')
        fd = io.BytesIO()
        flist = []

        for f in files:
            flower = f.lower()

            if flower.endswith(imgext + vidext):
                flist.append(f)

        with zipfile.ZipFile(fd, 'a') as zf:
            [zf.write(f) for f in flist]

        return fd.getbuffer()

    def generate_html(self, files):
        self.send_header('Content-type', 'text/html')
        htmltags = []

        for f in files:
            flower = f.lower()
            url = urlparse.quote(f)
            f = escape(f)

            if flower.endswith(imgext):
                htmltags.append(f'''<a href="{url}" download>
                                <img src="{url}" alt="{f}"></a>''')
            elif flower.endswith(vidext):
                htmltags.append(f'<video src="{url}" controls>"{f}"</video>')

        if not htmltags:
            return bytes(html[0] + html[3] + html[1], 'utf-8')

        return bytes(html[0] + html[2] + '\n'.join(htmltags) + html[1], 'utf-8')

    def list_directory(self, path):
        files = os.listdir()
        self.send_response(200)

        # get queries to sort files and find out to use html or zip
        queries = ('sort', 'reverse', 'zip')
        if '?' in self.path:
            num, reverse, zipf = handle_queries(self.path, queries)
            if zipf:
                wfile = self.generate_zip(files)
            elif num:
                if reverse >= 1:
                    files = sort(files, num, True)
                else:
                    files = sort(files, num)
                wfile = self.generate_html(files)
        else:
            files = sort(files)
            wfile = self.generate_html(files)

        # sends the content
        self.send_header('Content-Length', str(len(wfile)))
        self.end_headers()
        self.wfile.write(wfile)


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
            print('directory not specified and PyQt5 not installed.')
            sys.exit(1)

        gui = QApplication(sys.argv)
        root = QFileDialog.getExistingDirectory()

    try:
        os.chdir(root)
    except (NotADirectoryError, FileNotFoundError):
        print(f'"{root}" not found or is not a directory')
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
