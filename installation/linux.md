# Installation on Linux

## Using the Package Manager

Binary installation packages are provided for Ubuntu-based Linux distributions.
The latest Ubuntu LTS release is specifically supported.

The packages are held in the silentdynamics apt repository. Add it and install InsightCAE:

```bash
sudo apt-key adv --fetch-keys http://downloads.silentdynamics.de/SD_REPOSITORIES_PUBLIC_KEY.gpg
sudo add-apt-repository http://downloads.silentdynamics.de/ubuntu
sudo apt-get update
sudo apt-get install insightcae
```

After installation, continue with [Setting up the Environment](#setting-up-the-environment).

### Development Version

Besides the standard repository at `http://downloads.silentdynamics.de/ubuntu` (built from
the _master_ branch), a second repository provides packages built from the _next-release_
branch. To use it, replace the URL in the commands above with:

```
http://downloads.silentdynamics.de/ubuntu_dev
```

### Customer Packages

Customers who receive supported, tailored simulation apps from silentdynamics receive a
dedicated repository URL. The packages are named `insightcae-<customername>`:

```bash
sudo apt-get install insightcae-<customername>
```

## Setting up the Environment

To set up the environment variables and search paths required by InsightCAE, a script must
be sourced. Add the following line to the _beginning_ of your `~/.bashrc` file:

```bash
source /opt/insightcae/bin/insight_setenv.sh
```

Note that the default contents of `~/.bashrc` differ between Linux distributions. Some
distributions contain statements that prematurely end its evaluation for non-interactive
shells. The `source` line must be placed before any such early-exit statement.
