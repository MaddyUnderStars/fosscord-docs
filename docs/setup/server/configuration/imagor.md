# Imagor

[Imagor](https://github.com/cshum/imagor) is a "fast, secure image processing server"
used by Fosscord for external image resizing, primarily by embeds from other websites when linked in a message.
If left unused, Fosscord will simply not provide a proxy_url for clients, which may leave external images unavailable
or cause the client to download directly from the image host. Downloading images directly from the host may lead to
privacy concerns, as an attacker may be able to learn users IP addresses.

## Dependencies

-   [Docker](https://www.docker.com/)

## Setup

To setup Imagor for Fosscord, first grab the `security_requestSignature` config value from Fosscord's database,
and create a `imagor.env` file somewhere safe, with the following content (do not include the `[]`)

```
IMAGOR_SECRET=[security_requestSignature value]
PORT=8000
```

You can now start Imagor with

```bash
docker run --env-file ./imagor.env -p 8000:8000 shumc/imagor
```

`8000` here is our port. Make sure that it'd available to people outside your network.
If you would like to change the port Imagor listens on, be sure to change both the PORT value in `imagor.env`,
and the `-p` value used in docker.

If you're using a [reverse proxy](../reverseProxy.md) such as Nginx for Fosscord already, you could add this to your config's `server` block

```nginx
location /media {
	# If you changed the port, be sure to change it here too
	proxy_pass http://127.0.0.1:8000;
}
```

Along with any additional config you already have, of course.
Alternative (and perhaps the better choice) would be to create a new domain, say `media.whatever.com` specifically for Imagor.

??? "Example config for `media.whatever.com` site"

    ```nginx
    server {
    	# Change the server_name to reflect your true domain
        server_name media.whatever.com

        add_header Last-Modified $date_gmt;
        proxy_set_header Host $host;
        proxy_pass_request_headers on;
        proxy_set_header X-Forwarded-For $remote_addr;

        location / {
    			# If you had changed the port, change it here as well
                proxy_pass http://127.0.0.1:8000;
        }
    }
    ```

Our last step is to simply tell Fosscord about Imagor. Just set the `cdn_imagorServerUrl` config value to your public endpoint for Imagor, wrapped in quotes.

Congrats! After a restart, you've now got Imagor resizing your images!
