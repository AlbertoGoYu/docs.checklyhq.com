---
title: Checks and groups for Terraform - Checkly Docs
displayTitle: Checks and groups for Terraform
navTitle: Checks and groups
weight: 2
menu:
  integrations:
    parent: "Terraform provider"
---

Checks make up the most important unit of your Checkly monitoring setup. Groups help you split them up according to your needs and keep complexity down.

## Checks

[Browser checks](/docs/browser-checks/), [API checks](/docs/api-checks/), [Heartbeat monitors](/docs/heartbeat-monitors/) and [Multistep checks](/docs/multistep-checks/) share many arguments, configuration-wise, 
but also have their own unique ones. The type of your check is controlled using the `type` argument.

### Browser checks

For example, a browser check can look as follows:

```terraform
resource "checkly_check" "e2e-login" {
  name                      = "Login flow"    // The name of the check
  type                      = "BROWSER"       // The type of the check
  activated                 = true            // Whether the check will start as active on creation
  frequency                 = 10              // The frequency of the check in minutes
  ssl_check                 = true            // Whether your SSL cert for the given domain should be checked too
  use_global_alert_settings = true            // Whether to use account level alert setting instead of the alert setting defined on this check
  run_parallel              = true            // Whether the check would run in a single location at time (round-robin) or all locations on each run

  locations = [                               // Which locations the check should run from (if not in a group)
    "us-west-1",
    "eu-central-1"
  ]

  retry_strategy = {                          // The retry strategy used for failed runs
    type = "LINEAR"
  }            

  // The script the check should execute
  script = <<EOT
const { expect, test } = require('@playwright/test');

test('Login flow', async ({ page }) => {
  await page.goto('https://danube-web.shop/');
  await page.getByRole('button', { name: 'Log in' }).click();

  await page.getByPlaceholder('Email').type('user@email.com');
  await page.getByPlaceholder('Password').type('supersecure1');

  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page.getByText(/Welcome back/)).toBeVisible();
});
EOT
}
```

For tidiness and ease of use, it is recommended to store scripts in separate files, instead of using the inline option:

```terraform
resource "checkly_check" "e2e-login" {
  name                      = "Login Flow"
  type                      = "BROWSER"
  activated                 = true
  should_fail               = false
  frequency                 = 1
  ssl_check                 = false
  use_global_alert_settings = true

  locations = [
    "us-west-1",
    "eu-central-1"
  ]

  retry_strategy = {
    type = "LINEAR"
  }

  script = file("${path.module}/scripts/login.spec.js") // Our script is contained in this file
}
```

