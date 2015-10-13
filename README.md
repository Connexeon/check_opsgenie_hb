# check_opsgenie_hb
Nagios/Icinga plugin to send a heartbeat to OpsGenie.

[OpsGenie](www.opsgenie.com) is a service that handles notifications, on-call schedules & escalations. It integrates with Nagios/Icinga(2).

An OpsGenie heartbeat expects to be called every x minutes. If not, it will expire and alerts you.

This is a great way to monitor your Icinga/Nagios server, if it's not able to execute the heartbeat, it is probably not monitoring anything else either.

## Create an OpsGenie heartbeat

From your OpsGenie account, setup a heartbeat:

* Create a Heartbeat Integration from Integrations page, copy the API Key from that page
* Add your heartbeat from the Heartbeats page, define a unique name and select your desired interval. Without adding heartbeat you are not able to send heartbeat. There can be more than one heartbeat per API key, so the plugin needs this name to identify the heartbeat to call.

The Icinga2 config sample included below sends the Icinga2 constant `NodeName` as the name of the heartbeat to check. By default that constant contains the full server FQDN as returned by `hostname --fqdn`. Check for the exact value of it in `/etc/icinga2/constants.conf`. You can modify the config to send any name you prefer though.

![http://i.imgur.com/qEJspVP.png](http://i.imgur.com/qEJspVP.png)
## Add plugin to Nagios / Icinga(2)
Put the plugin on your server in the usual plugin directory.

For Icinga2 you can check for the location in `/etc/icinga2/constants.conf`.
```icinga2
const PluginDir = "/opt/monitoring-plugins/libexec"
```

### Icinga2 config example
Add an your OpsGenie API key to the constant `OpsGenieAPIKey`. The constant is used because if you have more OpsGenie configuration (for notifications), you can use that API key constant too instead of repeating it.

The config includes a `CheckCommand` to call the heartbeat and applies a service to the host who's hostname matches the `NodeName` constant, which should be your local Icinga2 server.

Make sure your checks run faster than the expire interval of the heartbeat you configured at OpsGenie.  

```icinga2
const OpsGenieAPIKey = "your-api-key"

apply Service "opsgenie-hb" {
    import "generic-service"
    check_command = "opsgenie-hb"
    assign where host.name == NodeName
}

object CheckCommand "opsgenie-hb" {
    import "plugin-check-command"
    command = [ PluginDir + "/check_opsgenie_hb" ]
    arguments = {
        "-a" = OpsGenieAPIKey
        "-n" = NodeName
    }
}
```
