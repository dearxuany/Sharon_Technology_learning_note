# ansible 简介
4 大主流的组态管理工具 (Puppet, SaltStack, Chef, Ansible) ，Python 阵营的组态管理工具</br>
Puppet 和 Chef 基于 Ruby，SaltStack 和 Ansible 基于 Python</br>

## ansible 原理
ansible 通过 inventory 档案来定义有哪些 Managed node (被控端)，并借由 SSH 和 Python 进行沟通。</br>
只需在 Control Machine 里安装 Ansible 即可，通过SSH被连上的机器里有预载 Python 时，Ansible 就可以运作。</br>

