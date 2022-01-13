# Setup Environment

Setting Kubectl autocomplete

**BASH**

```bash
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```

**Shorthand alias for kubectl**

```bash
alias k=kubectl
complete -F __start_kubectl k
```