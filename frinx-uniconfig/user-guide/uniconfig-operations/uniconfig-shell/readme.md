# UniConfig Shell

UniConfig shell provides SSH server API for communication with UniConfig using command-line interface. Using shell,
it is possible to read operational data of devices, manipulate configuration of devices, templates, or unistore
nodes, invoke device or UniConfig operations, or read/change some UniConfig settings.
Most of the shell interface is auto-generated from YANG schemas (e.g., tree structure of data-nodes or available
RPC/action operations).

## Configuration

By default, UniConfig shell is disabled. To enable it, flag 'cliShell/sshServer/enabled' must be set to 'true'
in the 'config/lighty-uniconfig-config.json' file.

All available settings and descriptions are displayed in the following JSON snippet.

```json UniConfig shell configuration (config/lighty-uniconfig-config.json)
    "cliShell": {
        "sshServer": {
            // Flag that determines if ssh server will be started or not.
            "enabled": false,
            // Port bind to ssh server.
            "port": 2022,
            // IP address bind to ssh server.
            "inetAddress": "127.0.0.1",
            // Basic username + password authentication.
            "usernamePasswordAuth": {
                "username": "admin",
                "password": "secret"
            }
        },
        // Flag that determines if uniconfig-shell provides scrolling of output
        "enableScrolling": false,
        // Number of history items to keep in memory
        "historySize": 500,
        // Number of history items to keep in the history file
        "historyFileSize": 1000
    }
```

After starting UniConfig, SSH server with UniConfig shell is started listening on port 2022 and loopback interface.

## Navigation in the shell

* Every command line starts with command prompt that ends with '>' character. Identifier of the command prompt changes
  based on current shell mode and state of the execution in this mode.
* Commands 'exit' and 'quit' appear in all shell modes. Command 'exit' returns state to the parent state, command 'quit'
  returns state to the nearest parent mode (e.g., configuration mode, root mode, operational show mode). If the current
  state of shell represents some mode, commands 'quit' and 'exit' have the same effect - returning to the parent mode.
* Typed command can be sent to the UniConfig using ENTER key. After that, UniConfig processes the command and may
  send response to the console depending on the command behaviour. All commands are processed synchronously - user 
  cannot execute multiple commands in parallel in the same SSH session.
* CTRL-A and CTRL-E can be used for moving cursor on the current line to the beginning of the line, or the end of the
  line, respectively.
* CTRl-L is used for clearing of the shell screen.
* Arrow keys UP/DOWN are used for loading of the previous or more recent command in the command history.
* To cancel current line and moving to the blank line, user can use CTRL+C shortcut.
* Key TAB can be used for loading of the available suggestions in the current context. By hitting TAB again, user can
  navigate through suggested commands using arrow keys and select some of them using ENTER. Submode with suggestions
  can be left using CTRL-E shortcut. Text in the brackets contain description of the next command.

```shell Loading available suggestions under 'show uniconfig iosxr' command
config>show uniconfig iosxr
>                                                                            (output to file)
evpn                                       (Top level configuration containers for EVPN data)
interfaces        (Top level container for interfaces, including configuration
and state data.)
logging                                               (Enclosing container for logging data.)
network-instances                       (The L2, L3, or L2+L3 forwarding instances that are…)
oam                                 (Top level configuration container for ethernet OAM data)
snmp                                    (Enclosing container for snmp interface-specific
data.)
|
```

* If displayed output is longer than the current length of command-line window, output is displayed with scrolling 
  capability. ENTER is used for displaying next line, while SPACE is used for displaying next page. Character 'q'
  can be used to leave scrolling mode. It is possible to scroll only in the one direction - towards end of output.

![Scrolling through long output](scrolling_example.png)

## Root mode

* Root mode represents initial mode that is provided to user after successful authentication.
* Example: login into UniConfig shell:

```shell Connecting to UniConfig shell using SSH client
Connection to 127.0.0.1 closed.
[jtoth@JT-WORK ~]$ ssh admin@127.0.0.1 -p 2022
Password authentication
(admin@127.0.0.1) Password:
uniconfig>
```

```shell Root mode overview
uniconfig>
configuration-mode      (opening configuration mode)
exit
request                          (execution of RPCs)
show                      (reading data from device)
show-history (show history [max number of commands])
```

* Command 'exit' is used for leaving of the whole UniConfig shell interface (disconnecting SSH client).
* Example - leaving UniConfig shell:

```shell Leaving UniConfig shell
uniconfig>exit
Console closed. Press any key to exit!
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
```

!!!
Currently, only username/password single-user authentication is supported based on configuration
in the 'config/lighty-uniconfig-config.json'.
!!!

### Accessing sub-modes

* This mode supplies gateway for opening configuration, show, and request operational modes.
* Example - switching into Configuration mode:

