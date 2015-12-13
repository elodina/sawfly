# Pisaura a Simoorg on Mesos Scheduler

Pisaura is is a scheduler that runs Simoorg on Mesos.

Simoorg is linkedin’s open source chaos failure inducer framework. It was designed to be easy to extend and most of the important components are pluggable.

Pisaura is being actively developed by Elodina Inc. and is available as a free trial. In the event that community support is sufficient, Elodina plans to release the framework as an open source project distributed under the Apache License, Version 2.0.

# Failure scenario
```python
{
    'service_name': 'tested service',
    'cpu': 1,
    'mem': 200,
    'hosts': ['hostname1', 'hostname2'],
    'ssh': {
        'username': '',
        'password': '',
        'identity_file': ''
    },
    'loglevel': 'debug',
    'failfast': True,
    'failures': [
        {
            'name': '',
            'inducer': ["/path/to/inducer1.sh", "arg1", "argN"],
            'reverter': ["/path/to/reverter1.sh", "arg1", "argN"],
            'healthcheck': ["/path/to/healthcheck1.sh", "arg1", "argN"]
        },
        {
            'name': '',
            'inducer': ["/path/to/inducerN.sh", "arg1", "argN"],
            'reverter': ["/path/to/reverterN.sh", "arg1", "argN"],
            'healthcheck': ["/path/to/healthcheckN.sh", "arg1", "argN"]
        }
    ]
}
```

## Main config

| Section | Value |
| --------| ------|
| service_name | title of tested service |
| cpu | needed cpu resources for handling tasks |
| mem | needed memory resources for handling tasks|
| hosts | list of service's hosts |
| ssh | parameters for ssh connection, used for copy inducer and reverter to the hosts and execute them |
| loglevel | level of verbosity, output of logs is stdout/stderr |
| failfast | stop testing if healthcheck didn't passed |
| failures | list of objects with faliures and revertes |


## Failure config

| Section | Value |
| --------| ------|
| name    | output name for failure |
| inducer | script for producing failure, represented as a list, format: `["/path/to/script.sh", "arg1", "argN"]`, will be copied to the tested host |
| reverter | script for reverting state of service, represented as a list, format: `["/path/to/script.sh", "arg1", "argN"]`, will be copied to the tested host |
| healthcheck | script for checking is service in the working state, represented as a list, format: `["/path/to/script.sh", "arg1", "argN"]`, `won't` be copied to the tested host |
