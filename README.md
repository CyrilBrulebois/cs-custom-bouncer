<p align="center">
<img src="https://github.com/crowdsecurity/crowdsec-custom-bouncer/raw/main/docs/assets/crowdsec_custom_logo.png" alt="CrowdSec" title="CrowdSec" width="280" height="300" />
</p>
<p align="center">
<img src="https://img.shields.io/badge/build-pass-green">
<img src="https://img.shields.io/badge/tests-pass-green">
</p>
<p align="center">
&#x1F4DA; <a href="#installation/">Documentation</a>
&#x1F4A0; <a href="https://hub.crowdsec.net">Hub</a>
&#128172; <a href="https://discourse.crowdsec.net">Discourse </a>
</p>


# crowdsec-custom-bouncer
Crowdsec bouncer written in golang for custom scripts.

crowdsec-custom-bouncer will periodically fetch new and expired/removed decisions from CrowdSec Local API and will pass them as arguments to a custom user script.

## Installation

### With installer

First, download the latest [`crowdsec-custom-bouncer` release](https://github.com/crowdsecurity/crowdsec-custom-bouncer/releases).

```sh
$ tar xzvf crowdsec-custom-bouncer.tgz
$ sudo ./install.sh
```

### From source

Run the following commands:

```bash
git clone https://github.com/crowdsecurity/crowdsec-custom-bouncer.git
cd crowdsec-custom-bouncer/
make release
tar xzvf crowdsec-custom-bouncer.tgz
cd crowdsec-custom-bouncer-v*/
sudo ./install.sh
```

### Start

If your bouncer runs on the same machine as your crowdsec local API, you can start the service directly since the `install.sh` took care of the configuration.
```sh
sudo systemctl start crowdsec-custom-bouncer
```

## Upgrade

If you already have `crowdsec-custom-bouncer` installed, please download the [latest release](https://github.com/crowdsecurity/crowdsec-custom-bouncer/releases) and run the following commands to upgrade it:

```bash
tar xzvf crowdsec-custom-bouncer.tgz
cd crowdsec-custom-bouncer-v*/
sudo ./upgrade.sh
```

## Usage

The custom binary will be called with the following arguments :

```bash
<my_custom_binary> add <ip> <duration> <reason> <json_object> # to add an IP address
<my_custom_binary> del <ip> <duration> <reason> <json_object> # to del an IP address
```

- `ip` : ip address to block `<ip>/<cidr>`
- `duration`: duration of the remediation in seconds
- `reason` : reason of the decision
- `json_object`: the serialized decision

:warning: don't forget to add execution permissions to your binary/script

### Examples:

```bash
custom_binary.sh add 1.2.3.4/32 3600 "test blacklist"
custom_binary.sh del 1.2.3.4/32 3600 "test blacklist"
```

## Configuration

Before starting the `crowdsec-custom-bouncer` service, please edit the configuration to add your API url and key.
The default configuration file is located under : `/etc/crowdsec/bouncers/`

```sh
$ vim /etc/crowdsec/bouncers/crowdsec-custom-bouncer.yaml
```

```yaml
bin_path: <absolute_path_to_binary>
piddir: /var/run/
update_frequency: 10s
daemonize: true
log_mode: file
log_dir: /var/log/
log_level: info
api_url: <API_URL>  # when install, default is "localhost:8080"
api_key: <API_KEY>  # Add your API key generated with `cscli bouncers add --name <bouncer_name>`
cache_retention_duration: 10s 
```

`cache_retention_duration` : The bouncer keeps track of all custom script invocations from the last `cache_retention_duration` interval. If a decision is identical to some decision already present in the cache, then the custom script is not invoked. The keys for hashing a decision is it's `Type` (eg `ban`, `captcha` etc) and `Value` (eg `1.2.3.4`,  `CH` etc).

You can then start the service:

```sh
sudo systemctl start crowdsec-custom-bouncer
```
