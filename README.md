PgBouncer
=========

Lightweight connection pooler for PostgreSQL.

Homepage: <https://www.pgbouncer.org/>

Sources, bug tracking: <https://github.com/pgbouncer/pgbouncer>

Building
---------

PgBouncer depends on few things to get compiled:

* [GNU Make] 3.81+
* [Libevent] 2.0+
* [pkg-config]
* [OpenSSL] 1.0.1+ for TLS support
* (optional) [c-ares] as alternative to Libevent's evdns
* (optional) PAM libraries
* (optional) GSSAPI libraries

[GNU Make]: https://www.gnu.org/software/make/
[Libevent]: http://libevent.org/
[pkg-config]: https://www.freedesktop.org/wiki/Software/pkg-config/
[OpenSSL]: https://www.openssl.org/
[c-ares]: http://c-ares.haxx.se/

When dependencies are installed just run:

    $ ./configure --prefix=/usr/local
    $ make
    $ make install

If you are building from Git, or are building for Windows, please see
separate build instructions below.

DNS lookup support
------------------

PgBouncer does host name lookups at connect time instead of just once
at configuration load time.  This requires an asynchronous DNS
implementation.  The following table shows supported backends and
their probing order:

| backend                    | parallel | EDNS0 (1) | /etc/hosts | SOA lookup (2) | note                                  |
|----------------------------|----------|-----------|------------|----------------|---------------------------------------|
| c-ares                     | yes      | yes       | yes        | yes            | IPv6+CNAME buggy in <=1.10            |
| udns                       | yes      | yes       | no         | yes            | IPv4 only                             |
| evdns, libevent 2.x        | yes      | no        | yes        | no             | does not check /etc/hosts updates     |
| getaddrinfo_a, glibc 2.9+  | yes      | yes (3)   | yes        | no             | N/A on non-glibc                      |
| getaddrinfo, libc          | no       | yes (3)   | yes        | no             | N/A on Windows, requires pthreads     |

1. EDNS0 is required to have more than 8 addresses behind one host name.
2. SOA lookup is needed to re-check host names on zone serial change.
3. To enable EDNS0, add `options edns0` to `/etc/resolv.conf`.

c-ares is the most fully-featured implementation and is recommended
for most uses and binary packaging (if a sufficiently new version is
available).  Libevent's built-in evdns is also suitable for many uses,
with the listed restrictions.  The other backends are mostly legacy
options at this point and don't receive much testing anymore.

By default, c-ares is used if it can be found.  Its use can be forced
with `configure --with-cares` or disabled with `--without-cares`.  If
c-ares is not used (not found or disabled), then specify `--with-udns`
to pick udns, else Libevent is used.  Specify `--disable-evdns` to
disable the use of Libevent's evdns and fall back to a libc-based
implementation.

PAM authentication
------------------

To enable PAM authentication, `./configure` has a flag `--with-pam`
(default value is no).  When compiled with PAM support, a new global
authentication type `pam` is available to validate users through PAM.

GSSAPI authentication
---------------------

To enable GSSAPI authentication, `./configure` has a flag `--with-gss`
(default value is no).  When compiled with GSSAPI support a new global
authentication type 'gss' is available to validate users through GSSAPI
and Kerberos. This require a correctly configured Kerberized environment.

To enable GSSAPI in run-time the `auth_type` should be set to `gss` and
the `krb_server_keyfile` should be assigned the full path to the service
keytab (usually the 'postgres' service). It can be accessed using
PGBouncer directly, given that the process user has read access to that
file, or if GSSProxy is configured properly it should be able to protect
against privilege escalation.

This authentication does not delegate the user identity to PostgreSQL, but
if the connections between PGBouncer and PostgreSQL are set-up in safe
enough way a trust relationship could enable an end-to-end user identity
propagation.

Building from Git
-----------------

Building PgBouncer from Git requires that you fetch the libusual
submodule and generate the header and configuration files before
you can run `configure`:

	$ git clone https://github.com/pgbouncer/pgbouncer.git
	$ cd pgbouncer
	$ git submodule init
	$ git submodule update
	$ ./autogen.sh
	$ ./configure ...
	$ make
	$ make install

Additional packages required: autoconf, automake, libtool, pandoc

Building on Windows
-------------------

The only supported build environment on Windows is MinGW.  Cygwin and
Visual $ANYTHING are not supported.

To build on MinGW, do the usual:

	$ ./configure ...
	$ make

If cross-compiling from Unix:

	$ ./configure --host=i586-mingw32msvc ...

Running on Windows
------------------

Running from the command line goes as usual, except that the `-d` (daemonize),
`-R` (reboot), and `-u` (switch user) switches will not work.

To run PgBouncer as a Windows service, you need to configure the
`service_name` parameter to set name for service.  Then:

	$ pgbouncer -regservice config.ini

To uninstall service:

	$ pgbouncer -unregservice config.ini

To use the Windows event log, set `syslog = 1` in the configuration file.
But before that you need to register `pgbevent.dll`:

	$ regsvr32 pgbevent.dll

To unregister it, do:

	$ regsvr32 /u pgbevent.dll