```shell Opening configuration mode
uniconfig>configuration-mode
config>
```

### Show command history

* Command 'show-history' is used for displaying list of N last invoked commands.
* Example - show executed last 5 commands:

```shell Show history
config>show-history 5
----- History of commands -----
10-05-2022 14:48:14 : configuration-mode
10-05-2022 14:48:16 : request
10-05-2022 14:48:17 : exitr
10-05-2022 14:48:18 : exit
10-05-2022 14:48:24 : show-history 5
```

* This command is also available from the Configuration mode.

!!!
List of invoked commands are persisted across UniConfig restarts and SSH connections.
!!!

## Configuration mode

* Configuration mode provides access to:
  
1. CRUD operations on top of persisted uniconfig, unistore and template nodes.
2. CRUD operations on top of persisted UniConfig settings.
3. UniConfig RPC operations such as commit or calculate-diff.

* After opening Configuration mode, a new UniConfig transaction is created. All operations that are invoked from
  the configuration mode are executed in scope of created transaction.
* Transaction is automatically closed after leaving of configuration mode ('exit' or 'quit' command).
* If user invokes 'commit' or 'checked-commit', transaction is automatically refreshed (user stays in the configuration
  mode with newly created transaction).

```shell Configuration mode overview
config>
others
delete                                     (delete operation)
exit                                  (return to parent mode)
quit                                    (return to root mode)
request                                   (execution of RPCs)
set                                           (set operation)
show                                         (show operation)
show-history          (show history [max number of commands])
aliases
diff (alias for 'request calculate-diff target-nodes/node *')
```

### Show configuration

* Show operation can be used for displaying selected subtrees.
* Subtree path can be constructed interactively with help from shell suggestions / auto-completion mechanism.
  Construction of the path works the same way in the Show, Delete, and Set operations.

* Example - displaying configuration of selected container:

```shell Show operation: available topologies or root configuration containers
config>show
settings            (settings)   uniconfig (uniconfig topology)
template   (template topology)   unistore   (unistore topology)
```

```shell Show operation: selection of installed node
config>show uniconfig
nodes
iosxr    iosxr2
```

```shell Show operation: selection of root data container
config>show uniconfig iosxr
>                                                                            (output to file)
evpn                                       (Top level configuration containers for EVPN data)
interfaces        (Top level container for interfaces, including configuration
and state data.)
logging                                               (Enclosing container for logging data.)
network-instances                       (The L2, L3, or L2+L3 forwarding instances that are…)
oam                                 (Top level configuration container for ethernet OAM data)
snmp                                    (Enclosing container for snmp interface-specific
data.)
```

```shell Show operation: selection of interface id
config>show uniconfig iosxr interfaces interface
>     (output to file)   GigabitEthernet0/0/0/2   MgmtEth0/0/CPU0/0
GigabitEthernet0/0/0/0   GigabitEthernet0/0/0/3   |               (pipe)
GigabitEthernet0/0/0/1   GigabitEthernet0/0/0/4
```

```shell Show operation: invocation of command
config>show uniconfig iosxr interfaces interface GigabitEthernet0/0/0/0 config
{
  "type": "iana-if-type:ethernetCsmacd",
  "enabled": false,
  "name": "GigabitEthernet0/0/0/0"
}
```

### Delete configuration

* Delete operation is used for removal of selected subtree.
* Example - removal of container:

```shell Construction of path with help from suggestions
config>delete uniconfig iosxr
evpn                                       (Top level configuration containers for EVPN data)
interfaces        (Top level container for interfaces, including configuration
and state data.)
logging                                               (Enclosing container for logging data.)
network-instances                       (The L2, L3, or L2+L3 forwarding instances that are…)
oam                                 (Top level configuration container for ethernet OAM data)
snmp                                    (Enclosing container for snmp interface-specific
data.)
```

```shell Delete 'config' container under 'network-instance' with key value 'default'
config>delete uniconfig iosxr network-instances network-instance default config
config>
```

```shell Verify state of 'network-instance'
config>show uniconfig iosxr network-instances network-instance default
{
  "name": "default",
  "protocols": {
    "protocol": [
      {
        "identifier": "frinx-openconfig-policy-types:STATIC",
        "name": "default",
        "config": {
          "identifier": "frinx-openconfig-policy-types:STATIC",
          "name": "default"
        }
      }
    ]
  }
}
```

### Set configuration

* Set operation can be used for:

1. Setting value of single leaf.
2. Setting values of multiple leaves in the single shell operation.
3. Setting list of values of leaf-list.
4. Replacing the whole subtree using provided JSON snippet.

* Example - setting value of single leaf:

```shell Providing datatype hints at selected leaf
config>set uniconfig iosxr lacp config system-priority
(type: uint16, constraints: [Range: [[0..65535]]])
```

