# Extension conventions                                                        
                                                                                 
There are three ways of passing information to plugins using the Container Network Interface (CNI), none of which require the [spec](SPEC.md) to be updated. These are 
- plugin specific fields in the JSON config
- `args` field in the JSON config
- `CNI_ARGS` environment variable 

This document aims to provide guidance on which method should be used and to provide a convention for how common information should be passed.
Establishing these conventions allows plugins to work across multiple runtimes. This helps both plugins and the runtimes.

## Plugins
* Plugin authors should aim to support these conventions where it makes sense for their plugin. This means they are more likely to "just work" with a wider range of runtimes.

## Runtimes
* Runtime authors should follow these conventions if they want to pass additional information to plugins. This will allow the extra information to be consumed by the widest range of plugins.

# Current conventions
Additional conventions can be created by creating PRs which modify this document.

## Plugin specific fields
[Plugin specific fields](https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration) formed part of the original CNI spec and have been present since the initial release.
> Plugins may define additional fields that they accept and may generate an error if called with unknown fields. The exception to this is the args field may be used to pass arbitrary data which may be ignored by plugins.

A plugin can define any additional fields it needs to work properly. It is expected that it will return an error if if can't act on fields that were expected or where the field values were malformed.

This method of passing information to a plugin is recommended when the information should be acted on and has specific meaning to that plugin.

| Area  | Purpose| Spec and Example | Runtime implementations | Plugin Implementations |
| ------ | ------ | ------             | ------  | ------                  | ------                 |  
| port mappings | Pass mapping from ports on the host to ports in the container network namespace. | <pre>"port_mappings" : [<br />  { "host_port": 8080, "container_port": 80, "protocol": "tcp" },<br />  { "host_port": 8000, "container_port": 8001, "protocol": "udp" }<br />] </pre> | none | none |

## "args" in network config
`args` in [network config](https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration) were introduced as an optional field into the `0.1.0` CNI spec. The first CNI code release that it appeared in was `v0.4.0`. 
> args (dictionary): Optional additional arguments provided by the container runtime. For example a dictionary of labels could be passed to CNI plugins by adding them to a labels field under args.

`args` provide a way of providing more structured data than the flat strings that CNI_ARGS can support.

`args` should be used for _optional_ meta-data. Runtimes can place additional data in `args` and plugins that don't understand that data should just ignore it. Runtimes should not require that a plugin understands or consumes that data provided, and so a runtime should not expect to receive an error if the data could not be acted on.

This method of passing information to a plugin is recommended when the information is optional and the plugin can choose to ignore it. It's often that case that such information is passed to all plugins by the runtime whithout regard for whether the plugin can understand it. 

The conventions documented here are all namepaced under `cni` so they don't conflict with any existing `args`.

For example:
```json
{  
   "cniVersion":"0.2.0",
   "name":"net",
   "args":{  
      "cni":{  
         "labels": [{"key": "app", "value": "myapp"}]
      }
   },
   <REST OF CNI CONFIG HERE>
   "ipam":{  
     <IPAM CONFIG HERE>
   }
}
```

| Area  | Purpose| Spec and Example | Runtime implementations | Plugin Implementations |
| ------ | ------ | ------             | ------  | ------                  | ------                 |  
| labels | Pass`key=value` labels to plugins | <pre>"labels" : [<br />  { "key" : "app", "value" : "myapp" },<br />  { "key" : "env", "value" : "prod" }<br />] </pre> | none | none |

## CNI_ARGS
CNI_ARGS formed part of the original CNI spec and have been present since the initial release.
> `CNI_ARGS`: Extra arguments passed in by the user at invocation time. Alphanumeric key-value pairs separated by semicolons; for example, "FOO=BAR;ABC=123"

The use of `CNI_ARGS` is deprecated and "args" should be used instead.

| Field  | Purpose| Spec and Example | Runtime implementations | Plugin Implementations |
| ------ | ------ | ------             | ------  | ------                  | ------                 |  
| IP     | Request a specific IP from IPAM plugins | IP=192.168.10.4 | *rkt* supports passing additional arguments to plugins and the [documentation](https://coreos.com/rkt/docs/latest/networking/overriding-defaults.html) suggests IP can be used. | host-local (since version v0.2.0) supports the field for IPv4 only - [documentation](https://github.com/containernetworking/cni/blob/master/Documentation/host-local.md#supported-arguments).|
                                                                                                                  
