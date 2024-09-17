+++
title = "cmpct IRCd"
description = "An internet relay chat server"
weight = 900

[extra]
local_image = "/portfolio/cmpct-1.png"
+++

- [GitHub project](https://github.com/cmpct/cmpctircd)

I like chatting to friends. Speaking of friends and chatting, I helped develop a server with my friends to chat to them!

[IRC](https://en.wikipedia.org/wiki/IRC), or internet relay chat, is a pretty old text-based protocol for online chats. It was created in 1988 and standardized in 1993 under [RFC 1459](https://datatracker.ietf.org/doc/html/rfc1459). While in declining use, us nerds really liked to use it and it's a pretty simple protocol to implement.

Ironically, we tend to use Discord now and we haven't maintained the server for a few years. A lot of us are too busy to be at a computer now and the lack of push notifications from IRC doesn't work very well with that. Still, it was a really great way to teach each other. I'd never really needed to use pull requests until then.

One of my best moments was locating and [fixing](https://github.com/cmpct/cmpctircd/commit/248b67cd96ac7765375cc47b001185a702f9246f) a really horrible bug which would cause the server to lock up seemingly at random. It turned out the TLS handshake was blocking the main thread and would do so for a very long time if not completed.