# FAQ

###  Search is not working. I can not save the product in admin panel. The elasticsearch container is down.

The reason may be seen in the logs of application container, to see the logs, use:

```bash
docker-compose logs -f app
```

There you will see the message saying "indexer is not available". The **elasticsearch** is an indexer of Magento 2 (by our configuration). Make sure this container is up:

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

### 