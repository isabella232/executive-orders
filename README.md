# executive-orders

## What is this?

**Executive Orders is a cron job that posts to a Slack webhook whenever new a new document is published on [whitehouse.gov](https://www.whitehouse.gov/briefing-room/presidential-actions).**

## Assumptions

The following things are assumed to be true in this documentation.

* You are running OSX.
* You are using Python 3. (Probably the version that came OSX.)
* You have [virtualenv](https://pypi.python.org/pypi/virtualenv) and [virtualenvwrapper](https://pypi.python.org/pypi/virtualenvwrapper) installed and working.

For more details our stack, see our [development environment blog post](http://blog.apps.npr.org/2013/06/06/how-to-setup-a-developers-environment.html).

## What's in here?

The project contains the following folders and important files:

* ``fabfile`` -- [Fabric](http://docs.fabfile.org/en/latest/) commands for automating setup, deployment, data processing, etc.
* ``app_config.py`` -- Global project configuration for scripts, deployment, etc.
* ``crontab`` -- Cron jobs to be installed as part of the project.
* ``requirements.txt`` -- Python requirements.
* ``run_on_server.sh`` -- Shell script to run at the beginning of cron jobs to source environment variables and virtual environments.

## Bootstrap the project

```
cd executive-orders
mkvirtualenv -p `which python3` executive-orders
pip install -r requirements.txt
```

## Hide project secrets

Project secrets should **never** be stored in ``app_config.py`` or anywhere else in the repository. They will be leaked to the client if you do. Instead, always store passwords, keys, etc. in environment variables and document that they are needed here in the README.

Any environment variable that starts with ``$PROJECT_SLUG_`` will be automatically loaded when ``app_config.get_secrets()`` is called.

## Connecting to Slack

To connect Clerk to your Slack, you will need to create an [incoming webhook](https://api.slack.com/incoming-webhooks) and copy the webhook endpoint to an environment variable called `clerk_WEBHOOK`. The default server setup expects the environment variable to be placed in `/etc/environment`.

## Setting up servers

Use the server configuration in [app_config.py](https://github.com/nprapps/clerk/blob/master/app_config.py#L36-L46) to set up your servers correctly. You can set the user, python version, and the folders where various pieces install.

## Deploy to EC2

1. Run ``fab staging master servers.setup`` to configure the server.
2. Run ``fab staging master deploy`` to deploy the project to the server. Note that this will also install your crontab.

## Managing cron jobs

Cron jobs are defined in the file `crontab`. Each task should use the `run_on_server.sh` shim to ensure the project's virtualenv is properly activated prior to execution. For example:

```
* * * * * ubuntu bash /home/ubuntu/apps/executive_orders/repository/cron.sh fab $DEPLOYMENT_TARGET cron_jobs.test >> /var/log/executive_orders/crontab.log 2>&1
```

The cron jobs themselves should be defined in `fabfile/cron_jobs.py` whenever possible.

To install the cronjob, run either `fab staging master deploy`, which will also synchronize the repo with the latest on Github, or `fab staging master servers.install_crontab` to simply copy the crontab on the server to `etc/cron.d`.

To uninstall the cronjob (a recommended practice for disabling the job), run `fab staging master servers.uninstall_cronjob`.







