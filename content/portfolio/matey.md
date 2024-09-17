+++
title = "Matey"
description = "A container ingress auto configurator"
weight = 800

[extra]
local_image = "/portfolio/matey-1.png"
+++

- [GitHub project](https://github.com/getmatey/Matey)
- [Documentation](https://getmatey.github.io/)

While at my previous job, we started packaging more web applications in containers. Deploying containers was enough work, but then we found ourselves manually configuring IIS to forward to those containers.

This problem is largely solved using something like [traefik](https://doc.traefik.io/traefik/) but it isn't really compatible with how we were running our servers. We had many applications served directly by IIS and routing through to another web server was very different from how the servers had previously been managed.

That, and I thought it would be really interesting to try to separate the responsibility of configuring HTTP routing to containers and the actual routing already done by existing HTTP servers. So, while inspired by traefik, it instead uses container labels to configure a standalone HTTP server instead of serving its own.

I left the company before being production ready with it, but it became a very good exercise in technical writing. I also created a [WiX installer](https://github.com/getmatey/Matey/tree/master/Matey.Setup) for it.

The name comes from "arr matey," as reverse proxying in IIS is provided by [ARR](https://www.iis.net/downloads/microsoft/application-request-routing) and Docker generally has a nautical theme.
