# transmission-4-centos-7

Files to support installation of the Transmission 4.0 client on CentOS 7.

## Repository contents

This directory contains:

- An Ansible task/playbook to compile and install Transmission 4.0 on CentOS 7

- A systemd unit file suitable for running the Transmission 4.0 daemon

- logrotate, sysconfig, and tmpfiles.d configuration files for the Transmission 4.0 daemon

## The Ansible task

The [transmission4-installation.yaml](transmission4-installation.yaml) 
file is an Ansible task that will build and install Transmission 4.0 from source on CentOS 7. It can be included
from your playbook, or run as its own independent playbook. Configuration and usage are described below.

### Configuring the Ansible task

Set the options in the "General configuration, "Transmission configuration," and "cURL configuration" sections to your
liking. The default values should work for most cases. If you `include` the task from within your own playbook, you can
define any of the following variables there to override the task defaults:

- `build_directory` - a temporary directory for downloading and compiling files
- `do_cleanup` - whether or not to automatically delete build artifacts after successful installation
- `transmission_version` - the target transmission version e.g. '4.0.0'
- `trasmission_sha256` - sha256 of the .tar.xz distribution of the target transmission version
- `transmission_configure_options` - CMake3 options for configuring Transmission
- `curl_configure_options` - configure options for cURL

### Running the Ansible task

You can either `ansible.builtin.include` the .yaml file from your own playbook, or use it as a stand-alone 
playbook by uncommenting a few lines and specifying your own `hosts:` value.

Running the task performs the following actions:

- Create a temporary build directory (by default, `/tmp/transmission4`)

- Check to see if cURL 7.77.0 or newer is present; if not, download and install a temporary copy of the latest cURL
release in the build directory. Transmission 4 requires libcurl features not available in cURL < 7.77.0.

- Install several prerequisites (via yum) required for the build process: cmake3, gettext, intltool, m4, make, 
openssl-devel, perl-XML-Parser, pkgconfig

- Temporarily install (via yum) the Red Hat Developer Toolset 7.0. CentOS 7 has gcc-c++ version 4.8.5, which is not 
sufficient to compile Transmission 4.0. The devtools-7 package provides gcc-c++ 7.3.1.

- Download, compile, and install the specified Transmission version

- Add a `transmission` user and group to the system, if they don't already exist

- Create `/var/lib/transmission` where the daemon will write its files

- Install a systemd unit file, plus logrotate, sysconfig, and tmpfiles.d configuration files for Transmission; those
files are all downloaded from this repository

- After successful installation, the Red Hat Developer Toolset 7.0 will be uninstalled, and the temporary build
directory will be removed. You can disable this by setting `do_cleanup` to false.

**The following will be added to your system if they don't already exist:**

- a "transmission" user and group, which will run the transmission daemon
- /etc/logrotate.d/transmission4-daemon
- /etc/sysconfig/transmission4-daemon
- /etc/systemd/system/transmission4-daemon.service
- /etc/tmpfiles.d/transmission4-daemon.conf
- /var/lib/transmission
- /var/log/transmission

## Notes and caveats

The Ansible task installs a systemd unit file, but it doesn't automatically start or enable the service. You'll need
to do that yourself with `systemctl start transmission4-daemon` and `systemctl enable transmission4-daemon`.

A `settings.json` file is *not* installed. If you have a `settings.json` file you want to use, place it into 
`/var/lib/transmission` prior to starting the transmission4-daemon service. Otherwise, Transmission will create a default
settings file the first time it runs.

I only use the Transmission daemon and the web interface, so that's all I've tested. Building the Qt client is
configured to "AUTO" by default, which I suppose will compile the GUI if you're running in a desktop environment. You'll
at least end up with the CLI binaries: `transmission-cli`, `transmission-create`, `transmission-daemon`, `transmission-edit`,
`transmission-remote`, and `transmission-show`.

The Transmission 4.0 binaries will be installed to `/usr/local/bin`. If you already have Transmission 3.0 from
the excellent [Geekery repository](http://geekery.altervista.org/dokuwiki/doku.php) -- which installs its binaries to
`/usr/bin` instead -- you'll end up with both versions of Transmission installed. This is intentional. Running the
Ansible task *shouldn't* clobber or overwrite any existing files, and the system files it installs are all named
using a `transmission4` scheme to keep them separate from Transmission 3.0.

## Bugs

If you encounter a problem with the Ansible task, please [open a new issue](https://github.com/parseword/transmission-4-centos-7/issues/new) in this repository.
