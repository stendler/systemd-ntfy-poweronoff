# notifications on system startup and shutdown

Systemd services to announce system startup and shutdown via [ntfy.sh](https://ntfy.sh) notifications.

## Install

To install, copy the service files into `/etc/systemd/system/`.

Optionally, copy `ntfy-failure@.service` and `test-fail.service` into `/etc/systemd/user/` to allow them to be used for user services.  
In this case, for ntfy-failure use the ExecStart with the `--user` flag added to the journalctl command.
Otherwise, it defaults to the system logs, which the current user might not have access to and which likely won't contain the relevant error message.

## Usage

Create a topic on ntfy.sh (for a custom instance, see customization below) and enable the services using *yourTopicName*:

```sh
systemctl enable ntfy-startup@yourTopicName.service
systemctl enable ntfy-shutdown@yourTopicName.service
```

To just test if it works, use `start` instead of `enable`.

For the `ntfy-failure@.service` an override is required to set the topic: `systemctl [--user] edit ntfy-failure@.service` and add:

```ini
[Service]
Environment=NTFY_TOPIC=yourTopicName
```

To test if this works run `systemctl [--user] start test-fail`.

Now, you can set `OnFailure=ntfy-failure@$i.service` in the `[Unit]` section of any service or unit you want.
Even globally for all services in `/etc/systemd/system/service.d/onfailure.conf` and/or in `/etc/systemd/user/service.d/onfailure.conf`:

```ini
[Unit]
OnFailure=ntfy-failure@%i.service
```

## Customization

Aside from directly editing the service files, they can be customized using override configurations (conf file in `/etc/systemd/system/ntfy-startup@yourTopicName.service.d/` or using `systemctl edit ntfy-startup@yourTopicName.service`).

The following environment variables are intended to be customizable:

| Variable name | Description                                                                                                              | Default value                                           | supported by startup | supported by shutdown | supported by failure |
| ------------- |--------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|----------------------|-----------------------|----------------------|
| `NTFY_URL`    | Base URL to self hosted instance.                                                                                        | `https://ntfy.sh`                                       | ✅️                   | ✅️                    | ✅                    |
| `NTFY_TAGS`   | Tags/Emoji displayed with the message, see https://docs.ntfy.sh/emojis/                                                  | `green_circle` 🟢️ or `red_circle` 🔴️ or `warning` ⚠️  | ✅️                   | ✅️                    | ✅                    |
| `NTFY_MESSAGE` | Additional message content (after "*hostname* is online")                                                                | `Startup time: $(uptime)`                               | ✅️                   | ❌️                    | ❌                    |
| `NTFY_TOPIC` | Specify topic to publish to. Only for `ntfy-failure` - set this as part of the service name for startup/shutown instead. |                                                         | ❌                    | ❌                     | ✅                    |

E.g., a message to display the public IPv4, IPv6 and local IP:

```ini
[Service]
Environment=NTFY_MESSAGE="IPv4: $(dig @resolver3.opendns.com myip.opendns.com +short -4) - IPv6: $(dig @resolver3.opendns.com myip.opendns.com +short -6) - Local IPs: $(hostname -i)"
```