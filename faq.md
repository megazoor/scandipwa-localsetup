# FAQ

## The `elasticsearch` is not working

Is a source of following problems:

1. Search is not working.
2. I can not save the product in admin panel.

The reason of problems above can be seen in the logs of application container, to see the logs, use:

```bash
docker-compose logs -f app
```

There you will see the message saying "indexer is not available". The **elasticsearch** is an indexer of Magento 2 (by our configuration). Make sure this container (the container will be named `scandipwa-base_elasticsearch_1`) is `up`:

```bash
docker-compose ps
```

If `elasticsearch` is showing `stopped` status, then it is down and must be restarted. But there must be a reason of elasticsearch stop. Check the logs of this container:

```bash
docker-compose logs -f elasticsearch
```

If you see an error log related to `max_map_count` value being to low, do following:

On your **host** machine, execute following command:

```bash
sudo sysctl -w vm.max_map_count=262144
```

> **Note**, to set this value permanently, follow [this guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html#vm-max-map-count).

After this, you can restart the `elasticsearch` container. To do it:

```bash
docker-compose restart elasticsearch
```

If for some reason issue persits, and the `elasticsearch` container keeps getting stopped after restart - you have a temporary option to switch the indexer itself.

To switch indexer, in Magento 2 admin, go to:
_Stores > Configuration > Catalog > Catalog > Catalog Search > Search Engine_ and set to `MySQL`.

> **Note**, after the next deploy, this value will be switched back to `elasticsearch` as this setting is set during the deploy.

### The nginx can not find `varnish` host

Is a source of following problems:

1. The site does not open at all

Execute following commands:

```bash
# if you have the alias set up
dc restart nginx ssl-term
dc restart varnish

# without aliases (not recommended)
docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml restart nginx ssl-term
docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml restart varnish
```

### The `varnish` container is not working

Is a source of following problems:

1. Site responds with `503 backend fetch failed`

Execute following commands:

```bash
# if you have the alias set up
dc restart varnish

# without aliases (not recommended)
docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml restart varnish
```

### The `app` container is not working

Is a source of following problems:

1. Site responds with `502 bad gateway`

First, check if the `app` container is `up`. You can do this by executing (look for the `scandipwa-base_app_1` container status):

```bash
docker-compose ps
```

If the `app` container is up, then you need to see the application logs to make a decision.

You can see when the application is ready to receive connections by watching `app` logs, using this command:

```bash
# if you have the alias set up
applogs

# without aliases (not recommended)
docker-compose logs -f --tail=100 app
```

If you can see following output, the application is ready!

```bash
NOTICE: ready to handle connections
```

Wait for this process to finish, afterwards, the `502` error should be resolved.

If the app is ready to handle connections, but the site still respond with `502`, you might want to look into `app` logs a little deeper. Execute:

```bash
docker-compose logs -f app
```

Scroll those logs to the very top and see if any `error` appears. If it does, search for this error mentions in this FAQ. If there are no error, execute the same instructions as in the **The nginx can not find `varnish` host** FAQ section.

