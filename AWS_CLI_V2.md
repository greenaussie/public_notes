# AWS CLI v2

Add to `.bashrc`:

## Podman

```bash
# AWS CLI v2
alias aws='podman run --rm -it -e AWS_DEFAULT_REGION -e AWS_SECRET_ACCESS_KEY -e AWS_ACCESS_KEY_ID -e AWS_SESSION_TOKEN -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli'
complete -C aws_completer aws
```

## Docker

```bash
alias aws='docker run --rm -it -e AWS_DEFAULT_REGION -e AWS_SECRET_ACCESS_KEY -e AWS_ACCESS_KEY_ID -e AWS_SESSION_TOKEN -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli'
complete -C aws_completer aws
```
