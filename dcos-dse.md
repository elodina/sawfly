# DataStax Enterprise Scheduler for Mesosphere Data Center Operating System

# Setting up DCOS DSE

- install dcos-cli (either 1. [repository mesosphere/dcos-cli](https://github.com/mesosphere/dcos-cli/) or 2. [docs mesosphere on dcos cli](https://docs.mesosphere.com/install/cli/))
- clone [elodina/universe](https://github.com/elodina/universe)


    git clone https://github.com/elodina/universe



- if you installed dcos cli via option 1. then set up core.dcos_url, core.mesos_master_url, core.mesos_master_url, marathon.url through `dcos config`
  (NOTE: in development mode, when using [Using the CLI without DCOS](https://github.com/mesosphere/dcos-cli/#using-the-cli-without-dcos), set dse.url via `dcos config set dse.url "http://host0:port0"` , where host0 is host where DSE scheduler launched and port0 is its REST API)
- dcos config append package.sources file://{path to clone of universe repository dir}


    example:
    dcos config append package.sources file:///Users/okovalenko/w/universe


- update packages


    dcos package update


- make sure dse exists in search results


    dcos package search


- installing dse


    dcos package install --options=/Users/okovalenko/w/dse-mesos/run-aws.json dse

    content of run-aws.json

    {
      "mesos": {
        "master": "zk://172.29.233.208:2181/mesos"
      },
      "dse": {
        "app": {
          "mem": 512
          ,"heap-mb": 512
        }
        ,"storage": "zk:172.29.233.208:2181/dse-mesos"
        ,"other-options": "--framework-timeout 0s --debug true"
      }
    }




DCOS DSE CLI proxies requests to jar bundled with DCOS DSE (dcos_dse/dse-mesos*.jar 
which in turn proxies request to DSE scheduler REST API running in DCOS cluster)
thus right after :code:`dcos dse` you could put any known commands that accept
[dse-mesos (applies to scheduler, node, ring options)](https://github.com/elodina/datastax-enterprise-mesos).
NOTE: since dse-mesos*.jar bundled with dcos-dse make sure jar up to date.

For instance:
    
    adding node

    $ dcos dse node add 0 --cpu 1.0 --mem 20480
    node added:
      node:
        id: 0
        state: Inactive
        cpu: 1.0
        mem: 20480
        seed: true

    $ dcos dse node add 1 --cpu 1.0 --mem 20480
    node added:
      node:
        id: 1
        state: Inactive
        cpu: 1.0
        mem: 20480
        seed: true


    list nodes

    $ dcos dse node list
    nodes:
      node:
        id: 0
        state: Running
        cpu: 1.0
        mem: 20480
        seed: true
        seeds: ip-172-29-227-109.dcos2
        runtime:
          task id: node-0-1449576168597
          slave id: 20151207-174947-3504938412-5050-12825-S2
          executor id: node-0-1449576168632
          hostname: ip-172-29-227-109.dcos2
          attributes:

      node:
        id: 1
        state: Running
        cpu: 1.0
        mem: 20480
        seed: false
        seeds: ip-172-29-227-109.dcos2
        runtime:
          task id: node-1-1449576504916
          slave id: 20151207-174947-3504938412-5050-12825-S1
          executor id: node-1-1449576504916
          hostname: ip-172-29-205-30.dcos2
          attributes:

