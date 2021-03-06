# Introduction

This module is a part of Imagine'20 Cloud Docker MFTF Lab session.

## Terms
* [Docker](https://www.docker.com): A software that performs operating-system-level virtualization
* [Mutagen](https://mutagen.io): A real-time file synchronization and flexible network forwarding, extending the reach of your existing development tools to cloud-based containers and infrastructure

# Related repositories

* [ECE-Tools](https://github.com/magento/ece-tools): A deployment tool to deploy Magento 2 on Cloud
* [Cloud Template](https://github.com/magento/magento-cloud): A set of Magento Cloud templates for different Magento 2 versions
* [Cloud Docker repository](https://github.com/magento/magento-cloud-docker): A source files for Cloud Docker images
* [Cloud Docker images](https://cloud.docker.com/u/magento): A set of pre-build images on Docker Hub

# Environment setup

Follow instruction on [DevDocs](https://devdocs.magento.com/guides/v2.3/cloud/docker/docker-development.html)

### MFTF dependency

**If** you don't have MFTF dependency added:

```bash
composer require "magento/magento2-functional-testing-framework" --no-update && composer update
```

## Production mode

<details><summary>Toggle</summary>

### Build docker-compose.yml

```bash
./vendor/bin/ece-docker build:compose --with-selenium
```

### Start containers

```bash
./bin/magento-docker up
```

### Deploy Magento

```bash
./bin/magento-docker ece-redeploy
```

### [Continue MFTF setup](#prepare-magento)
</details>

## Developer mode

<details><summary>Toggle</summary>

### Add dependencies

```bash
composer config repositories.demo vcs https://github.com/shiftedreality/module-imagine-docker-demo-2020
composer config minimum-stability dev
composer require "magento/module-demo" --no-update
```

### Build docker-compose.yml

```bash
./vendor/bin/ece-docker build:compose --mode developer --with-selenium --sync-engine mutagen
```

### Start containers

```bash
./bin/magento-docker up
```

### Start Mutagen

```bash
./mutagen.sh
```

This step takes some time. To verify the status, run:

```bash
mutagen monitor
```

### Deploy Magento

```bash
./bin/magento-docker ece-redeploy
```

**OR** if Magento was previosuly compiled and patched:

```bash
./bin/magento-docker ece-deploy
```

### Set Developer Mode

```bash
docker-compose run --rm deploy magento-command deploy:mode:set developer
```

</details>

## Prepare Magento

```bash
docker-compose run --rm test magento-command config:set system/full_page_cache/caching_application 2 --lock-env
docker-compose run --rm test magento-command setup:config:set --http-cache-hosts=varnish -n
docker-compose run --rm test magento-command config:set admin/security/admin_account_sharing 1
docker-compose run --rm test magento-command config:set admin/security/use_form_key 0
docker-compose run --rm test magento-command config:set web/secure/use_in_adminhtml 0
docker-compose run --rm test magento-command cache:clean
```

## Open Magento

* https://magento2.docker

## Prepare MFTF tests

### Prepare configs

```bash
CONFIG="MAGENTO_BASE_URL=http://magento2.docker/
MAGENTO_BACKEND_NAME=admin
MAGENTO_ADMIN_USERNAME=admin
MAGENTO_ADMIN_PASSWORD=123123q
MODULE_WHITELIST=Magento_Framework,Magento_ConfigurableProductWishlist,Magento_ConfigurableProductCatalogSearch
SELENIUM_HOST=selenium"

docker-compose run --rm test bash -c "echo \"$CONFIG\" > /app/dev/tests/acceptance/.env"
```

### Build artifacts

```bash
docker-compose run --rm test mftf-command build:project
docker-compose run --rm test mftf-command generate:tests --debug=none
```

## Run Tests

```bash
docker-compose run --rm test mftf-command run:test AdminLoginTest --debug=none
```

**IF** you added a Demo moudle in *Developer* mode:

```bash
docker-compose run --rm test mftf-command run:test OpenStorefrontDemoPageTest --debug=non
```
