# Deployment Strategies

Vagrant Orchestrate supports several deployment strategies, including parallel and
blue-green. Here we'll cover how to use the various strategies as well as describing
situations when each might be useful.

## Specifying a strategy

### Command line

Strategies can be passed on the command line with the `--strategy` parameter

    $ vagrant orchestrate push --strategy parallel

### Vagrantfile configuration

Alternatively, you can specify the deployment strategy in your Vagrantfile

    config.orchestrate.strategy = :parallel

Command line parameters take precedence over Vagrantfile set configuration values.

## Strategies

### Serial (default)
This will deploy to the target servers one at a time. This can be useful if you
have a small number servers, or if you need to keep the majority of your servers
online in order to support your application's load.

    $ vagrant orchestrate push --strategy serial

    config.orchestrate.strategy = :serial


### Parallel
This strategy will deploy to all of the target servers at the same time. This is
useful if you want to minimize the amount of time that an overall deployment takes.
Depending on how you've written your provisioners, this could cause downtime for
the application that is being deployed.

    $ vagrant orchestrate push --strategy parallel

    config.orchestrate.strategy = :parallel

### Canary
The canary strategy will deploy to a single server, provide an opportunity to
test the deployed server, and then deploy the remainder of the servers in parallel.
This is a great opportunity to test one node of your cluster before blasting your
changes out to them all. This can be particularly useful when combined with post
provision [trigger](https://github.com/emyl/vagrant-triggers) to run a smoke test.

    $ vagrant orchestrate push --strategy canary

    config.orchestrate.strategy = :canary

### Blue Green
The [Blue Green](http://martinfowler.com/bliki/BlueGreenDeployment.html) deployment
strategy deploys to half of the cluster in parallel, then the other half, with
a pause in between. This doesn't manage any of your load balancing or networking
configuraiton for you, but if your application has a healthcheck that your load
balancer respects, it should be easy to turn it off at the start of your provisioning
and back on at the end. If your application can serve the load on half of its nodes
then this will be the best blend of getting the deployment done quickly and maintaining
a working application.

    $ vagrant orchestrate push --strategy blue_green

    config.orchestrate.strategy = :blue_green

### Canary Blue Green
This strategy simply combines the two immediately above - deploying to a single
server, pausing, then to half the cluster in parallel, pausing, and then the remainder,
also in parallel. Good if you have a large number of servers and want to do a
smoke test of a single server before committing to pushing to half of your farm.

    $ vagrant orchestrate push --strategy canary_blue_green

		config.orchestrate.strategy = :canary_blue_green

## Suppressing Prompts
In order to automate the deployment process, it can be very useful to suppress
prompts. You can achieve that in two ways:

From the command line, add the `--force` or `-f` parameter

    $ vagrant orchestrate push --strategy canary -f


Within your vagrantfile, set the `force_push` setting to true

    config.orchestrate.force_push = true