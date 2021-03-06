# analytics-nginx

## A specialised nginx container

This container is designed to serve HTTPS requests to the
`analytics.incisive.ly` domain, as illustrated:

```

      (internet)
          |
          | HTTPS request to
          | analytics.incisive.ly
          |
 ---------------------
|   analytics-nginx   |
 ---------------------
   |              |
   | GET /tags/*  | POST /collect/*
   |              |
  ---            ---
   A              B


A: static snippet file from /tags on the filesystem
B: proxy-pass to a backend service

```

You can run the container like so:

```sh
docker run -v /path/to/tags:/tags -v /path/to/ssl/files:/ssl -p 443:443 -d analytics-nginx
```

As you can see, this command expects two paths to be present on the host system:

1. `/path/to/tags`, which should contain the desired directory structure &
snippet code. For example, given this example path, in order to serve
`https://analytics.incisive.ly/tags/1234/iy.js`, the following file should
exist on the host system: `/path/to/tags/1234/iy.js`

2. `/path/to/ssl/files`, which should contain the `.crt` and `.key` files for
the SSL certificate you want to use.


## Zero-downtime configuration changes from a remote container

The `/etc/nginx` volume can be mounted by another container. This allows access
to all of nginx's configuration files (including virtual hosts), plus an
additional special file: `/etc/nginx/reload.sock`, which is a Unix socket.
When you write to `reload.sock`, nginx will reload, picking up any
configuration changes.

### Example:

Given a running `analytics-nginx` instance named `distracted_wozniak`:

1. `docker run --volumes-from distracted_wozniak --rm -t -i ubuntu:precise bash`
2. Make your config changes, e.g. create a new nginx host file
   `/etc/nginx/sites-enabled/trashbat.conf`
3. `nc -U /etc/nginx/reload.sock`

The `analytics-nginx` container has now picked up the new configuration, and we
can exit the container we created in these steps.


## boot2docker gotcha

If you're using boot2docker, remember that the host system is the boot2docker
VM, *not* your Mac!

## TODO:

- Store tags in another Docker container instead of the local filesystem
- Finish work to proxy-pass /capture to Analytics Service
  - should write a new vhost file to `/etc/nginx/sites-enabled/service.conf`
  and then send `SIGHUP` to reload nginx. (Use `EXPOSE /etc/nginx` for use by
  another container via `--volumes-from`.)
