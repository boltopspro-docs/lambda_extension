<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/lambda_extension/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Lambda Extension

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This is a [lono extension](https://lono.cloud/docs/extensions/) that provides lambda related resources like permission, IAM role, lambda layers, security group, events rule.

## Installation

Add this line to the Gemfile:

    gem "lambda_extension", git: "git@github.com:boltopspro/lambda_extension.git"

And then execute:

    bundle

Note, lono will auto-materialize and download the extension for blueprints that use the extension. So you don't have to necessarily add the extension your Gemfile unless you want to lock it down to a specific version.

## Usage

There are 2 general ways to use this lono extensions:

1. implicit approach
2. explicit approach

With the implicit approach, you only add a few lines to  your template code:

app/blueprints/demo/templates/demo.rb:

```ruby
extend_with "lambda_extension"
lambda_resources # Comment shows what lambda_resources creates. Refer to extension for latest.
# lambda_function
# lambda_permission # who can invoke lambda
# lambda_execution_role # what lambda can do
# lambda_layer
# lambda_security_group
# lambda_events_rule
```

Here, you won't have to do anything if the extension is updated. The comment is recommended to make it more transparent as to what is going on.

To be more explicit about the resources, in your template code add:

app/blueprints/demo/templates/demo.rb:

```ruby
extend_with "lambda_extension"
lambda_function
lambda_permission # who can invoke lambda
lambda_execution_role # what lambda can do
lambda_layer
lambda_security_group
lambda_events_rule
```

Here's the the same thing with the explicit declarations. There's more control with this approach. Though, you'll need to update the code above if helpers are added to the extension.

### Lambda Function Code

The lambda function code is expected to reside in `app/files/lambda-function` in your own blueprint. Example:

app/files/lambda-function/index.rb:

```ruby
require 'bundler/setup'
Bundler.require
require 'json'

def lambda_handler(event:, context:)
  puts "event: #{JSON.dump(event)}"
  {message: "hello world"}
end
```

You should also have `Gemfile` as it is used to build the Lambda layer.

app/files/lambda-function/Gemfile:

```ruby
source "https://rubygems.org"
gem "s3-secure" # just an selected example gem. need at least one to build a Lambda Layer
```

## Configure Details

### Schedule Expression

The default `ScheduleExpression` is `cron(15 10 * * ? *)`.  You can override this with `@schedule_expression`.  Example:

configs/log-group-cleaner/variables/development.rb:

```ruby
@schedule_expression = "rate(1 minute)"
```

The [ScheduledExpression](https://docs.aws.amazon.com/eventbridge/latest/userguide/scheduled-events.html) docs are helpful.

### Lambda Function in VPC

To configure the Lambda function with VpcConfig set `@subnet_ids` and either `@vpc_id` or `@security_group_ids`.

* When the `@vpc_id` is set, the template creates a managed security group for you and the Lambda function is configured to use that security group.
* When `@security_group_ids` is set, the Lambda function will use those existing security groups.
* The subnet must be a private subnet with configured with a NAT.

Here's an example of the managed security group.

configs/log-group-cleaner/variables/development.rb:

```ruby
@subnet_ids = ["subnet-111"]
@vpc_id = "vpc-111"
```

For Lambda VPC to work, the subnet must be a private subnet configured with a NAT.

Note, Lambda functions configured with VPCs may take much longer to deploy, typically 30-45 minutes. This is because Lambda creates and attaches an ENI to the Lambda function to make the VPC feature possible. If the function is deleted or updated, requiring replacement, the ENI takes 30-45m to be removed. Because of this, it is recommended to write code for your Lambda function code without the VpcConfig first. Get it working and then add VpcConfig at the end.

### X-Ray Tracing

Lambda X-Ray tracing is set to `Active` by default. You can disable this by setting `@tracing_config_mode = false`. Example:

configs/log-group-cleaner/variables/development.rb:

```ruby
@tracing_config_mode = false
```

You can also change the mode with the same `@tracing_config_mode` variable:

```ruby
@tracing_config_mode = "Active" # or "PassThrough"
```

