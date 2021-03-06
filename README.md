## SSM shell

[![Build Status](https://travis-ci.org/itsdalmo/ssm-sh.svg?branch=master)](https://travis-ci.org/itsdalmo/ssm-sh)

Little experiment to mimic SSH by using SSM agent to send commands to
remote instances and fetching the output.

## Install

Grab a binary from the [releases](https://github.com/itsdalmo/ssm-sh/releases).

#### Docker

There is also a docker image [here](https://hub.docker.com/r/itsdalmo/ssm-sh/).

#### Manual install

Have Go installed:

```bash
$ which go
/usr/local/bin/go

$ echo $GOPATH
/Users/dalmo/go

$ echo $PATH
# Make sure $GOPATH/bin is in your PATH.
```

Get the repository:

```bash
go get -u github.com/itsdalmo/ssm-sh
```

If everything was successful, you should have a shiny new binary:

```bash
which ssm-sh
# Should point to $GOPATH/bin/ssm-sh
```

## Usage

```bash
$ ssm-sh --help

Usage:
  ssm-sh [OPTIONS] <list | run | shell>

AWS Options:
  -p, --profile= AWS Profile to use. (If you are not using Vaulted).
  -r, --region=  Region to target. (default: eu-west-1)

Help Options:
  -h, --help     Show this help message

Available commands:
  list   List managed instances. (aliases: ls)
  run    Run a command on the targeted instances.
  shell  Start an interactive shell. (aliases: sh)
```

#### List usage

```bash
$ ssm-sh list --help

...
[list command options]
      -f, --filter= Filter the produced list by tag (key=value,..)
      -l, --limit=  Limit the number of instances printed (default: 50)
      -o, --output= Path to a file where the list of instances will be written as JSON.
```

#### Run/shell usage

```bash
$ ssm-sh run --help

...
[run command options]
      -i, --timeout=     Seconds to wait for command result before timing out. (default: 30)
      -t, --target=      One or more instance ids to target
          --target-file= Path to a JSON file containing a list of targets.
```

## Example

```bash
$ vaulted -n lab-admin -- ssm-sh list --filter Name="*itsdalmo" -o example.json

Instance ID         | Name                             | State   | Image ID     | Platform     | Version | IP            | Status | Last pinged
i-03762678c45546813 | ssm-manager-manual-test-itsdalmo | running | ami-db1688a2 | Amazon Linux | 2.0     | 172.53.17.163 | Online | 2018-02-09 12:37
i-0d04464ff18b5db7d | ssm-manager-manual-test-itsdalmo | running | ami-db1688a2 | Amazon Linux | 2.0     | 172.53.20.172 | Online | 2018-02-09 12:39

$ vaulted -n lab-admin -- ssm-sh shell --target-file example.json
Initialized with targets: [i-03762678c45546813 i-0d04464ff18b5db7d]
Type 'exit' to exit. Use ctrl-c to abort running commands.

$ ps aux | grep agent
i-03762678c45546813 - Success:
root      3261  0.0  1.9 243560 19668 ?        Ssl  Jan27   4:29 /usr/bin/amazon-ssm-agent
root      9058  0.0  0.0   9152   936 ?        S    15:02   0:00 grep agent

i-0d04464ff18b5db7d - Success:
root      3245  0.0  1.9 317292 19876 ?        Ssl  Feb05   0:27 /usr/bin/amazon-ssm-agent
root      4893  0.0  0.0   9152   924 ?        S    15:02   0:00 grep agent

$ echo $HOSTNAME
i-03762678c45546813 - Success:
ip-172-53-17-163.eu-west-1.compute.internal

i-0d04464ff18b5db7d - Success:
ip-172-53-20-172.eu-west-1.compute.internal
```

#### Note

If you don't see any instances listed and still want to test `ssm-sh`,
you can see the [terraform/README.md](./terraform/README.md) for a quick
way of setting up some test instances.
