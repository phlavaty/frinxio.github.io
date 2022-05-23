---
order: 15
---

# Uniconfig 5.0.7 Release Notes


## :white_check_mark: New Features

- Implementation of context-match shell operation

## :x: Bug Fixes

- Fixed establishing of NETCONF stream sessions

- Fixed NetconfDeviceCommunicatorTest

- Fixed Uniconfig client json parser tests

- Fix building history output

- Fixed execution of YANG action under some list entry from shell

- Fixed ordering transaction log by date

- Fixed types of the network-instance/interfaces

- Making subscription monitoring loop more robust

- Cli session closed/disconnected

- Fixed removing of data-change-event subscription

- Fixed merging template attribute to replaced node

- CLI shell: harmonised composite key delimiter input

## :bulb: Improvements

- Updated netconf-node-topology:concurrent-rpc-limit parameter

- Refactored global DOMSchemaService

- Optimization of calculate-diff RPC

- Swagger and YangPackager improvements

- Fix all owasp sec issues level 8

## :wrench: Other

- Upgrade sshd libs to version 2.8.0

- Upgrade and cleanup usage of jaxb

- Upgrade jetty/jersey/jax-rs dependencies

- Reorganisation of NETCONF connection parameters

- Renaming/reorganisation of NETCONF connection parameters
