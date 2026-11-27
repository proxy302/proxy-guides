# Ansible 让远程主机走带账号密码的代理，怎么配？

在 play、block 或 task 上加 `environment:` 关键字，写 `http_proxy`、`https_proxy`、`no_proxy` 三个变量，值是 `http://USER:PASS@HOST:PORT`；远程主机上执行的模块（`package`、`get_url`、`pip` 等）就会带着这些变量运行。密码放在用 `ansible-vault` 加密的变量文件里，不要明文写进 playbook。apt、dnf 这类包管理器最好再用模板写一份它们自己的代理配置，系统以后手动装包也能走代理。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP IP 1 天 0.5 美元、30 天 7.99 美元，动态住宅 1.5 美元/GB 起。

*内容对照 Ansible 官方文档「Setting the remote environment」核对，没有实机运行。*

## 第一步：把代理信息放进加密变量

```bash
ansible-vault encrypt_string 'PASS' --name 'proxy_pass'
```

把输出贴进 `group_vars/all/proxy.yml`：

```yaml
# group_vars/all/proxy.yml
proxy_user: USER
proxy_pass: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  （encrypt_string 输出的密文）
proxy_hostport: "HOST:PORT"

proxy_url: "http://{{ proxy_user }}:{{ proxy_pass | urlencode }}@{{ proxy_hostport }}"

proxy_env:
  http_proxy: "{{ proxy_url }}"
  https_proxy: "{{ proxy_url }}"
  no_proxy: "localhost,127.0.0.1,.internal.example"
```

- `urlencode` 会把密码里的 `@`、`:` 等字符编码成 `%40`、`%3A`，但不编码 `/`。密码里有 `/` 的话，换一个没有 `/` 的密码最省事。
- 运行时加 `--ask-vault-pass` 或 `--vault-password-file`。
- USER、PASS、HOST、PORT 是占位符。

## 第二步：在 play 或 task 上用它

整个 play 都走代理：

```yaml
- hosts: web
  become: true
  environment: "{{ proxy_env }}"
  tasks:
    - name: 安装 nginx
      ansible.builtin.package:
        name: nginx
        state: present

    - name: 下载安装包
      ansible.builtin.get_url:
        url: https://example.com/tool.tar.gz
        dest: /tmp/tool.tar.gz
```

只让某一个 task 走代理：

```yaml
    - name: 用 pip 装依赖
      ansible.builtin.pip:
        name: requests
      environment: "{{ proxy_env }}"
```

官方文档里的写法也是这样：task 级、play 级都可以写 `environment:`，常用的做法是把代理变量定义成一个字典，放在 `group_vars` 里给多个 playbook 复用。

## 第三步：给包管理器写一份自己的配置

`environment:` 只在这次 Ansible 执行模块时生效。服务器以后有人手动 `apt install`、或者定时任务在跑，都用不到它。包管理器的代理写进它自己的配置文件更稳：

```yaml
    - name: apt 代理（Debian、Ubuntu）
      ansible.builtin.copy:
        dest: /etc/apt/apt.conf.d/95proxy
        mode: "0600"
        content: |
          Acquire::http::Proxy "{{ proxy_url }}/";
          Acquire::https::Proxy "{{ proxy_url }}/";
      when: ansible_facts['os_family'] == 'Debian'
      no_log: true
```

`no_log: true` 让这一步的内容不打进 Ansible 输出。dnf 的写法（`proxy=`、`proxy_username=`、`proxy_password=`）见 [apt、dnf 怎么走代理](../apt-dnf-proxy-zh/README.md)。

## 官方文档提醒的几点

| 规则 | 意思 |
|---|---|
| 作用范围 | play 或 block 上设的变量，只对其中由同一个用户执行的 task 有效 |
| 不影响 Ansible 自己 | 不改 Ansible 本身和它的配置，也不影响 lookup、filter 这类插件 |
| 不会自动变成 facts | 想让 facts 也带上，要单独写一个带 `environment:` 的 `gather_facts` 任务 |
| 明文传输 | 环境变量一般是明文传过去的，取决于 shell 插件，所以文档不建议拿它传机密 |

最后一条的意思是：密码在 vault 里是加密的，但运行时会以环境变量的形式出现在远程主机的进程里。能接受这一点再用；更在意的话，只给需要的 task 加 `environment:`，并用专门的代理账号。

## 运行 Ansible 的那台电脑要走代理时

`ansible-galaxy collection install`、`delegate_to: localhost` 的任务、各种 lookup，都是在你自己运行 Ansible 的那台电脑（控制机）上跑，`environment:` 管不到。直接在终端里设：

```bash
export https_proxy="http://USER:PASS@HOST:PORT"
export http_proxy="$https_proxy"
ansible-galaxy collection install community.general
```

## 先确认代理本身可用

在一台远程主机上：

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

能返回 JSON、国家是你买的，再去查 playbook。买 IP 时选 HTTP 协议，apt、dnf、pip、get_url 都按 HTTP 代理用最省事；协议下单时选定，一个 IP 只有一种。

## 常见问题排查

| 现象 | 原因 |
|---|---|
| 装包超时，像是没走代理 | `environment:` 写在了别的 play 上，或者包管理器读的是自己的配置文件、里面没配 |
| 407 | 账号密码错，或密码特殊字符没编码 |
| 内网源、内网服务连不上 | 也被送去了代理，加进 `no_proxy` |
| `ansible-galaxy` 下载失败 | 它在控制机上跑，要在本机终端设代理 |
| 昨天还能用，今天连不上 | 静态 IP 到期被释放了 |

## 常见问题

**一批服务器要不要每台一个 IP？** 不需要。只是下载软件包，一批机器共用一个出口没问题；如果对方按 IP 白名单放行，给它们一个固定的静态 IP 去加白名单。

**动态还是静态？** 只是装包、下文件，动态住宅（1.5 美元/GB 起）或动态机房（0.4 美元/GB）就够；需要固定出口时用静态 IP。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（1 到 90 天，下单时选 HTTP 或 SOCKS5，所有国家同价）和动态住宅、动态机房流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。相关：[Docker 怎么走代理](../docker-proxy-zh/README.md)、[需要 IP 白名单的接口用什么代理](../fixed-ip-for-api-allowlist/README.md)。

*Facts checked 27 November 2026.*
