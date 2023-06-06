# notifications on system startup and shutdown

Systemd services to announce system startup and shutdown via [ntfy.sh](https://ntfy.sh) notifications.

## Install

To install, copy the service files into `/etc/systemd/system/`.

## Usage

Create a topic on ntfy.sh (for custom instance see customization below) and enable the services using *yourTopicName*:

```sh
systemctl enable ntfy-startup@yourTopicName.service
systemctl enable ntfy-shutdown@yourTopicName.service
```

To just test if it works, use `start` instead of `enable`.

## Customization

Aside from directly editing the service files, they can be customized using override configurations (conf file in `/etc/systemd/system/ntfy-startup@yourTopicName.service.d/` or using `systemctl edit ntfy-startup@yourTopicName.service`).

The following environment variables are intended to be customizable:

| Variable name | Description  | Default value | supported by startup | supported by shutdown |
| ------------- | ------------ | --------------| -------------------- | --------------------- |
| `NTFY_URL`    | Base URL to self hosted instance. | `https://ntfy.sh` | ✅️ | ✅️ |
| `NTFY_TAGS`   | Tags/Emoji displayed with the message, see https://docs.ntfy.sh/emojis/ | `green_circle` 🟢️ or `red_circle` 🔴️ | ✅️ | ✅️ |
| `NTFY_MESSAGE` | Additional message content (after "*hostname* is online") | `Startup time: $(uptime)` | ✅️ | ❌️ |

E.g. a message to display the public IPv4, IPv6 and local IP:

```ini
[Service]
Environment=NTFY_MESSAGE="IPv4: $(dig @resolver3.opendns.com myip.opendns.com +short -4) - IPv6: $(dig @resolver3.opendns.com myip.opendns.com +short -6) - Local IPs: $(hostname -i)"
```