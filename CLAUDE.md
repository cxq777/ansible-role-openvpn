# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

## 项目概述

这是一个用于安装和配置 OpenVPN 服务器或客户端的 Ansible 角色。该角色支持服务器和客户端配置，并使用 Easy-RSA 进行自动证书管理。

## 架构说明

### 角色结构
- `tasks/main.yml` - 主入口点，导入断言并委托给服务器/客户端任务
- `tasks/server.yml` - OpenVPN 服务器设置，包括 PKI 初始化、证书生成和配置
- `tasks/client.yml` - 客户端证书生成和 .ovpn 文件创建
- `tasks/assert.yml` - 变量验证和断言
- `defaults/main.yml` - 默认变量定义
- `vars/main.yml` - 针对不同操作系统的变量映射（包、路径、服务、用户组）
- `templates/` - server.conf 和 client.ovpn 配置文件的 Jinja2 模板
- `handlers/main.yml` - 服务重启处理器
- `bin/` - OpenVPN 管理工具脚本（ovpn_genconfig、ovpn_getclient 等）

### 核心组件
- **角色模式**: 由 `openvpn_role` 变量控制（server/client）
- **证书管理**: 使用 Easy-RSA 进行 PKI 操作，证书存储在 `/etc/openvpn/server/{service_name}/easy-rsa/pki/`
- **操作系统兼容性**: 支持企业级 Linux、Debian 和 Ubuntu，具有特定于操作系统的变量映射
- **服务管理**: 具有操作系统感知的 systemd 服务名称和配置路径

## 开发命令

### 测试命令
```bash
# 运行 molecule 测试（需要 Docker）
molecule test

# 使用特定操作系统矩阵运行测试
image=debian tag=latest molecule test
image=ubuntu tag=focal molecule test

# 仅运行代码检查
molecule lint
# 或单独运行检查工具
yamllint .
ansible-lint
```

### CI/CD 流程
项目使用 GitHub Actions，在多个 Python 版本（3.9、3.10、3.13）和操作系统组合上进行 molecule 测试。测试在推送到 master 分支和拉取请求时自动运行。

## 配置选项

### 服务器配置
```yaml
openvpn_role: server
server_port: 1194
server_subnet: 10.9.0.0 255.255.255.0
service_name: default
extra_conf: ""
```

### 客户端配置
```yaml
openvpn_role: client
openvpn_client_server: vpn.example.com
cn: "client"
```

### 操作系统特定行为
- 包名称、服务名称和路径根据操作系统系列和版本而异
- `vars/main.yml` 中的变量自动处理这些差异
- Easy-RSA 路径在 Debian（/usr/share/easy-rsa）和其他系统（/usr/share/easy-rsa/3）之间有所不同