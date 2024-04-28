Here I describe how to make Ocsigen applications (more) static in
order to ease deployment of the application.

## Background
One of the challenges I have found with
[Ocsigen](https://ocsigen.org/home/intro.html) application has been
that I have ha to copy parts of the development environment to the
target nodes.

Ocsigen supports dynamic loading of both library modules and your
application. For example the example code in
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

In this post is a description how to make Ocsigen application
"semi-static". That is, the majority of modules used in the
application is statically linked with the server, while the
application (and a few modules that I was unable to statically link)
are linked dynamically when the server starts.

This is how far I got with making static Ocsigen applications.
Hopefully it might be useful for other that needs to deploy
applications written using Ocsigen. If you have tips on how to improve
it and make it more static, please let me know.

## Making "ocsigen-start" demo application "semi-static"