> You can see all the configuration options for checks on the official Terraform registry [documentation page](https://registry.terraform.io/providers/checkly/checkly/latest/docs/resources/check#schema).

### API checks

On the other hand, an API check shares some part of the initial configuration, but is organized around the `request` argument instead of the `script` argument:

```terraform
resource "checkly_check" "get-books" {
  name                      = "GET /books"   // The name of the check
  type                      = "API"     // The type of the check
  activated                 = true      // Whether the check will start as active on creation
  should_fail               = false     // Whether the check's HTTP response's status is expected to be >399
  frequency                 = 1         // The frequency of the check in minutes
  ssl_check                 = true      // Whether your SSL cert for the given domain should be checked too
  use_global_alert_settings = true      // Whether to use account level alert setting instead of the alert setting defined on this check
  run_parallel              = true            // Whether the check would run in a single location at time (round-robin) or all locations on each run

  locations = [                         // Which locations the check should run from (if not in a group)
    "us-west-1",
    "eu-central-1"
  ]
  
  retry_strategy = {                    // The retry strategy used for failed runs
    type = "LINEAR"
  }

  request {                             // All the settings for the check's HTTP request
    url              = "https://danube-web.shop/api/books"   // The request URL
    follow_redirects = true             // Whether the request should follow redirects
    skip_ssl         = false            // Whether to skip the SSL validation on the target server
    assertion {                         // One or more assertions to run against the HTTP response
      source     = "STATUS_CODE"        // What to assert against
      comparison = "EQUALS"             // How to assert
      target     = "200"                // Expected value
    }
  }
}
```

> You can see all the configuration options for checks on the official Terraform registry [documentation page](https://registry.terraform.io/providers/checkly/checkly/latest/docs/resources/check#schema).

### Heartbeat monitors

For example, a heartbeat monitor can look as follows:

```terraform
resource "checkly_check" "send-weekly-digest-v-2" {
  name                      = "Send Weekly Digest"    // The name of the monitor
  type                      = "HEARTBEAT"             // The type of the monitor
  activated                 = true                    // Whether the monitor will start as active on creation
  heartbeat {
    period                  = 7                       // The expected period of time between each ping. Between 30 seconds and 365 days.
    period_unit             = "days"                  // The unit of time for the period, the available options are `'seconds' | 'minutes' | 'hours' | 'days'`
    grace                   = 1                       // The grace period to wait for before sending an alert. Between 0 seconds and 365 days.
    grace_unit              = "days"                  // The unit of time for the grace period, the available options are `'seconds' | 'minutes' | 'hours' | 'days'`.
  }
  use_global_alert_settings = true                    // Whether to use account level alert setting instead of the alert setting defined on this check
}
```

Upon applying your terraform configuration changes, you will be returned a read-only key value for the heartbeat ping token. The token is stored in your `tfstate` file.


### Multistep checks

As with Browser checks, when constructing a Multistep check it is possible to provide the script directly in-line, 
but the recommended approach is to store scripts in separate files.

> Multistep checks are only supported on runtime 2023.09 or later. See [Runtimes](/docs/runtimes/) for more details.

For example, a Multistep check can look as follows:
```terraform
 resource "checkly_check" "e2e-shopping" {
  name                      = "Shopping Flow"
  type                      = "MULTI_STEP"
  activated                 = true
  should_fail               = false
  frequency                 = 1
  use_global_alert_settings = true
  locations = [
    "us-west-1",
    "eu-central-1"
  ]

  script = file("${path.module}/scripts/shopping.js")
}
```


### Tcp Monitors

TCP monitors verify the availability and responsiveness of services running on specific ports, ensuring they are reachable and operating within acceptable response time thresholds.

Below is a sample configuration for a TCP monitor using Terraform:

```terraform
resource "checkly_tcp_monitor" "example-tcp-monitor-2" {
  name                   = "Example TCP monitor 2"
  activated              = true
  should_fail            = true
  frequency              = 1
  degraded_response_time = 4000
  max_response_time      = 5000

  locations = [
    "us-west-1",
    "ap-northeast-1",
    "ap-south-1",
  ]

  alert_settings {
    escalation_type = "RUN_BASED"

    run_based_escalation {
      failed_run_threshold = 1
    }

    reminders {
      amount = 1
    }
  }

  retry_strategy {
    type                 = "FIXED"
    base_backoff_seconds = 60
    max_duration_seconds = 600
    max_retries          = 3
    same_region          = false
  }

  request {
    hostname = "api.checklyhq.com"
    port     = 80
    data     = "hello"

    assertion {
      source     = "RESPONSE_DATA"
      property   = ""
      comparison = "CONTAINS"
      target     = "welcome"
    }

    assertion {
      source     = "RESPONSE_TIME"
      property   = ""
      comparison = "LESS_THAN"
      target     = "2000"
    }
  }
}
```

## Groups

Once you start having more than just a handful of checks, it makes sense to start looking into [groups](/docs/groups/) to keep things tidy:

```terraform
resource "checkly_check_group" "key-shop-flows" {
  name      = "Key Shop Flows"  // The name of the group
  activated = true              // Whether the group will start as active on creation
  muted     = false             // Whether the group will start as muted on creation

  locations = [                 // Which locations the check should run from (if not in a group)
    "eu-west-1",
    "eu-central-1"
  ]

  concurrency               = 3     // How many checks to run at once when triggering the group using CI/CD triggers
  use_global_alert_settings = false // Whether to use global alert settings or group-specific ones
  run_parallel              = true  // Whether the check would run in a single location at time (round-robin) or all locations on each run

  retry_strategy = {                // The retry strategy used for failed runs
    type = "LINEAR"
  }
}
```

### Adding checks to groups

To add a check to a group, link it to that group by setting the `group_id` argument to the group's resource ID. For example, to link the API check and the group from this article:

```terraform
resource "checkly_check" "get-books" {
  name                      = "GET /books"
  type                      = "API"
  activated                 = true
  should_fail               = false
  frequency                 = 1
  ssl_check                 = true
  use_global_alert_settings = true

  locations = [
    "us-west-1",
    "eu-central-1"
  ]

  // This makes the check part of the group
  group_id = checkly_check_group.key-shop-flows.id

  retry_strategy = {
    type = "LINEAR"
  }

  request {
    url              = "https://danube-web.shop/api/books"
    follow_redirects = true
    skip_ssl         = false
    assertion {
      source     = "STATUS_CODE"
      comparison = "EQUALS"
      target     = "200"
    }
  }
}
```

> Locations and alert channel subscriptions defined at group level always trump the ones defined at check level. Double-check your config to make sure all checks are running from the intended regions and are set to alert on the correct channels.

You can see all the configuration options for groups, as well as more examples, on the official Terraform registry [documentation page](https://registry.terraform.io/providers/checkly/checkly/latest/docs/resources/check_group).
