# Introduction

There are many ACME/letsencrypt clients out there, but none that quite
fits all my requirements :

* A central state directory to manage which domains needs a certificate,
that is itself manageable by tools like Ansible/Chef

* A single certificate may contain multiple domains, and each domain
may have a different validation method

* Which validation method to use is decided by hooks scripts, not by
configuration

* Either packaged in main distributions, or at least easily deployable

There is one client that fulfills all those requirements to the
perfection: [acmetool](https://github.com/hlandau/acme). Unfortunately,
I also need wildcard certificates *right now*, which acmetool does not
supports (yet).

pyacmetool is a quick-and-dirty reimplementation of a subset of acmetool
using the ACME library used by the official client. It is intended as
a transition tool, and will probably go unmaintained once acmetool will
support wildcard certificates.

# Usage

## Installation

pyacmetool has the following dependencies :

* Python 3 (tested with Python 3.5 ; >= 3.2 *may* work, but is untested)

* [python-acme](https://pypi.org/project/acme/) >= 0.28 (present in
Debian in the stretch-backports repository as python3-acme)

* [python-yaml](https://pypi.org/project/PyYAML/)

Install the dependencies, put `pyacmetool` somewhere in your path,
and you’re good to go.

## State directory

The state directory, by default, sits at `/var/lib/pyacme`. You can change
it with the `--state` argument or the `ACME_STATE_DIRECTORY` environment
variable. It roughly follows the same structure as the one by acmetool.

## Account creation

Get the lastest TOS :

<code>
pyacmetool tos -u https://acme-v02.api.letsencrypt.org/directory
</code>

Account creation :

<code>
pyacmetool init
    -u https://acme-v02.api.letsencrypt.org/directory
    -m contact@example.com
    -t TOS_URL
</code>

Beware: the default URL is the Letsencrypt *staging* endpoint.

## Usage

It works like acmetool. You put your desired domains in `desired/`,
and run `pyacmetool` or `pyacmetool reconcile` to populate `live/`
with the desired certificates.

To answer ACME challenges, you have only two methods at your hands :

* By configuring `request.challenge.webroot-paths` in `conf/target` (can
be done in the initialization step with the `-w` argument). `pyacmetool`
will reply to `http-01` challenges by putting the required file here.

* By creating hooks in `/etc/acme/hooks`. See the acmetool documentation
for that, `acmetool` hooks and `pyacmetool` hooks should be compatible
with each other, except for the `filename` argument that `pyacmetool`
sets to empty.

## Debugging

Use the `DEBUG` variable environment. The general syntax is :

```(facility(=level)?,)*```

Where `facility` can be :

* `pyacmetool`
* `acme.client` for the ACME library

or other Python 3 libraries facilities (like `urllib3.connectionpool`)

Level can be any Python logging level : DEBUG, INFO, WARN…

For example: `DEBUG="pyacmetool,acme.client=INFO" pyacmetool`

## Differences with acmetool

Too much to count, but here are the main one :

* The third argument to `challenge-*` (`filename`) hooks is empty.

* All keys are RSA 2048. Hardcoded. Dont’t bother trying to configure
it.

* Multi-accounts is not supported. If you register multiple accounts,
a random one will be picked.

* In the `desired/` configuration files, only `satisfy.names` is used.

* `acmetool` tries to be clever and not request certificates that will
be redundant. `pyacmetool` is not.
