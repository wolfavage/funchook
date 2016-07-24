Duckhook - an API hook library
==============================

Note: This is unstable. Some functions may be changed.

This library depends on [diStorm3][].

[![Build Status](https://travis-ci.org/kubo/duckhook.svg?branch=master)](https://travis-ci.org/kubo/duckhook)

TODO
----

* write documents.
* add a function to get the error reason when `duckhook_prepare` returns NULL.
* add a function to debug duckhook itself.

Supported Platform
-----------------

* Linux x86_64 (both glibc and musl libc)
* Linux x86 (both glibc and musl libc)
* OS X x86_64 (tested on 10.11 El Capitan)
* Windows x64 (*1)
* Windows 32-bit (*1)

*1 compiled by mingw-w64 and tested on Wine. I haven't tested it on Windows yet.

Compilation
-----------

```shell
$ git clone https://github.com/kubo/duckhook.git
$ git submodule init
$ git submodule update # clone diStorm3
$ cd src
$ make
```

Example
-------

```c
static ssize_t (*send_func)(int sockfd, const void *buf, size_t len, int flags);
static ssize_t (*recv_func)(int sockfd, void *buf, size_t len, int flags);

static ssize_t send_hook(int sockfd, const void *buf, size_t len, int flags);
{
    ssize_t rv;

    ... do your task: logging, etc. ...
    rv = send_func(sockfd, buf, len, flags); /* call the original send(). */
    ... do your task: logging, checking the return value, etc. ...
    return rv;
}

static ssize_t recv_hook(int sockfd, void *buf, size_t len, int flags);
{
    ssize_t rv;

    ... do your task: logging, etc. ...
    rv = recv_func(sockfd, buf, len, flags); /* call the original recv(). */
    ... do your task: logging, checking received data, etc. ...
    return rv;
}

int install_hooks()
{
    duckhook_t *duckhook = duckhook_create();
    int rv;

    /* Prepare hooking.
     * The return value is used to call the original send function
     * in send_hook.
     */
    send_func = send;
    rv = duckhook_prepare(duckhook, (void**)&send_func, send_hook);
    if (rv != 0) {
       /* error */
       ...
    }

    /* ditto */
    recv_func = recv;
    rv = duckhook_prepare(duckhook, (void**)&recv_func, recv_hook);
    if (rv != 0) {
       /* error */
       ...
    }

    /* Install hooks.
     * The first 5-byte code of send() and recv() are changed respectively.
     */
    rv = duckhook_install(duckhook, 0);
    if (rv != 0) {
       /* error */
       ...
    }
}

```

License
-------

GPLv2 or later with a [GPL linking exception][].

You can use Duckhook in any software as long as the software
doesn't forbid API hooking. However if you modify Duckhook
itself, the modifed part must be under the GPL with or without
the linking exception.

[GPL linking exception]: https://en.wikipedia.org/wiki/GPL_linking_exception
[diStorm3]: https://github.com/gdabah/distorm/
