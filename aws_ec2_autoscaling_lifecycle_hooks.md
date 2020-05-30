# AWS EC2 Autoscaling Lifecycle Hooks

Richard Green 2020 - Use info below at your own risk.

A short note on a simple way to use lifecycle hooks to signal an `AWS::AutoScaling::AutoScalingGroup` resource so the EC2 intance it creates will only be attached at a load balancer if the Cfn-init process has successfully completed.

## Defining the hook

The hook is defined on the `AWS::AutoScaling::AutoScalingGroup` resource:

```yml
      LifecycleHookSpecificationList:
        -
          DefaultResult: ABANDON
          HeartbeatTimeout: "900"
          LifecycleHookName: cfn_init
          LifecycleTransition: autoscaling:EC2_INSTANCE_LAUNCHING
```

The lifecycle hook is created when the EC2 instance launches, with an expectation the instance will have launched within 15 minutes.

## IAM Permissions

The instance is going to act directly upon the lifecycle hooks, so it needs a policy for the  `ec2.amazonaws.com` service granting permission to carry out some API actions. The name of the hooks are determined by the name of the `AWS::AutoScaling::AutoScalingGroup`, so if (as here) that is not names, the policy is relaxed (see note at end of the page).

Policy snippet applied to the `AWS::IAM::Role attached to the Instance when it is launched:

```yml
        -
          PolicyName: "lifecycle-actions"
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              -
                Effect: "Allow"
                Action:
                  - "autoscaling:DescribeAutoScalingInstances"
                  - "autoscaling:DescribeLifecycleHooks"
                  - "autoscaling:CompleteLifecycleAction"
                Resource: "*"
```

## Cfn-init

The last action of Cfn-int actions defined within the `AWS::EC2::LaunchTemplate` resource is to learn the name of the lifecycle hook, if it exists, and complete the lifecycle action.

```yml
        90_manage_lifecycle_hook:
          commands:
            01_remove_lifecycle_hook_if_exists:
              command: !Sub |
                if [ "$(aws autoscaling describe-lifecycle-hooks --auto-scaling-group-name $(aws autoscaling describe-auto-scaling-instances --instance-ids $(curl -s http://169.254.169.254/latest/meta-data/instance-id) --region ${AWS::Region} --query AutoScalingInstances[].AutoScalingGroupName --output text) --lifecycle-hook-names cfn_init --region ${AWS::Region} --query LifecycleHooks[0].LifecycleHookName --output text)" == "cfn_init" ]
                then
                  echo "lifecycle hook cfn_init exists"
                  if [ "$(aws autoscaling describe-auto-scaling-instances --instance-ids $(curl -s http://169.254.169.254/latest/meta-data/instance-id) --region ${AWS::Region} --query AutoScalingInstances[0].LifecycleState --output text)" == "Pending:Wait" ]
                  then
                    echo "instance-id: $(curl -s http://169.254.169.254/latest/meta-data/instance-id) pending. Sending autoscaling complete-lifecycle-action"
                    aws autoscaling complete-lifecycle-action --auto-scaling-group-name $(aws autoscaling describe-auto-scaling-instances --instance-ids $(curl -s http://169.254.169.254/latest/meta-data/instance-id) --region ${AWS::Region} --query AutoScalingInstances[].AutoScalingGroupName --output text) --lifecycle-hook-name cfn_init --lifecycle-action-result CONTINUE --instance-id $(curl -s http://169.254.169.254/latest/meta-data/instance-id) --region ${AWS::Region}
                  fi
                fi
```

## Putting it together

If Cfn-init fails in an earlier step, we don't reach the last action, and hook time-out occurs, and the launch fails.

Earlier steps of Cfn-init could potenially 'self-check' a service availabilty.

Completion of Cfn-init may not in itself mean a service is built. The `AWS::AutoScaling::AutoScalingGroup` resource might also require a `HealthCheckGracePeriod`.

## Balancing security with functionality

Naming the `AWS::AutoScaling::AutoScalingGroup` would avoid the discovery process to learn the name of the hook, and thus the IAM policy granting `autoscaling:CompleteLifecycleAction` priviliges could be restricted further.
