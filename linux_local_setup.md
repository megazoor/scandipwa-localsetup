# Linux local setup

In code examples, you might stumble across such declaration:

```bash
# to clone the fork
git clone git@github.com:<YOUR GITHUB USERNAME>/scandipwa-base.git
```

> **Note**: the `<YOUR GITHUB USERNAME>` is not a literal text to keep, but the "template" to replace with the real value.

## Before you start

1. If you plan to make changes to the theme make sure to [create GitHub account](https://github.com/join), or [sign in](https://github.com/login) into existing one.

    1. Make sure you have a SSH key assigned to your GitHub account, and you current machine has the same key. Read [how to setup SSH key on GitHub](https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account).

    2. Make sure to fork [`scandipwa-base` repository](https://github.com/scandipwa/scandipwa-base). Read [how to fork repository on GitHub](https://help.github.com/en/github/getting-started-with-github/fork-a-repo).

2. Make sure you have `git` and `docker-compose` binaries installed. To test, execute following command in the bash terminal:

    ```bash
    git --version # it should be ^2
    docker-compose -v # it should be ^1.24
    ```

    - If `git` was not found, please follow [this installation instruction](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

    - If `docker-compose` was not found, please follow [this installation instruction](https://docs.docker.com/compose/install/).

3. Choose an installation directory. It can be anywhere on your computer. Folder `/var/www/public` is not necessary, prefer `~/Projects/` for ease of use.

4. Make sure the virtual memory `max map count` on your host machine is set high-enough. Follow [the instruction](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html#vm-max-map-count) to set it to appropriate level.

5. To make your life easier, make sure to create an aliases for docker-compose commands. Follow [the guide](https://www.tecmint.com/create-alias-in-linux/) to create **permanent** aliases. We recommend defining following:

    ```bash
    # use `dc` to start without `frontend` container
    alias dc="docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml"

    # use `dcf` to start with `frontend` container
    alias dcf="docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml -f docker-compose.frontend.yml"

    # use `inapp` to quickly get inside of the app container
    alias inapp="docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml -f docker-compose.frontend.yml exec -u user app"
    ```

    Those aliases are required to have all services available at all times. Otherwise, if just using `docker-compose` only services defined in `docker-composer.yml` will be available. Understand what services are available at all by reading [this part of our documentation](https://docs.scandipwa.com/#/docker/03-services?id=list-of-available-services).

6. Make sure you have a valid Magento 2 `COMPOSER_AUTH` set. This is an environment variable set on your host machine. To test if it is set, use:

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


7. Execute following command to add `scandipwa.local` to your `/etc/hosts` file and map it to the `127.0.0.1`:

    ```bash
    echo '127.0.0.1 scandipwa.local' | sudo tee -a /etc/hosts
    ```

## When you are ready

1. Get the copy of `scandipwa-base` - clone your fork, or clone the original repository. **Do not try to download the release ZIP - it will contain outdated code**.

    ```bash
    # to clone the fork
    git clone git@github.com:<YOUR GITHUB USERNAME>/scandipwa-base.git

    # to clone the original repository
    git clone git@github.com:scandipwa/scandipwa-base.git

    # to clone via HTTPS (not recommended)
    git clone https://github.com/scandipwa/scandipwa-base.git
    ```

    > **Note**: sometimes, after the repository is cloned, the git chooses the `master` branch as default. This is the legacy (incorrect) default branch in case of `scandipwa-base`. Please make sure you are using `2.x-stable`. You can do it using following command:

    ```bash
    git status # expected output `On branch 2.x-stable`
    ```

    If any other output has been returned, execute the following command to checkout the correct branch:

    ```bash
    git checkout 2.x-stable
    ```

2. Generate and trust a self-signed SSL certificate.

    1. Begin with generating a certificate. It will appear in `<PATH TO PROJECT ROOT>/opt/cert/scandipwa-ca.pem`.

        ```bash
        make cert
        ```

        > **Note**: you will be asked to type in the password 6 times. **This is NOT your root password**, this is a password for your certificate. This password must be at least 6 characters long.

    2. Add certificate to the list of trusted ones. Use this [guide](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html) (or [guide for Arch linux](https://bbs.archlinux.org/viewtopic.php?pid=1776753#p1776753)) to do it. The `new-root-certificate.crt` / `foo.crt` from these guide examples must be replaced with `<PATH TO PROJECT ROOT>/opt/cert/scandipwa-ca.pem`.

    3. Reload the Google Chrome. Sometimes, the Google Chrome caches the old-certificates. Make sure you have completely exited chrome, before opening it back. Sometimes, the "invalid certificate" issues only disappears after the full host machine reload.

3. Pull the images of all necessary containers

    ```bash
    # if you have the alias set up
    dcf pull

    # without aliases (not recommended)
    docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml pull
    ```

4. Start the infrastructure

    > **Note**: There are two ways to use the setup, with `frontend` container, and without it. The setup with `frontend` container is called **development**. The alias running it is `dcf` the alias for production-like run is `dc`.

    - For **production**-like setup:

        ```bash
        # if you have the alias set up
        dc up -d

        # without aliases (not recommended)
        docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml up -d
        ```

    - For **development** setup:

        ```bash
        # if you have the alias set up
        dcf up -d

        # without aliases (not recommended)
        docker-compose -f docker-compose.yml -f docker-compose.local.yml -f docker-compose.ssl.yml -f docker-compose.frontend.yml up -d
        ```
