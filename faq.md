# FAQ

<style>
/* Style like H3 */
summary {
    margin-bottom: 1rem;
    line-height: 1.25;
    font-size: 1.25em;
    /* font-weight: 600; */
}
</style>

In code examples, you might stumble across such declaration:

```bash
# to clone the fork
git clone git@github.com:<YOUR GITHUB USERNAME>/scandipwa-base.git
```

> **Note**: the `<YOUR GITHUB USERNAME>` is not a literal text to keep, but the "template" to replace with the real value.

<details>
<summary>The <code>elasticsearch</code> is not working</summary>

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

</details>

<details>
<summary>The nginx can not find <code>varnish</code> host</summary>

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
</details>

<details>
<summary>The <code>varnish</code> container is not working</summary>

Is a source of following problems:

1. Site responds with `503 backend fetch failed`

Execute following commands:

```bash
# if you have the alias set up
dc restart varnish

# without aliases (not recommended)
docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml restart varnish
```
</details>

<details>
<summary>The <code>app</code> container is not working</summary>

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

Wait for output above, afterwards, the `502` error should be resolved.

If the app is ready to handle connections, but the site still respond with `502`, you might want to look into `app` logs a little deeper. Execute:

```bash
docker-compose logs -f app
```

Scroll those logs to the very top and see if any `error` appears. If it does, search for this error mentions in this FAQ. If there are no error, execute the same instructions as in the **The nginx can not find `varnish` host** FAQ section.

</details>
<details>
<summary>The <code>composer</code> related issues</summary>

Inspect the `app` container logs, using following command:

```bash
# if you have the alias set up
applogs

# without aliases (not recommended)
docker-compose logs -f --tail=100 app
```

If you find the following error in the logs:

```bash
Please set COMPOSER_AUTH environment variable
```

Make sure you have a valid Magento 2 `COMPOSER_AUTH` set. This is an environment variable set on your host machine. To test if it is set, use:

```bash
env | grep COMPOSER_AUTH
```

If the output of this command is empty, or, if the output (JSON object) does not contain `"repo.magento.com"` key, you need to set / update the environment variable.

   1. Make sure you have a valid Magento account. You can [create](https://account.magento.com/applications/customer/create/) or [login to existing one](https://account.magento.com/applications/customer/login/) on Magento Marketplace site.

   2. Upon logging to your Magento Marketplace account follow the [official guide](https://devdocs.magento.com/guides/v2.3/install-gde/prereq/connect-auth.html) to locate and generate credentials.

   3. Now, using the following template, set the environment variable:

       ```bash
       export COMPOSER_AUTH='{"http-basic":{"repo.magento.com": {"username": "<PUBLIC KEY FROM MAGENTO MARKETPLACE>", "password": "<PRIVATE KEY FROM MAGENTO MARKETPLACE>"}}}'
       ```

       To set the environment variables follow [this guide](https://www.serverlab.ca/tutorials/linux/administration-linux/how-to-set-environment-variables-in-linux/). Make sure to make them persist (stay between reloads).

If upon ispection you see a different error:

```bash
COMPOSER_AUTH environment variable is malformed, should be a valid JSON object
```

Check if the environment variable is set properly, it must be valid JSON object.

This issue is common with AWS ECS setups. If you happened to use one, make sure to set it in the folowing way (without quotes):

```json
{
    "name": "COMPOSER_AUTH",
    "value": "{\"http-basic\":{\"repo.magento.com\": {\"username\": \"<PUBLIC KEY FROM MAGENTO MARKETPLACE>\", \"password\": \"<PRIVATE KEY FROM MAGENTO MARKETPLACE>\"}}}"
}
```

If the different, `Invalid credentials ...` error appears, like this, for example:

```bash
# the general one, like this:
Invalid credentials for 'https://repo.magento.com/packages.json', aborting

# the more specific one, like this:
The 'https://repo.magento.com/archives/magento/framework/magento-framework-102.0.3.0.zip' URL required authentication.
```

This indicates on:

- issue with credentials, try obtaining new ones from Magento Marketplace.

- the `COMPOSER_AUTH` might be valid JSON, but missing the `"repo.magento.com"` key in it. Again, refer to the instruction above to obtain tokens.

</details>
