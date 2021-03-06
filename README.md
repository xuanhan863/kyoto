Kyoto Tycoon
============

Kyoto Tycoon is a lightweight network server on top of the Kyoto Cabinet key-value database, built for high-performance and concurrency. Some of its features include:

  * _master-slave_ and _master-master_ replication
  * in-memory and persistent databases (with optional binary logging)
  * hash and tree-based database formats
  * server-side scripting ([Lua](http://www.lua.org/))

It has its own fully-featured [protocol](http://fallabs.com/kyototycoon/spex.html#protocol) based on HTTP and a (limited) binary protocol for even better performance. There are several client libraries implementing them for multiple languages (we're maintaining one for Python [here](https://github.com/sapo/python-kyototycoon)).

It also supports the [memcached](http://www.memcached.org/) protocol with some [limitations](http://fallabs.com/kyototycoon/spex.html#tips_pluggableserver) (eg. "prepend", "append" and "cas" are missing). This is useful if you wish to replace _memcached_ in larger-than-memory/persistency scenarios.

What's this fork?
-----------------

The development of Kyoto Tycoon and Kyoto Cabinet by the original authors seem to have halted around 2012. The software works as advertised and is very reliable, which may explain the lack of activity, but the upstream sources fail to build in recent operating system releases (with recent compilers).

This repository isn't really meant as an "hostile" fork for any of these packages, but as a place to keep readily usable versions for modern machines. Nevertheless, pull requests are welcome.

What's included?
----------------

Here you can find the latest upstream releases with additional modifications, intended to be used together. The changes include patches sourced from Linux distribution packages and some custom patches which have been tested in real-world production environments. Check the commit history for more information.

Installing
----------

To build and install everything in one go, run:

    $ make PREFIX=/custom/install/root
    $ sudo make install

Specifying the installation root directory (`PREFIX`) is optional. By default it installs into `/usr/local`.

Running
-------

If there's a place in need of improvement it's the documentation for the available server options in Kyoto Tycoon. Make sure to check the [command-line reference](http://fallabs.com/kyototycoon/command.html#ktserver) to understand what each option means and how it affects performance vs. data protection. But you may want to try this as a quick start for realistic use:

    $ mkdir -p /data/kyoto/db
    $ /usr/local/bin/ktserver -ls -th 16 -port 1978 -pid /data/kyoto/kyoto.pid \
                              -log /data/kyoto/ktserver.log -oat -uasi 10 -asi 10 -ash \
                              -sid 1001 -ulog /data/kyoto/db -ulim 104857600 \
                              '/data/kyoto/db/db.kct#opts=l#bnum=100000#msiz=256m#dfunit=8'

This will start a standalone server using a persistent B-tree database with binary logging enabled. The immediately tweakable bits are `bnum=100000`, which roughly optimizes for (but doesn't limit to) 100 thousand stored keys, and `msiz=256m`, which limits memory usage to around 256MB.

Another example, this time for an in-memory cache hash database limited to 256MB of stored data:

    $ /usr/local/bin/ktserver -log /var/log/ktserver.log -ls '*#bnum=100000#capsiz=256m'

To enable the _memcached_ protocol, use the `-plsv` and `-plex` parameters. The previous example would then become:

    $ /usr/local/bin/ktserver -log /var/log/ktserver.log -ls \
                              -plsv /usr/local/libexec/ktplugservmemc.so \
                              -plex 'port=11211#opts=f' \
                              '*#bnum=100000#capsiz=256m'

Automatic object removal in databases using the `capsiz` option is LRU-based. Based on our experience, we don't recommend using this option with persistent (on-disk) databases as the server will temporarily stop responding to free up space when the maximum capacity is reached. In this case, try to keep the database size under control using key expiration instead.

References
----------

  * http://fallabs.com/kyototycoon/
  * http://fallabs.com/kyotocabinet/
