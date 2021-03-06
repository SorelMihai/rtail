# `rtail(1)`

[![Wercker CI](https://img.shields.io/wercker/ci/556547b7be632a8c751c857d.svg?style=flat-square)](https://app.wercker.com/project/bykey/54b073dac5b9156509c26031c78c98d4)
[![Coveralls](https://img.shields.io/coveralls/kilianc/rtail.svg?style=flat-square)](https://coveralls.io/r/kilianc/rtail)
[![NPM version](https://img.shields.io/npm/v/rtail.svg?style=flat-square)](https://www.npmjs.com/package/rtail)
[![NPM downloads](https://img.shields.io/npm/dm/rtail.svg?style=flat-square)](https://www.npmjs.com/package/rtail)
[![GitHub Stars](https://img.shields.io/github/stars/kilianc/rtail.svg?style=flat-square)](https://github.com/kilianc/rtail)
[![License](https://img.shields.io/npm/l/rtail.svg?style=flat-square)](https://www.npmjs.com/package/rtail)
[![Gitter](https://img.shields.io/badge/≡_gitter-join_chat_➝-04cd7e.svg?style=flat-square)](https://gitter.im/kilianc/rtail?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## Pipe your terminal output to the browser in seconds, using UNIX pipes.

`rtail` is a command line utility that grabs every line in `stdin` and broadcast it over **UDP**. That's it, nothing fancy, nothing complicated. You can tail your log files, your app output or whatever you wish to pipe, in `rtail` to a `rtail-server` and see multiple streams in the browser, realtime.

## Installation

    $ npm install -g rtail

## Web app

![](https://s3.amazonaws.com/rtail/github/dark.png)

![](https://s3.amazonaws.com/rtail/github/light.png)

## Rationale

If you develop software for work and you deploy your code on remote servers using multiple environments, or simply have multiple projects, you know that to monitor realtime your logs **you need to `ssh` to every single machine running your code**.

There are many log aggregation tools out there, and few of them are realtime. **Most of them require you to change your application source code to support their logging protocol/transport**.

This is meant to be a replacement of [logio](https://github.com/NarrativeScience/Log.io/commits/master) which was a cool idea, but not actively maintained anymore, doesn't support node v0.12.* and uses *TCP*, which means strict handshaking between servers and clients other than being resource consuming and very difficult to scale.

**`rtail` approach is very simple:**
* pipe something into it using [UNIX I/O redirection](http://www.westwind.com/reference/os-x/commandline/pipes.html) [[2]](http://www.codecoffee.com/tipsforlinux/articles2/042.html)
* broadcast every line using UDP
* `rtail-server` **if listening** will dispatch the stream into your browser using [socket.io](http://socket.io/).

**There is no persistent layer and it is not meant to persist any data**, `rtail` is a realtime monitoring tool meant to aggregate multiple streams and serve them with a modern web interface, mostly for debugging and realtime monitoring. If you need a persistent layer use something like [loggly](https://www.loggly.com/).

## Examples

In your init script:

    $ node server.js 2>&1 | rtail --id "api.myproject.com"

    $ mycommand | rtail > server.log

    $ node server.js 2>&1 | rtail --mute

Supports JSON lines

    $ while true; do echo [1, 2, 3, "hello"]; sleep 1; done | rtail
    $ echo { "foo": "bar" } | rtail

Using log files (log rotate safe!)

    $ node server.js 2>&1 > log.txt
    $ tail -F log.txt | rtail

For fun and debugging

    $ cat ~/myfile.txt | rtail
    $ echo "Server rebooted!" | rtail --id `hostname`

## Params

    $ rtail  -h
    Usage: cmd | rtail --host [string] --port [num] [--mute] [--id [string]]

    Examples:
      server | rtail --host 127.0.0.1 > server.log    Broadcast to localhost + file
      server | rtail --port 43567                     Custom port
      server | rtail --mute                           Only remote
      server | rtail --id api.domain.com              Name the log stream
      server | rtail --not-tty                        Strips ANSI colors


    Options:
      --mute, -m     Don't pipe stdin with stdout
      --host         The recipient server host     [default: "127.0.0.1"]
      --port, -p     The recipient server port     [default: 9999]
      --id, --name   The log stream id             [default: moniker()]
      --not-tty      Strips ansi colors
      --help, -h     Show help
      --version, -v  Show version number

## `rtail-server(1)`

`rtail-server` catches all messages broadcasted from every `rtail` client and serves a web interface to view the incoming log streams in realtime. **Under the hood uses [socket.io](http://socket.io) to pipe every incoming UDP message to the browser.**

There is tiny to nothing to configure, you can chage the default UDP/HTTP ports, but other than that you're all set.

## Examples

With default values

    $ rtail-server

Stay up to date!

    $ rtail-server --web-version stable

With custom ports

    $ rtail-server --web-port 8080 --udp-port 9090

With debugging on

    $ DEBUG=rtail:* rtail-server

With serving always latest stable webapp

    $ DEBUG=rtail:* rtail-server --web-version stable

Open your browser and start tailing logs!

## Params

    $ rtail-server -h
    Usage: rtail-server [--udp-host [string] --udp-port [num] --web-host [string] --web-port [num] --web-version [stable,unstable,<version>]]

    Examples:
      rtail-server --web-port 8080         Use custom http port
      rtail-server --udp-port 8080         Use custom udp port
      rtail-server --web-version stable    Always uses latest stable webapp
      rtail-server --web-version 0.1.3     Use webapp v0.1.3


    Options:
      --udp-host, --uh  The listening udp hostname       [default: "127.0.0.1"]
      --udp-port, --up  The listening udp port           [default: 9999]
      --web-host, --wh  The listening http hostname      [default: "127.0.0.1"]
      --web-port, --wp  The listening http port          [default: 8888]
      --web-version     Define web app version to serve
      --help, -h        Show help
      --version, -v     Show version number

## UDP Broadcasting

You may want to scale and broadcast on multiple servers, in that case just instruct the `rtail` client to stream to the broadcast address and every message will be delivered to all servers in your subnet.

## Authentication layer

The webapp doesn't have an authentication layer for the time being, it's assumed you run it behind a VPN or a reverse proxy with a simple `Authorization` header check.

# How to contribute

This project follows the awesome [Vincent Driessen](http://nvie.com/about/) [branching model](http://nvie.com/posts/a-successful-git-branching-model/).

* You must add a new feature on his own topic branch
* You must contribute to hot-fixing directly into the master branch (and pull-request to it)

This project follows (more or less) the [Felix's Node.js Style Guide](http://nodeguide.com/style.html), your contribution must be consistent with this style.

The test suite is written on top of [visionmedia/mocha](http://visionmedia.github.com/mocha/) and it took hours of hard work. Please use the tests to check if your contribution is breaking some part of the library and add new tests for each new feature.

    $ npm test

## Contributors

* [Kilian Ciuffolo](https://github.com/kilianc)
* [Luca Orio](https://www.behance.net/lucaorio)
* [Sandaruwan Silva](https://github.com/s-silva)

## License

_This software is released under the MIT license cited below_.

    Copyright (c) 2015 Kilian Ciuffolo, me@nailik.org. All Rights Reserved.

    Permission is hereby granted, free of charge, to any person
    obtaining a copy of this software and associated documentation
    files (the 'Software'), to deal in the Software without
    restriction, including without limitation the rights to use,
    copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the
    Software is furnished to do so, subject to the following
    conditions:

    The above copyright notice and this permission notice shall be
    included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
    EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
    OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
    NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
    HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
    WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
    FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
    OTHER DEALINGS IN THE SOFTWARE.