```shell Setting value of LACP 'system-priority' to '100'
config>set uniconfig iosxr lacp config system-priority 100
config>
```

```shell Displaying changed LACP configuration 
config>show uniconfig iosxr lacp config
{
  "system-priority": 100
}
```

* Example - setting values of multiple leaves: under 'hold-time' container:

```shell Displaying hint for value of the leaf 'up'
config>set uniconfig iosxr interfaces interface GigabitEthernet0/0/0/0 hold-time config up
(type: uint32, constraints: [Range: [[0..4294967295]]])
```

```shell Setting value of the leaf 'up' and displaying hint for value of the leaf 'down'
config>set uniconfig iosxr interfaces interface GigabitEthernet0/0/0/0 hold-time config up 20 down
(type: uint32, constraints: [Range: [[0..4294967295]]])
```

```shell Setting values of the leaves 'up' and 'down'
config>set uniconfig iosxr interfaces interface GigabitEthernet0/0/0/0 hold-time config up 20 down 15
config>
```

```shell Verification of the 'hold-time' configuration
config>show uniconfig iosxr interfaces interface GigabitEthernet0/0/0/0 hold-time
{
  "config": {
    "up": 20,
    "down": 15
  }
}
```

* JSON snippet can be written to selected data-tree node by entering 'json' sub-mode. In the 'json', user can type
  multiple lines that must conform JSON formatting. At the end, user can confirm set operation with input json
  using pattern 'w!' + newline or cancel set operation using 'q!' + newline pattern.
* Example - replacing configuration of interface using provided JSON snippet:

```shell Replacing 'config' container under interface using provided JSON snippet
config>set uniconfig iosxr interfaces interface GigabitEthernet0/0/0/1 config json
{
>   "config": {
>     "type": "iana-if-type:ethernetCsmacd",
>     "enabled": true,
>     "name": "GigabitEthernet0/0/0/1"
>   }
> }
w!
```

```shell Verification of Set operation
config>show uniconfig iosxr interfaces interface GigabitEthernet0/0/0/1
{
  "name": "GigabitEthernet0/0/0/1",
    "config": {
    "type": "iana-if-type:ethernetCsmacd",
    "enabled": true,
    "name": "GigabitEthernet0/0/0/1"
  }
}
```

* Example - leaving 'json' sub-mode without execution of Set operation:

```shell Cancellation of the Set operation and leaving 'json' sub-mode
config>set uniconfig iosxr interfaces interface GigabitEthernet0/0/0/1 config json
{
>   "config": {
>     "type": "iana-if-type:ethernetCsmacd",
>     "enabled": true,
>     "name": "GigabitEthernet0/0/0/1"
>   }
> }
q!
```

```shell Verification of the cancelled Set operation
config>show uniconfig iosxr interfaces interface GigabitEthernet0/0/0/1
{
  "name": "GigabitEthernet0/0/0/1",
  "config": {
    "type": "iana-if-type:ethernetCsmacd",
    "enabled": false,
    "name": "GigabitEthernet0/0/0/1"
  }
}
```

### Execute UniConfig operation

* Command 'request' us used for execution of UniConfig operations such as 'commit' or 'calculate-diff'
  in the UniConfig transaction.
* User can fill in input parameters and values interactively or via provided JSON snippet.

- Example - execution of UniConfig RPCs in the scope of open UniConfig transaction:

```shell Displaying available 'commit' RPC parameters
config>request commit
>                                                                            (output to file)
do-rollback (Controls whether to roll back successfully configured devices in case of failu…)
do-validate (Option to enable/disable validation at commit. Default value is true - validate)
json                                                                             (JSON input)
target-nodes/node
|                                                                                      (pipe)
```

```shell Execution of 'calculate-diff' RPC with 1 argument - target node
config>request calculate-diff target-nodes/node iosxr
{
  "node-results": {
    "node-result": [
      {
        "node-id": "iosxr",
        "status": "complete",
        "created-data": [
          {
            "path": "/network-topology:network-topology/topology=uniconfig/node=iosxr/frinx-uniconfig-topology:configuration/frinx-openconfig-system:system",
            "data": "{
              "frinx-openconfig-system:system": {
                "frinx-huawei-global-config-extension:banner": {
                  "config": {
                    "banner-text": "Test banner"
                  }
                }
              }
            }"
          }
        ]
      }
    ]
  },
  "overall-status": "complete"
}
```

```shell Displaying available 'sync-from-network' RPC paramaters:
config>request sync-from-network
>                                                                            (output to file)
check-timestamp (Perform timestamp comparison(last known to Uniconfig vs current timestamp …)
json                                                                             (JSON input)
target-nodes/node
|                                                                                      (pipe)
```

