---
layout: "azurerm"
page_title: "Azure Resource Manager: azurerm_scheduler_job"
sidebar_current: "docs-azurerm-resource-scheduler-job-x
description: |-
  Manages a Scheduler Job.
---

# azurerm_scheduler_job

Manages a Scheduler Job.

## Example Usage (single web get now)

```hcl
resource "azurerm_scheduler_job" "web-once-now" {
    name                = "tfex-web-once-now"
    resource_group_name = "${azurerm_resource_group.rg.name}"
    job_collection_name = "${azurerm_scheduler_job_collection.jc.name}"

    state = "enabled" //re-enable it each run

    action_web {
        url = "http://this.url.fails" //defaults to get
    }

    //default start time is now
}
```

## Example Usage (recurring daily with retry and basic authentication)

```hcl
resource "azurerm_scheduler_job" "web-recurring-daily" {
    name                = "tfex-web-recurring-daily"
    resource_group_name = "${azurerm_resource_group.rg.name}"
    job_collection_name = "${azurerm_scheduler_job_collection.jc.name}"

     action_web {
        url     = "https://this.url.fails"
        method  = "put"
        body    = "this is some text"

        headers = {
            Content-Type = "text"
        }

        authentication_basic {
            username = "login"
            password = "apassword"
        }
    }

    retry {
        interval = "00:05:00" //retry every 5 min
        count    =  10        //a maximum or 10 times
    }

    recurrence {
        frequency = "day"
        count     = 1000
        hours     = [0,12]       //run every 12 hours
        minutes   = [0,15,30,45] //4 times an hour
    }

    start_time = "2018-07-07T07:07:07-07:00"
}
```

## Example Usage (recurring monthly with an error action and client certificate authentication)

```hcl
resource "azurerm_scheduler_job" "web-recurring-daily" {
    name                = "tfex-web-recurring-daily"
    resource_group_name = "${azurerm_resource_group.rg.name}"
    job_collection_name = "${azurerm_scheduler_job_collection.jc.name}"

    action_web {
        url     = "https://this.url.fails"
        authentication_certificate {
            pfx      = "${base64encode(file("your_cert.pfx"))}"
            password = "cert_password"
        }
    }

    error_action_web {
        url     = "https://this.url.fails"
        method  = "put"
        body    = "The job failed"

        headers = {
            "Content-Type" = "text"
        }

        authentication_basic {
            username = "login"
            password = "apassword"
        }
    }

    recurrence {
        frequency = "monthly"
        count     = 1000
        monthly_occurrences = [
            { //first sunday
                day        = "Sunday"
                occurrence = 1
            },
            { //third sunday
                day        = "Sunday"
                occurrence = 3
            },
            { //last sunday
                day        = "Sunday"
                occurrence = -1
            }
        ]
    }

    start_time = "2018-07-07T07:07:07-07:00"
}
```

## Argument Reference

The following arguments are supported:

* `name` - (Required) The name of the Scheduler Job. Changing this forces a new resource to be created.

* `resource_group_name` - (Required) The name of the resource group in which to create the Scheduler Job. Changing this forces a new resource to be created.

* `job_collection_name` - (Required) Specifies the name of the Scheduler Job Collection in which the Job should exist. Changing this forces a new resource to be created.

* `action_web` - (Required) A `action_web` block defining the job action as described below. Note this is identical to an `error_action_web` block.

* `error_action_web` - (Optional) A `error_action_web` block defining the action to take on an error as described below. Note this is identical to an `action_web` block.

* `retry` - (Optional) A `retry` block defining how to retry as described below. 

* `recurrence` - (Optional) A `recurrence` block defining a job occurrence schedule. 

* `start_time` - (Optional) The time the first instance of the job is to start running at. 

* `state` - (Optional) The sets or gets the current state of the job. Can be set to either `Enabled` or `Completed`


`web_action` & `error_web_action` block supports the following:

* `url` - (Required) Specifies the URL of the web request. Must be HTTPS for authenticated requests.
* `method` - (Optional) Specifies the method of the request. Defaults to `Get` and must be one of `Get`, `Put`, `Post`, `Delete`.
* `body` - (Optional) Specifies the request body.
* `headers` - (Optional) A map specifying the headers sent with the request.
* `authentication_basic` - (Optional) An `authentication_active_directory` block which defines the Active Directory oauth configuration to use.
* `authentication_certificate` - (Optional) An `authentication_certificate` block which defines the client certificate information to be use.
* `authentication_active_directory` - (Optional) An `authentication_active_directory` block which defines the OAUTH Active Directory information to use.


`authentication_basic` block supports the following:

* `username` - (Required) Specifies the username to use.
* `password` - (Required) Specifies the password to use.


`authentication_certificate` block supports the following:

* `pfx` - (Required) Specifies the pfx certificate in base-64 format.
* `password` - (Required) Specifies the certificate password.


`authentication_active_directory` block supports the following:

* `client_id` - (Required) Specifies the client ID to use.
* `tenant_id` - (Required) Specifies the tenant ID to use.
* `client_secret` - (Required) Specifies the secret to use.
* `audience` - (Optional) Specifies the audience.


`retry` block supports the following:

* `interval` - (Required) Specifies the duration between retries.
* `count` - (Required) Specifies the number of times a retry should be attempted.

`recurrence` block supports the following:

* `frequency` - (Required) Specifies the frequency of recurrence. Must be one of `Minute`, `Hour`, `Day`, `Week`, `Month`.
* `interval` - (Optional) Specifies the interval between executions. Defaults to `1`.
* `count` - (Optional) Specifies the maximum number of times that the job should run.
* `end_time` - (Optional) Specifies the time at which the job will cease running. Must be less then 500 days into the future.
* `minutes` - (Optional) Specifies the minutes of the hour that the job should execute at. Must be between `0` and `59`
* `hours` - (Optional) Specifies the hours of the day that the job should execute at. Must be between `0` and `23`
* `week_days` - (Optional) Specifies the days of the week that the job should execute on. Must be one of `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`, `Saturday`, `Sunday`. Only applies when `Week` is used for frequency.
* `month_days` - (Optional) Specifies the days of the month that the job should execute on. Must be non zero and between `-1` and `31`. Only applies when `Month` is used for frequency.
* `monthly_occurrences` - (Optional) Specifies specific monthly occurrences like "last sunday of the month" with `monthly_occurrences` blocks. Only applies when `Month` is used for frequency.

`monthly_occurrences` block supports the following:

* `day` - (Optional) Specifies the day of the week that the job should execute on. Must be one of  one of `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`, `Saturday`, `Sunday`.
* `occurrence` - (Optional) Specifies the week the job should run on. For example  `1` for the first week, `-1` for the last week of the month. Must be between `-5` and `5`.


## Attributes Reference

The following attributes are exported:

* `id` - The Scheduler Job ID.

`authentication_certificate` block exports the following:

* `thumbprint` - (Computed) The certificate thumbprint.
* `expiration` - (Computed)  The certificate expiration date.
* `subject_name` - (Computed) The certificate's certificate subject name.

## Import

Scheduler Job can be imported using a `resource id`, e.g.

```shell
terraform import azurerm_scheduler_job.job1 /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroup1/providers/Microsoft.Scheduler/jobCollections/jobCollection1/jobs/job1
```
