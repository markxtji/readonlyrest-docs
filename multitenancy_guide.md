# Multi-Tenancy Elastic Stack with ReadonlyREST Enterprise

This document will guide you through setting up your Elasticsearch and Kibana stack with ReadonlyREST such that:
* There will be two tenancies: one for Sales and one for Ops department.
* In each tenancy, 3 users will be able to login into Kibana using their own set of credentials
* All users will see **their own, independant** Kibana dashboards, and they may be restricted to visualizing distinct subsets of the whole data contained in Elasticsearch. 

### Users and capabilities
For this tutorials, we want to have three users, each of them has a distinct access level to a shared Kibana tenancy (set of dashboards and settings).

#### Sales Department
|                                       | "sales_admin" | "sales_rw_usr" | "sales_ro_usr" |
|---------------------------------------|---------|----------|----------|
| Can create,edit,delete Sales' dashboards| ✅      | ✅       |          |
| Can change Kibana settings for Sales  | ✅      | ✅       |          |
| Only sees "sales_logstash*" data from 2017|         |          | ✅      |
| Can see "add","delete","edit" buttons | ✅      | ✅       |          |
| "dev-tools" Kibana App is hidden       | ✅      | ✅       |          |
| "readonlyrest" Kibana App is hidden    | ✅      |          |          |


#### Ops Department
|                                       | "ops_admin" | "ops_rw_usr" | "ops_ro_usr" |
|---------------------------------------|---------|----------|----------|
| Can create,edit,delete Ops dashboards | ✅      | ✅       |          |
| Can change Kibana settings for Ops    | ✅      | ✅       |          |
| Only sees ops_logstash data from 2017 |         |          | ✅      |
| Can see "add","delete","edit" buttons | ✅      | ✅       |          |
| "dev-tools" Kibana App is hidden       | ✅      | ✅       |          |
| "readonlyrest" Kibana App is hidden    | ✅      |          |          |


> NB: ReadonlyREST for Elastisearch and ReadonlyREST Enterprise for Kibana have an great amount of features like groups, connector for external systems like LDAP, etc. Don't forget to visit the full documentation and the forum to know more about it.

> NB: The capabilities gained by admin users when they access the "readonlyrest" Kibana App **are global**, that is, they can add/remove tenancies, users, groups, etc.

## Before you start

For the scope of this guide, we will assume:
* You will have a functioning installation of Elasticsearch and Kibana
* You have [installed the ROR plugin for Elasticsearch](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#installing)
* You have [installed the ROR Enterprise plugin for Kibana](https://github.com/beshu-tech/readonlyrest-docs/blob/master/kibana.md#installation)

If you don't have the ROR [Enterprise](https://readonlyrest.com/enterprise.html) for Kibana plugin, get yourself a two weeks free trial build!


## Setup: the Elasticsearch side

Right beside your `elasticsearch.yml`, create a file called `readonlyrest.yml` and write the following settings into it.

```yml
readonlyrest:

    # IMPORTANT FOR LOGIN/LOGOUT TO WORK WITH ROR PLUGIN FOR KIBANA
    prompt_for_basic_auth: false

    access_control_rules:
    
    #########################################################
    # These credentials shall be used by the logstash daemon.
    #########################################################  
    - name: "::LOGSTASH::"
      auth_key: logstash:logstash
      actions: ["indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
      indices: ["*logstash-*"]


    #####################################################################################
    # These credentials have no limitations, and shall be used only by the Kibana deamon.
    #####################################################################################
    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

    ##############################
    # SALES: Actual human users...
    ##############################
    - name: "::RO::"
      auth_key: ro_usr:dev
      kibana_access: ro
      indices: [ ".kibana", ".kibana-devnull", "logstash-2017*"]
      kibana_hide_apps: ["readonlyrest_kbn", "kibana:dev_tools"]
      kibana_index: ".kibana_sales"

    - name: "::RW::"
      auth_key: rw_usr:dev
      kibana_access: rw
      indices: [".kibana", ".kibana-devnull", "logstash-*"]
      kibana_hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:management"]
      kibana_index: ".kibana_sales"

    - name: "::ADMIN::"
      auth_key: admin_usr:dev
      kibana_access: admin
      indices: [".kibana", ".kibana-devnull", "logstash-*"]
      kibana_index: ".kibana_sales"
 
    ###########################
    # OPS Actual human users...
    ###########################
    - name: "::RO::"
      auth_key: ro_usr:dev
      kibana_access: ro
      indices: [ ".kibana", ".kibana-devnull", "logstash-2017*"]
      kibana_hide_apps: ["readonlyrest_kbn", "kibana:dev_tools"]
      kibana_index: ".kibana_ops"

    - name: "::RW::"
      auth_key: rw_usr:dev
      kibana_access: rw
      indices: [".kibana", ".kibana-devnull", "logstash-*"]
      kibana_hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:management"]
      kibana_index: ".kibana_ops"

    - name: "::ADMIN::"
      auth_key: admin_usr:dev
      kibana_access: admin
      indices: [".kibana", ".kibana-devnull", "logstash-*"]
      kibana_index: ".kibana_ops"

```

## Setup: the Kibana side
With ROR, we try as much as possible to keep all the settings withing the Elasticsearch domain. Therefore, you'll notice how few settings are needed on the Kibana side, apart from actually installing the plugin.

Open up `config/kibana.yml` and add/edit the following settings:

```yml
# Kibana server use ::KIBANA-SRV:: credentials
elasticsearch.username: "kibana"
elasticsearch.password: "kibana"
```

## Running
Fire up Elasticsearch
```
$ bin/elasticsearch
```

And then Kibana
```
$ bin/kibana
```


## Loggin in
Now you are ready to point your browser to the Kibana server IP (defauling on port 5601) and you should see a login prompt.
You can login as any user i.e. "sales_rw_usr", or "ops_admin" and the password is always "dev".

Maybe just remember to login with a RW user first, so Kibana can create its own default settings.