```shell Execution of 'sync-from-network' RPC with 2 arguments - target node and 'check-timestamp' flag
config>request sync-from-network check-timestamp true target-nodes/node iosxr
{
  "node-results": {
    "node-result": [
      {
        "node-id": "iosxr",
        "status": "complete"
      }
    ]
  },
  "overall-status": "complete"
}
```

## Show operational mode

* Show mode allows user to:

1. Display operational data placed on UniConfig - e.g., logging status, list of open transactions, or list of acquired
   subscriptions.
2. Display operational data on NETCONF or CLI device.

```shell Overview of Show operational mode
show>
others
cli                    (reading data from CLI device)
exit                          (return to parent mode)
logging-status               (reading logging status)
netconf            (reading data from NETCONF device)
netconf-subscriptions (reading netconf subscriptions)
notifications                 (reading notifications)
quit                            (return to root mode)
show-history  (show history [max number of commands])
snapshots-metadata       (reading snapshots metadata)
transaction-log             (reading transaction log)
transactions               (reading transaction data)
aliases
lbr      (alias for 'logging-status broker restconf')
```

* After opening Show mode, a new UniConfif transaction is open. Transaction is closed after leaving this mode.

```shell Opening Show operational mode
uniconfig>show
show>
```

- Example - displaying configuration of selected subtree:

```shell Display configuration of GigabitEthernet0/0/0/0 interface
show>cli iosxr interfaces(frinx-openconfig-interfaces) interface GigabitEthernet0/0/0/0
{
  "name": "GigabitEthernet0/0/0/0",
  "config": {
    "type": "iana-if-type:ethernetCsmacd",
    "enabled": false,
    "name": "GigabitEthernet0/0/0/0"
  }
}
```

- Example - displaying selected system configuration:

```shell Display list of open UniConfig transactions
show>transactions transaction-data
[
  {
    "transaction-id": "5d9c8819-5b05-4c7a-b7e5-3c84478aeeb0",
    "idle-timeout": 300,
    "last-access-time": "2022-May-16 07:02:31.501 +0000",
    "hard-timeout": 1800,
    "creation-time": "2022-May-16 07:02:31.501 +0000"
  },
  {
    "transaction-id": "80091b4b-5432-41cd-9277-1b18ae77b45f",
    "idle-timeout": 300,
    "last-access-time": "2022-May-16 07:02:42.747 +0000",
    "hard-timeout": 1800,
    "creation-time": "2022-May-16 07:02:42.747 +0000"
  }
]
```

## Request operational mode

* Request mode allows user to:

1. Invoke selected UniConfig requests that reads or alter UniConfig operation.
2. Invoke RPCs or actions that are provided by devices or southbound mount-points.

* User can fill in input parameters and values interactively or via provided JSON snippet.
* After opening Request mode, a new UniConfif transaction is open. Transaction is closed after leaving this mode.

- Example - invocation of RPC 'execute-and-read' with typed input parameters:

```shell Displaying available RPCs provided by device 'iosxr'
request>cli iosxr
operations
clear-journal
execute (Simple execution of single or multiple commands on remote terminal. Multiple comma…)
execute-and-expect (Form of the 'execute-and-read' RPC that can contain 'expect(..)' patter…)
execute-and-read (Execution of the sequence of commands specified in the input. These comma…)
execute-and-read-until
read-journal
```

```shell Displaying available 'execute-and-read' RPC parameters
request>cli iosxr execute-and-read
>                                                                            (output to file)
command        (Input configuration snippet (one or multiple commands separated by newline).)
json                                                                             (JSON input)
wait-for-output-timer (If no output is received during this time, then execute next command…)
|                                                                                      (pipe)
```

```shell Execution of 'execute-and-read' RPC with two arguments - 'wait-for-output-timer' and 'command'
request>cli iosxr execute-and-read wait-for-output-timer 2 command "show users"
{
   "output": "Mon May 16 07:28:30.405 UTC
   Line            User                 Service  Conns   Idle        Location
*  vty0            cisco                ssh          0  00:00:00     192.168.1.42"
}
```

- Example - execution of the same RPC 'execute-and-read' using input JSON:

```shell Execution of 'execute-and-read' RPC with input JSON snippet:
request>cli iosxr execute-and-read json
  {
>     "input": {
>         "command": "show users",
>         "wait-for-output-timer": 2
>     }
> }
w!
{
   "output": "Mon May 16 07:37:55.256 UTC
   Line            User                 Service  Conns   Idle        Location
*  vty0            cisco                ssh          0  00:00:00     192.168.1.42"
}
```

!!!
UniConfig shell doesn't support interactive typing of input arguments to RPC/action that contains 'list' YANG element.
Such operation must be executed using input JSON.
!!!

## Pipe operations

## Redirection of output

## Aliases
