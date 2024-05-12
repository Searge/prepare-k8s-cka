# Tips and Tricks

## Usefull aliases

```bash
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kcuc='kubectl config use-context'
```

## Usefull variables

```bash
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
```

## Autocompletion

```bash
# add autocomplete permanently to your bash shell.
echo ". <(kubectl completion bash)" >> ~/.bashrc

# source bashrc
. ~/.bashrc
```

## Vim configuration

```vim:~/.vimrc
" set tab size to 2 spaces and use spaces instead of tabs
set ts=2   " number of spaces a tab is counted for
set sw=2   " number of spaces to use for autoindent
set et     " use spaces instead of tabs
set ai     " autoindent

" toggle paste mode
set pastetoggle=<F3>
```

## Cheat Sheet

```bash
# Get the documentation of the resource and its fields
kubectl explain pods

# Get all the fields in the resource
kubectl explain pods --recursive

# Get the documentation of the field in the resource
kubectl explain pods.spec.containers

# Get the documentation of the field in the resource
kubectl explain pods.spec.containers.image
```

> Reaf about [required fields](required_fields_in_manifests.md) in the resource and their values.
