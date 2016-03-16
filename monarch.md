# Monarch

## Requirements
- Go 1.4 or higher
- Godep (go get github.com/tools/godep)
- Mesos
- Kafka
- Vault

## Build
Get the source:
```bash
$ git clone git@github.com:elodina/monarch.git
$ cd monarch
```

And buld it:
```bash
$ godep restore
$ go build .
```

## Using Docker
You can use prepared script to build docker image:
```bash
./build_docker.sh
```
It'll create docker image with tag `monarch`

For this build you should have files in current dir:
```
'server.crt' - SSL cert for server
'server.key' - SSL private key
'producer.properties' - Configuration for Kafka Producer
```

Than, you can run it, for example this way:
```bash
docker run \
  -e "VAULT_TOKEN=$VAULT_TOKEN" \
  monarch \
  monarch scheduler -artifact.host=192.168.99.100 -master=192.168.99.100:5050
```

## Without Docker
You can use Monarch without docker, just prepare following files and place them somewhere in your mesos cluster:
- "monarch": executable binary (for launching scheduler and cli commands)
- "executor.tgz": archive with "monarch" binary for executors
- "server.crt" - SSL cert for server
- "server.key" - SSL private key

## Usage
```bash
usage: monarch [--version] [--help] <command> [<args>]

Available commands are:
    adduser         Add new client and get api key
    executor        Start Monarch executor
    refreshtoken    Get a new token for user
    scale           Scale executors
    scheduler       Starts Monarch Scheduler
```

### Step 1. Connect to Vault
Ensure that Vault is up and running and set environment variable `VAULT_TOKEN` to your Vault authentication token.

### Step 2. Run scheduler
```bash
$ ./monarch scheduler --artifact-host=<artifact_server_host> --master=<mesos_master_host:port> --broker-list=<kafka.broker1:port>,<kafka.broker2:port>
```
On this step by default you will have no executor instances run with scheduler.

Parameters:
```
  --artifact-host: Artifact server host. Default: "localhost"
  --artifact-port: Artifact server port. Default: 4242
  --master: Mesos Master address <ip:port>. Default: "127.0.0.1:5050"
  --executor.archive: Executor archive name. Default: "executor.tgz"
  --vault: Vault URL. Default: "http://127.0.0.1:8200/"
  --broker-list: Kafka broker list. Default: "localhost:9092"
  --cpu: CPU per executor task. Default: 0.1
  --mem: Memory per task. Default: 64
  --user: Framework user. Default: "root"
  --topic: Topic for schema registry. Default: "schemas"
```

### Step 3. Run executors
To add new executors to scheduler there is `scale` command:
```bash
$ ./monarch scale --executor <executor_name> --instances <number>
```
Parameters:
```
  --executor: Executor name to scale. Now supporterd `beaker` and `schema` executors.
  --instances: Number of instances. Could be less or more than now launched.
```

### Step 4. Add clients
To add client to Monarch use `adduser` command:
```bash
$ ./monarch adduser -name=johnsnow -api="http://scheduler.server.ip:port"
User johnsnow created. API key: fc36f4c8-edd5-499b-8601-bcc9640cb233
```

### Step 5 (Optional). Refresh client api key
To assign new api key for user:
```bash
$ ./monarch refreshtoken -name=johnsnow -api="http://scheduler.server.ip:port"
Done! New token: 6014b9db-75c9-462f-a8af-a55f8a188963
```

## Monarch Push API

### Authentication
To push messages via Monarch Push API, you will need to send authentication headers:

- X-Api-User: client name
- X-Api-Key: client api key

### Posting data
Executors of Monarch are listening for HTTPS POST requests with JSON arrays of messages. Message have following structure:

- topic: string with name of topic
- data: base64-encoded bytestring with data

Example request:
```json
[{
    "topic": "metrics",
    "data": "dGVzdA=="
}]
```
This request will produce data to the namespaced topic `<clientname>_metrics`.

## Monarch Schema Registry

Monarch Schema Registry executor is fully compatible with [Confluent Schema Registry API](http://docs.confluent.io/2.0.0/schema-registry/docs/index.html).
