docker-bluster
==============

Docker image for bluster.

This docker image provides a bluster (https://github.com/rafaelmartins/bluster) application. The easiest way to deploy bluster!

This image does not installs a full web stack. It just runs bluster as a FastCGI application, listening to port 8080. You must redirect this port to a host port and use nginx (or any other FastCGI capable webserver) there to handle the requests and forward them back to the FastCGI application running in this docker image.

To create a new container you need to fetch this image from the Docker hub:

    $ docker pull rafaelmartins/bluster

Now you can create the container:

    $ docker run -d \
         --name my-bluster-app \
         -p 8888:8080 \
         -e BLUSTER_GIST_ID=xxxxxxxxxxxxxxxxxx \
         -e BLUSTER_GIST_TTL=10 \
         -e BLUSTER_OAUTH_TOKEN=xxxxxxxxxxxxxxxxx \
         rafaelmartins/bluster

Where `8888` is the host port that will be redirected to the container's `8080` port, `BLUSTER_GIST_ID` is the Github Gist identifier (the hex string at the end of the Gist URL), `BLUSTER_GIST_TTL` is the life time of the Gist content cache in minutes (defaults to 5 minutes) and `BLUSTER_OAUTH_TOKEN` is a GitHub oauth2 token with `gist` scope.

You can create the GitHub oauth2 token here: https://github.com/settings/tokens/new

Make sure to set `--name` properly, it is the identifier of the container, that will be used to start/stop/reload it!


## How to setup nginx

If you keep the port settings from the previous example, you'll have the FastCGI application from the docker container listening to port 8888 in the host. The following `location` blocks are enough to serve this application. Add one of them to your `server` block, depending on where you want to "mount" the FastCGI application:

    location / {
        fastcgi_pass 127.0.0.1:8888;
        fastcgi_split_path_info ^(/)(/.*)$;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        include fastcgi_params;
    }

    location /mybluster/ {
        fastcgi_pass 127.0.0.1:8888;
        fastcgi_split_path_info ^(/mybluster)(/.*)$;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        include fastcgi_params;
    }

Make sure that you have the `fastcgi_params` file in your nginx configuration directory (e.g. `/etc/nginx`) it is usually shipped with nginx.
