Here is a description how to make Ocsigen applications (more) static
in order to ease deployment of the application. This is done on Linux
(Ubuntu). Not sure if it will work on other platforms.

## Background
One of the challenges I have found with
[Ocsigen](https://ocsigen.org/home/intro.html) application has been
that I have ha to copy parts of the development environment to the
target nodes.

Ocsigen supports dynamic loading of both library modules and your
application. For example, the example code in
[ocsigen-start](https://ocsigen.org/ocsigen-start/latest/manual/intro)
demo relies on dynamic linking. This is fine and easy to work with
when developing, but gives challenges when the application is to be
deployed to other nodes.

In addition, Ocsigen supports [static
linking](https://ocsigen.org/ocsigenserver/latest/manual/staticlink),
however, I have [not managed to get it fully
working](https://sympa.inria.fr/sympa/arc/ocsigen/2018-06/msg00000.html).
In particular I have problems with delaying registration of service
when using RPC.

Here is a description how to make Ocsigen application
"semi-static". That is, the majority of modules used in the
application is statically linked with the server, while the
application (and a few modules that I was unable to statically link)
are linked dynamically when the server starts.

## Example on making "ocsigen-start" demo application "semi-static"
Create and build the demo application as described in the
[documentation](http://ocsigen.org/ocsigen-start/latest/manual/intro).

### Configuration for static linked modules
Make a copy of the configuration file
```shell
myproject$ cp local/etc/myproject/myproject-test.conf local/etc/myproject/myproject-static.conf
```

Change the file so that it initializes the modules that is part of the
server binary. An example of a modified version is in
[local/etc/myproject](local/etc/myproject/).

To find the name used for modules, you might serch for
`Ocsigen_extensions.register` is source code. I'm not sure it is a way
to list the name of registered extensions.

### Static linked server
For each module that used to be linked dynamically, make sure that
they are linked into the server.

In [server](server/) is a project that will build a "ocsigenserver"
with most of the modules linked in. 

```shell
deploying_ocsigen/server$ dune build
```

Copy the binary to the project folder.
```shell
myproject$ mkdir bin
myproject$ cp ../deploying_ocsigen/server/_build/default/bin/ocsigen_semi_static_server.exe bin/
```

### Modules that need to be loaded dynamically
The ocsigen-toolkit.server and ocsigen-start.server modules can not be
linked with the binary. The libraries are missing ".a" files.

So we need to put the modules as part of our release, and load those dynamically.

```shell
myproject$ l=$(ocamlfind query ocsigen-start.server)  ;\
cp $l/*.cmxs local/lib/myproject/ ;
l=$(ocamlfind query ocsigen-toolkit.server) ;\
cp $l/*.cmxs local/lib/myproject/
```
The `*.cmxs` files get loaded trough the 

```
    <host hostfilter="*">
        ...
      <eliommodule module="local/lib/myproject/ocsigen-toolkit.server.cmxs" />
      <eliommodule module="local/lib/myproject/ocsigen-start.server.cmxs" />
      ...
```

### Findlib
For some reason, the server seems to need `findlib.conf`. Might be a
way to disable this, but the quickest solution is joust to create a
dummy file for it.

```shell
myproject$ mkdir lib
myproject$ touch lib/findlib.conf
```

### Packing the demo application
```shell
myproject$ cd ..
$ tar zcfv myproject.tgz myproject/bin/ myproject/lib myproject/local myproject/local_db/
```

### Running application
Copy the tar file to the target node and unpack it.

```shell
othernode:myproject$ export OCAMLFIND_CONF=./lib/findlib.conf; \
  ./bin/ocsigen_semi_static_server.exe -c local/etc/myproject/myproject-static.conf
```

This is how far I got with making static Ocsigen applications.
Hopefully it might be useful for other that needs to deploy
applications written using Ocsigen. If you have tips on how to improve
it and make it more static, please let me know.
