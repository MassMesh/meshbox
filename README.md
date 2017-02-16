meshbox
=======

This is the LEDE package feed for the [cjdns][cjdns] routing protocol. It provides LEDE integration and a web-based UI.

![UI screenshot](https://github.com/bostonmeshnet/meshbox/blob/for-17.02%2Bterrier/screenshot.png)

[cjdns]: https://github.com/cjdelisle/cjdns


Building from Source
------------

Note: The package `luci-app-cjdns` is already available upstream via the `routing` feed. You may have to battle between the two feeds when doing `./scripts feeds install -a`

Building meshbox for development

    $ git clone git://git.lede-project.org/source.git lede-source
    $ cd lede-source
    $ git clone git@github.com:bostonmeshnet/meshbox.git meshbox

    $ echo "src-link meshbox $PWD/meshbox" | tee feeds.conf
    $ cat feeds.conf.default | tee -a feeds.conf

    $ ./scripts/feeds update -a
    $ ./scripts/feeds install -a


Building with git as a repo (repeatable builds of latest (or specific) git-revision)


    $ git clone git://git.lede-project.org/source.git lede-source
    $ cd lede-source

    $ cp feeds.conf.default feeds.conf
    $ echo 'src-git meshbox https://github.com/bostonmeshnet/meshbox.git;for-17.02+terrier' >> feeds.conf
    $ ./scripts/feeds update -a
    $ ./scripts/feeds install -a

Then configure your firmware image: enable the luci-app-cjdns module, in addition to your usual settings, such as target system and profile. As usual, you'll need to hit space twice to make it `[*]` rather than `[M]`.

    $ make menuconfig
    LuCI -> Collections -> [*] luci
    LuCI -> Applications -> [*] luci-app-cjdns

Then build with `make`. You can append `-j $n`, where n is the number of CPU threads you want to use for compilation.


Contact
-------

- Issue tracker: [github.com/bostonmeshnet/meshbox/issues](https://github.com/bostonmeshnet/meshbox/issues)
- IRC: #BostonMeshnet on Freenode
- Mailing list: [boston@lists.projectmesh.net](https://lists.projectmesh.net/mailman/listinfo/boston)
- Development updates: [bostonmeshnet.github.io](https://bostonmeshnet.github.io/)


Development
-----------

Almost all of the development can be conducted using only a Docker container.

```
$ docker run -i -t lgierth/meshbox /sbin/init
> printf "12345\n12345\n" | passwd
> ifconfig eth0 | grep 'inet addr'
>
```

Then you can code away, and deploy the changed files as needed.

```
$ ./deploy.sh root@ADDRESS 12345
```

This will deploy `cjdns/files`, `cjdns/lua`, and `luci-app-cjdns/luasrc` to the appropriate directories in the container. If your changes require a restart, or the changed code is only run at boot time, you'll need to build your own image.

```
$ cd lede-source/
$ vim feeds.conf # src-git meshbox ... => src-link meshbox /path/to/meshbox
$ ./scripts/feeds update -a
$ ./feeds/meshbox/docker.sh
$ docker run -i -t meshbox /sbin/init
```

In case you want to make changes to cjdns itself, you can modify `<meshbox>/cjdns/Makefile` to use a local clone of cjdns.

```
PKG_SOURCE_URL:=file:///path/to/cjdns
PKG_SOURCE_PROTO:=git
PKG_SOURCE_VERSION:=master
```

Make sure to commit your changes to cjdns before building the package. The LEDE buildroot will clone the local cjdns into the build directory, omitting uncommitted changes.

You can then build a fresh container including the changes to cjdns.
