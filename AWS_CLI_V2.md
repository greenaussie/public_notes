# AWS CLI v2

Note there are [breaking changes](https://docs.aws.amazon.com/cli/latest/userguide/cliv2-migration.html) for AWS CLI v1. The one which got me for a while before I found that page was the use `AWS_PAGER` which was messing with parsing of outputs from commands.

Add to `.bashrc`:

## Podman

```bash
# AWS CLI v2
alias aws='podman run --rm -it -e AWS_PAGER="" -e AWS_DEFAULT_REGION -e AWS_SECRET_ACCESS_KEY -e AWS_ACCESS_KEY_ID -e AWS_SESSION_TOKEN -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli'
complete -C aws_completer aws
```

## Docker

```bash
alias aws='docker run --rm -it -e AWS_PAGER="" -e AWS_DEFAULT_REGION -e AWS_SECRET_ACCESS_KEY -e AWS_ACCESS_KEY_ID -e AWS_SESSION_TOKEN -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli'
complete -C aws_completer aws
```
