---
layout: default
title: INS'hAck 2019 - Exploring the Universe
---

<h1 align="center">[INS'hAck 2019 - Exploring the Universe]</h1>
## Challenge Descripton:
```
Will you be able to find the flag in the universe/? I've been told that the guy who wrote this nice application called server.py is a huge fan of nano (yeah... he knows vim is better).<br>
URL: [http://exploring-the-universe.ctf.insecurity-insa.fr/](http://exploring-the-universe.ctf.insecurity-insa.fr/)
```

We are given a web page with the flag located in <i>universe/flag</i>.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/index.png"></p><br>

Hmm, it looks like the page contains a game, nothing special. Let's take a look at the page source.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/page_source.png"></p><br>

Nothing special either, just a game written in <b>Javascript</b>. What sorcery is this? xD<br>
Let's get back to the challenge description. It says that the guy who wrote the challenge used a certain kind of text editor. At this point, the writer couldn't determine whether it's <i>nano</i> or <i>vim</i>. But it gave us the idea that we should look for something related to these 2 text editors and the file <b>server.py</b>. At first, the writer tried to access <b>server.py</b> but failed and got into 404 (not found) error page.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/404.png"></p><br>

So, first thing first, when we write anything using <i>nano</i> or <i>vim</i>, we can also save the file with whatever extension we want. That is, if you save the file properly.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/open_nano.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/write_nano.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/save_nano.png"></p><br>

But what if we try to modify a file in <i>nano</i> and then without saving it we force close the terminal? This is what happened.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/open_nano.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/modify_nano.png"></p><br>

Then we force close the terminal and reopen it.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/emergencyfile_nano.png"></p><br>

It creates a so-called <i>emergency file</i> to save your progress called <b>\<filename\>.save</b>.<br>

Next, if we try to write something in <i>vim</i> and save the file properly, it will create the file we want.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/open_vim.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/write_vim.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/save_vim.png"></p><br>

But what is we write something and then force close the terminal? Here's what happens.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/modify_vim.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/tempfile_vim.png"></p><br>

It creates a temporary file called <b>.\<filename\>.swp</b>. <i>Note: There is a dot (.) in front of the filename since it is a hidden file.<i><br>

<i>Note: The writer found out recently that if you write something in nano and force close the nano with CTRL + Z it creates the same temporary file as vim.</i><br>

Since now we have these informations, perhaps this website has a temporary file of <b>server.py</b> that we can access. So here we try to access:
```
http://exploring-the-universe.ctf.insecurity-insa.fr/.server.py.swp
```
And it downloads the source code.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/source_download.png"></p><br>

Now let's take a look at the source code.<br>
```python
from pathlib import Path
from mimetypes import guess_type
from aiohttp import web

ROOT = Path().resolve()
print(ROOT)
PUBLIC = ROOT.joinpath('public')

async def stream_file(request, filepath):
    '''Streams a regular file
    '''
    filepath = PUBLIC.joinpath(filepath).resolve()
    if filepath.is_dir():
        return web.Response(headers={'DT': 'DT_DIR'})
    if not filepath.is_file():
        raise web.HTTPNotFound(headers={'DT': 'DT_UNKNOWN'})
    try:
        filepath.relative_to(ROOT)
    except:
        raise web.HTTPForbidden(reason="You can't go beyond the universe...")
    mime, encoding = guess_type(str(filepath))
    headers = {
        'DT': 'DT_REG',
        'Content-Type': mime or 'application/octet-stream',
        'Content-Length': str(filepath.stat().st_size)
    }
    if encoding:
        headers['Content-Encoding'] = encoding
    resp = web.StreamResponse(headers=headers)
    await resp.prepare(request)
    with filepath.open('rb') as resource:
        while True:
            data = resource.read(4096)
            if not data: break
            await resp.write(data)
    return resp

async def handle_403(request):
    '''Stream 403 HTML file
    '''
    return await stream_file(request, '403.html')

async def handle_404(request):
    '''Stream 404 HTML file
    '''
    return await stream_file(request, '404.html')

def create_error_middleware(overrides):
    '''Create an error middleware for aiohttp
    '''
    @web.middleware
    async def error_middleware(request, handler):
        '''Handles specific web exceptions based on overrides
        '''
        try:
            response = await handler(request)
            override = overrides.get(response.status)
            if override:
                return await override(request)
            return response
        except web.HTTPException as ex:
            override = overrides.get(ex.status)
            if override:
                return await override(request)
            raise
    return error_middleware

def setup_error_middlewares(app):
    '''Setup error middleware on given application
    '''
    error_middleware = create_error_middleware({
        403: handle_403,
        404: handle_404
    })
    app.middlewares.append(error_middleware)

async def root(request):
    '''Web server root handler
    '''
    path = request.match_info['path']
    if not path:
        path = 'index.html'
    path = Path(path)
    print(f"client requested: {path}")
    return await stream_file(request, path)

def app():
    app = web.Application()
    setup_error_middlewares(app)
    app.add_routes([web.get(r'/{path:.*}', root)])
    web.run_app(app)

if __name__ == '__main__':
    app()
```
Okay, at first this source code was so damn confusing since the writer had zero experience with ALL of these library. It took around 2-3 hours for the writer to understood the code.<br>
The most important part of this source code is actually this part:
```python
ROOT = Path().resolve()
print(ROOT)
PUBLIC = ROOT.joinpath('public')

async def stream_file(request, filepath):
    '''Streams a regular file
    '''
    filepath = PUBLIC.joinpath(filepath).resolve()
```
<b>CMIIW</b>
This part of code tells us that the root path will be concatenated with "public" and then with our path input. From this part, we can assume that maybe we can do a <i>path traversal</i> attack.<br>

The writer then tried to debug the code in the local machine since we now have the source code. Remember that the flag is located in <i>universe/flag</i>. So, in order to get to the flag we need to get back one directory to the root, and then access the universe directory then flag. But if we tried to do it casually it happens like this.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/test_input.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/got_404.png"></p><br>

We got a 404 error. Let's see what the server received.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/debug.png"></p><br>

We can see that the server removed the "../". Then how can we access the flag? There are 2 ways:
* Use URL Encode on "/"
* Use cURL option --path-as-is

In this writeup, we will use the first option. By converting "/" into url encoded form "%2F" we can technically access the <i>universe/flag</i>.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/debug_flag.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/debug_flag_terminal.png"></p><br>

And by implementing this method to the challenge link it will download the flag file.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/inshack2019_explore/flag_download.png"></p><br>

And voila! FLAG!<br>
Flag: INSA{3e508f6e93fb2b6de561d5277f2a9b26bc79c5f349c467a91dd12769232c1a29}
