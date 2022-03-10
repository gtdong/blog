---
title:  Python调用ansible API系列
tags:
  - Python
  - Ansible
categories:
  - Python之路
  - Devops
date: 2018-08-25 13:05:00
---



# Python调用ansible API系列（一）获取资产信息

<!--more-->

你想让ansible工作首先就需要设置资产信息，那么我们如何通过使用Python调取Ansible的API来获取资产信息呢？

要提前准备一个hosts文件

![](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409151025966-1418351234.png)

## 获取组或者主机

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
from collections import namedtuple
# 核心类
# 用于读取YAML和JSON格式的文件
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager


# InventoryManager类的调用方式
def InventoryManagerStudy():
    dl = DataLoader()
    # loader= 表示是用什么方式来读取文件  sources=就是资产文件列表，里面可以是相对路径也可以是绝对路径
    im = InventoryManager(loader=dl, sources=["hosts"])

    # 获取指定资产文件中所有的组以及组里面的主机信息，返回的是字典，组名是键，主机列表是值
    allGroups = im.get_groups_dict()
    print(allGroups)

    # 获取指定组的主机列表
    print(im.get_groups_dict().get("test"))

    # 获取指定主机，这里返回的是host的实例
    host = im.get_host("172.16.48.242")
    print(host)
    # 获取该主机所有变量
    print(host.get_vars())
    # 获取该主机所属的组
    print(host.get_groups())


def main():
    InventoryManagerStudy()

if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```

![img](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409151136151-459511536.png)

## 获取变量

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
from collections import namedtuple
# 核心类
# 用于读取YAML和JSON格式的文件
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager

# VariableManager类的调用方式
def VariablManagerStudy():
    dl = DataLoader()
    im = InventoryManager(loader=dl, sources=["hosts"])
    vm = VariableManager(loader=dl, inventory=im)

    # 必须要先获取主机，然后查询特定主机才能看到某个主机的变量
    host = im.get_host("172.16.48.242")

    # 动态添加变量
    vm.set_host_variable(host=host, varname="AAA", value="aaa")
    # 获取指定主机的变量
    print(vm.get_vars(host=host))


def main():
    VariablManagerStudy()


if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```

![](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409151057032-257704423.png)

 



# Python调用ansible API系列（二）执行adhoc和playbook

## 执行adhoc

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
from collections import namedtuple
# 核心类
# 用于读取YAML和JSON格式的文件
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager
# 操作单个主机信息
from ansible.inventory.host import Host
# 操作单个主机组信息
from ansible.inventory.group import Group
# 存储执行hosts的角色信息
from ansible.playbook.play import Play
# ansible底层用到的任务队列
from ansible.executor.task_queue_manager import TaskQueueManager
# 核心类执行playbook
from ansible.executor.playbook_executor import PlaybookExecutor


def adhoc():
    """
    ad-hoc 调用
    资产配置信息  这个是通过 InventoryManager和VariableManager 定义
    执行选项 这个是通过namedtuple来定义
    执行对象和模块 通过dict()来定义
    定义play 通过Play来定义
    最后通过 TaskQueueManager 的实例来执行play
    :return:
    """
    # 资产配置信息
    dl = DataLoader()
    im = InventoryManager(loader=dl, sources=["hosts"])
    vm = VariableManager(loader=dl, inventory=im)
    # 执行选项，这个类不是ansible的类，这个的功能就是为了构造参数
    Options = namedtuple("Options", [
        "connection", "remote_user", "ask_sudo_pass", "verbosity", "ack_pass",
        "module_path", "forks", "become", "become_method", "become_user", "check",
        "listhosts", "listtasks", "listtags", "syntax", "sudo_user", "sudo", "diff"
    ])
    """
    这里就是Options的实例，然后你就可以赋值，这个为了给ansible设置执行选项 ansibile 172.16.48.171 -m shell -a 'ls /tmp' -f 5
    这里的选项就是ansible命令中 -f -C -D -m等执行选项
    """
    options = Options(connection='smart', remote_user=None, ack_pass=None, sudo_user=None, forks=5, sudo=None, ask_sudo_pass=False,
                      verbosity=5, module_path=None, become=None, become_method=None, become_user=None, check=False, diff=False,
                      listhosts=None, listtasks=None, listtags=None, syntax=None)
    # play的执行对象和模块，这里设置hosts，其实是因为play把play_source和资产信息关联后，执行的play的时候它会去资产信息中设置的sources的hosts文件中
    # 找你在play_source中设置的hosts是否在资产管理类里面。
    play_source = dict(name="Ansible Play",  # 任务名称
                       hosts="172.16.48.242",  # 目标主机，可以填写具体主机也可以是主机组名称
                       gather_facts="no",  # 是否收集配置信息

                       # tasks是具体执行的任务，列表形式，每个具体任务都是一个字典
                       tasks=[
                           dict(action=dict(module="shell", args="ls /tmp"))
                       ])
    # 定义play
    play = Play().load(play_source, variable_manager=vm, loader=dl)

    passwords = dict()  # 这个可以为空，因为在hosts文件中
    #
    tqm = TaskQueueManager(
        inventory=im,
        variable_manager=vm,
        loader=dl,
        options=options,
        passwords=passwords,
    )
    result = tqm.run(play)
    print(result)


def main():
    adhoc()


if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```

![](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409151624695-1987882561.png)

## 执行playbook

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
from collections import namedtuple
# 核心类
# 用于读取YAML和JSON格式的文件
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager
# 操作单个主机信息
from ansible.inventory.host import Host
# 操作单个主机组信息
from ansible.inventory.group import Group
# 存储执行hosts的角色信息
from ansible.playbook.play import Play
# ansible底层用到的任务队列
from ansible.executor.task_queue_manager import TaskQueueManager
# 核心类执行playbook
from ansible.executor.playbook_executor import PlaybookExecutor


def execplaybook():
    """
    调用 playbook
    调用playboo大致和调用ad-hoc相同，只是真正调用的是使用PlaybookExecutor
    :return:
    """
    # 资产配置信息
    dl = DataLoader()
    im = InventoryManager(loader=dl, sources=["hosts"])
    vm = VariableManager(loader=dl, inventory=im)
    # 执行选项，这个类不是ansible的类，这个的功能就是为了构造参数
    Options = namedtuple("Options", [
        "connection", "remote_user", "ask_sudo_pass", "verbosity", "ack_pass",
        "module_path", "forks", "become", "become_method", "become_user", "check",
        "listhosts", "listtasks", "listtags", "syntax", "sudo_user", "sudo", "diff"
    ])
    """
    这里就是Options的实例，然后你就可以赋值，这个为了给ansible设置执行选项 ansibile 172.16.48.171 -m shell -a 'ls /tmp' -f 5
    这里的选项就是ansible命令中 -f -C -D -m等执行选项
    """
    options = Options(connection='smart', remote_user=None, ack_pass=None, sudo_user=None, forks=5, sudo=None,
                      ask_sudo_pass=False,
                      verbosity=5, module_path=None, become=None, become_method=None, become_user=None, check=False,
                      diff=False,
                      listhosts=None, listtasks=None, listtags=None, syntax=None)
    passwords = dict()  # 这个可以为空，因为在hosts文件中
    #
    try:
        # playbooks参数里面放的就是playbook文件路径
        playbook = PlaybookExecutor(playbooks=["f1.yml"], inventory=im, variable_manager=vm, loader=dl, options=options, passwords=passwords)
        playbook.run()
    except Exception as err:
        print(err)


def main():
    execplaybook()

if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```

![](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409151901071-2038729072.png)

下图为f1.yml

![](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409151921422-1305806270.png)









# Python调用ansible API系列（三）带有callback的执行adhoc和playbook

在第二篇文章中虽然可以执行adhoc和playbook但是执行结果的输出并不是特别直观，虽然没有报错但是到底什么结果其实你是不知道的尤其是在执行adhoc的时候，这时候我们要利用callback来设置一下执行结果的输出。

## 执行adhoc

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from collections import namedtuple
# 核心类
# 用于读取YAML和JSON格式的文件
import sys
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager
# 存储执行hosts的角色信息
from ansible.playbook.play import Play
# ansible底层用到的任务队列
from ansible.executor.task_queue_manager import TaskQueueManager
# 状态回调，各种成功失败的状态
from ansible.plugins.callback import CallbackBase


class MyCallbackBase(CallbackBase):
    """
    通过api调用ac-hoc的时候输出结果很多时候不是很明确或者说不是我们想要的结果，主要它还是输出到STDOUT，而且通常我们是在工程里面执行
    这时候就需要后台的结果前端可以解析，正常的API调用输出前端很难解析。 对比之前的执行 adhoc()查看区别。
    为了实现这个目的就需要重写CallbackBase类，需要重写下面三个方法
    """
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)  # python3中重载父类构造方法的方式，在Python2中写法会有区别。
        self.host_ok = {}
        self.host_unreachable = {}
        self.host_failed = {}

    def v2_runner_on_unreachable(self, result):
        """
        重写 unreachable 状态
        :param result:  这是父类里面一个对象，这个对象可以获取执行任务信息
        """
        self.host_unreachable[result._host.get_name()] = result

    def v2_runner_on_ok(self, result, *args, **kwargs):
        """
        重写 ok 状态
        :param result:
        """
        self.host_ok[result._host.get_name()] = result

    def v2_runner_on_failed(self, result, *args, **kwargs):
        """
        重写 failed 状态
        :param result:
        """
        self.host_failed[result._host.get_name()] = result


def useMyCallbackBase():
    """
    这里通过调用ad-hoc来使用自定义callback
    :return:
    """
    dl = DataLoader()
    im = InventoryManager(loader=dl, sources=["hosts"])
    vm = VariableManager(loader=dl, inventory=im)
    Options = namedtuple("Options", [
        "connection", "remote_user", "ask_sudo_pass", "verbosity", "ack_pass",
        "module_path", "forks", "become", "become_method", "become_user", "check",
        "listhosts", "listtasks", "listtags", "syntax", "sudo_user", "sudo", "diff"
    ])
    options = Options(connection='smart', remote_user=None, ack_pass=None, sudo_user=None, forks=5, sudo=None,
                      ask_sudo_pass=False,
                      verbosity=5, module_path=None, become=None, become_method=None, become_user=None, check=False,
                      diff=False,
                      listhosts=None, listtasks=None, listtags=None, syntax=None)
    play_source = dict(name="Ansible Play",  # 任务名称
                       hosts="test",  # 目标主机，可以填写具体主机也可以是主机组名称
                       gather_facts="no",  # 是否收集配置信息

                       # tasks是具体执行的任务，列表形式，每个具体任务都是一个字典
                       tasks=[
                           dict(action=dict(module="shell", args="touch /tmp/bbb.txt", warn=False))
                       ])
    play = Play().load(play_source, variable_manager=vm, loader=dl)

    passwords = dict()  # 这个可以为空，因为在hosts文件中

    mycallback = MyCallbackBase()  # 实例化自定义callback

    tqm = TaskQueueManager(
        inventory=im,
        variable_manager=vm,
        loader=dl,
        options=options,
        passwords=passwords,
        stdout_callback=mycallback  # 配置使用自定义callback
    )
    tqm.run(play)
    # print(mycallback.host_ok.items())  # 它会返回2个东西，一个主机一个是执行结果对象
    # 定义数据结构
    result_raw = {"success": {}, "failed": {}, "unreachable": {}}
    # 如果成功那么  mycallback.host_ok.items() 才可以遍历，上面的任务肯定能成功所以我们就直接遍历这个
    for host, result in mycallback.host_ok.items():
        result_raw["success"][host] = result._result

    for host, result in mycallback.host_failed.items():
        result_raw["failed"][host] = result._result

    for host, result in mycallback.host_unreachable.items():
        result_raw["unreachable"][host] = result._result

    print(result_raw)


def main():
    useMyCallbackBase()

if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```

![](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409152302534-1572428882.png)

## 执行playbook

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
from collections import namedtuple
# 核心类
# 用于读取YAML和JSON格式的文件
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager
from ansible.executor.playbook_executor import PlaybookExecutor
# 状态回调，各种成功失败的状态
from ansible.plugins.callback import CallbackBase


class MyPlaybookCallbackBase(CallbackBase):
    """
    playbook的callback改写，格式化输出playbook执行结果
    """
    CALLBACK_VERSION = 2.0

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.task_ok = {}
        self.task_unreachable = {}
        self.task_failed = {}
        self.task_skipped = {}
        self.task_status = {}

    def v2_runner_on_unreachable(self, result):
        """
        重写 unreachable 状态
        :param result:  这是父类里面一个对象，这个对象可以获取执行任务信息
        """
        self.task_unreachable[result._host.get_name()] = result

    def v2_runner_on_ok(self, result, *args, **kwargs):
        """
        重写 ok 状态
        :param result:
        """
        self.task_ok[result._host.get_name()] = result

    def v2_runner_on_failed(self, result, *args, **kwargs):
        """
        重写 failed 状态
        :param result:
        """
        self.task_failed[result._host.get_name()] = result

    def v2_runner_on_skipped(self, result):
        self.task_skipped[result._host.get_name()] = result

    def v2_playbook_on_stats(self, stats):
        hosts = sorted(stats.processed.keys())
        for h in hosts:
            t = stats.summarize(h)
            self.task_status[h] = {
                "ok": t["ok"],
                "changed": t["changed"],
                "unreachable": t["unreachable"],
                "skipped": t["skipped"],
                "failed": t["failed"]
            }


def usecallbackplaybook():
    """
    调用 playbook
    调用playboo大致和调用ad-hoc相同，只是真正调用的是使用PlaybookExecutor
    :return:
    """
    dl = DataLoader()
    im = InventoryManager(loader=dl, sources=["hosts"])
    vm = VariableManager(loader=dl, inventory=im)
    Options = namedtuple("Options", [
        "connection", "remote_user", "ask_sudo_pass", "verbosity", "ack_pass",
        "module_path", "forks", "become", "become_method", "become_user", "check",
        "listhosts", "listtasks", "listtags", "syntax", "sudo_user", "sudo", "diff"
    ])
    options = Options(connection='smart', remote_user=None, ack_pass=None, sudo_user=None, forks=5, sudo=None,
                      ask_sudo_pass=False,
                      verbosity=5, module_path=None, become=None, become_method=None, become_user=None, check=False,
                      diff=False,
                      listhosts=None, listtasks=None, listtags=None, syntax=None)
    passwords = dict()  # 这个可以为空，因为在hosts文件中
    #
    try:
        playbook = PlaybookExecutor(playbooks=["f1.yml"], inventory=im, variable_manager=vm, loader=dl, options=options, passwords=passwords)
        callback = MyPlaybookCallbackBase()
        playbook._tqm._stdout_callback = callback  # 配置callback
        playbook.run()

        # print(callback.task_ok.items())
        result_raw = {"ok": {}, "failed": {}, "unreachable": {}, "skipped": {}, "status": {}}
        for host, result in callback.task_ok.items():
            result_raw["ok"][host] = result._result

        for host, result in callback.task_failed.items():
            result_raw["failed"][host] = result._result

        for host, result in callback.task_unreachable.items():
            result_raw["unreachable"][host] = result._result

        for host, result in callback.task_skipped.items():
            result_raw["skipped"][host] = result._result

        for host, result in callback.task_status.items():
            result_raw["status"][host] = result._result

        print(result_raw)
    except Exception as err:
        print(err)


def main():
    usecallbackplaybook()


if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```

![img](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409153602800-1185171651.png)

 



# Python调用ansible API系列（四）动态生成hosts文件

## 方法一：通过最原始的操作文件的方式

![img](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190409153932517-1418160854.png)

## 方法二：通过数据库或者调用其他API获取数据来动态获得

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
通过ansible API动态生成ansible资产信息但不产生实际的hosts文件
主机信息都可以通过数据库获得，然后生成指定格式，最后调用这个类来
生成主机信息。
"""

import sys
# 用于读取YAML和JSON格式的文件
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager
# 操作单个主机信息
from ansible.inventory.host import Host
# 操作单个主机组信息
from ansible.inventory.group import Group


class MyInventory:
    def __init__(self, hostsresource):
        """
        初始化函数
        :param hostsresource: 主机资源可以有2种形式
        列表形式: [{"ip": "172.16.48.171", "port": "22", "username": "root", "password": "123456"}]
        字典形式: {
                    "Group1": {
                        "hosts": [{"ip": "192.168.200.10", "port": "1314", "username": "root", "password": None}],
                        "vars": {"var1": "ansible"}
                    },
                    "Group2": {}
                }
        """
        self._hostsresource = hostsresource
        self._loader = DataLoader()
        self._hostsfilelist = ["temphosts"]
        """
        sources这个我们知道这里是设置hosts文件的地方，它可以是一个列表里面包含多个文件路径且文件真实存在，在单纯的执行ad-hoc的时候这里的
        文件里面必须具有有效的hosts配置，但是当通过动态生成的资产信息的时候这个文件必须存在但是它里面可以是空的，如果这里配置成None那么
        它不影响资产信息动态生成但是会有一个警告，所以还是要配置一个真实文件。
        """
        self._inventory = InventoryManager(loader=self._loader, sources=self._hostsfilelist)
        self._variable_manager = VariableManager(loader=self._loader, inventory=self._inventory)

        self._dynamic_inventory()

    def _add_dynamic_group(self, hosts_list, groupname, groupvars=None):
        """
        动态添加主机到指定的主机组

        完整的HOSTS文件格式
        [test1]
        hostname ansible_ssh_host=192.168.1.111 ansible_ssh_user="root" ansible_ssh_pass="123456"

        但通常我们都省略hostname，端口也省略因为默认是22，这个在ansible配置文件中有，除非有非22端口的才会配置
        [test1]
        192.168.100.10 ansible_ssh_user="root" ansible_ssh_pass="123456" ansible_python_interpreter="/PATH/python3/bin/python3"

        :param hosts_list: 主机列表 [{"ip": "192.168.100.10", "port": "22", "username": "root", "password": None}, {}]
        :param groupname:  组名称
        :param groupvars:  组变量，格式为字典
        :return:
        """
        # 添加组
        self._inventory.add_group(groupname)
        my_group = Group(name=groupname)

        # 添加组变量
        if groupvars:
            for key, value in groupvars.items():
                my_group.set_variable(key, value)

        # 添加一个主机
        for host in hosts_list:
            hostname = host.get("hostname", None)
            hostip = host.get("ip", None)
            if hostip is None:
                print("IP地址为空，跳过该元素。")
                continue
            hostport = host.get("port", "22")
            username = host.get("username", "root")
            password = host.get("password", None)
            ssh_key = host.get("ssh_key", None)
            python_interpreter = host.get("python_interpreter", None)

            try:
                # hostname可以不写，如果为空默认就是IP地址
                if hostname is None:
                    hostname = hostip
                # 生成一个host对象
                my_host = Host(name=hostname, port=hostport)
                # 添加主机变量
                self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_host", value=hostip)
                self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_port", value=hostport)
                if password:
                    self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_pass", value=password)
                self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_user", value=username)
                if ssh_key:
                    self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_private_key_file", value=ssh_key)
                if python_interpreter:
                    self._variable_manager.set_host_variable(host=my_host, varname="ansible_python_interpreter", value=python_interpreter)

                # 添加其他变量
                for key, value in host.items():
                    if key not in ["ip", "hostname", "port", "username", "password", "ssh_key", "python_interpreter"]:
                        self._variable_manager.set_host_variable(host=my_host, varname=key, value=value)

                # 添加主机到组
                self._inventory.add_host(host=hostname, group=groupname, port=hostport)
            except Exception as err:
                print(err)

    def _dynamic_inventory(self):
        """
        添加 hosts 到inventory
        :return:
        """
        if isinstance(self._hostsresource, list):
            self._add_dynamic_group(self._hostsresource, "default_group")
        elif isinstance(self._hostsresource, dict):
            for groupname, hosts_and_vars in self._hostsresource.items():
                self._add_dynamic_group(hosts_and_vars.get("hosts"), groupname, hosts_and_vars.get("vars"))

    @property
    def INVENTORY(self):
        """
        返回资产实例
        :return:
        """
        return self._inventory

    @property
    def VARIABLE_MANAGER(self):
        """
        返回变量管理器实例
        :return:
        """
        return self._variable_manager


def main():
    temphosts_list = [{"ip": "192.168.200.10", "port": "22", "username": "root", "password": "123456"}]

    temphosts_dict = {
        "Group1": {
            "hosts": [{"ip": "192.168.200.10", "port": "1314", "username": "root", "password": None}],
            "vars": {"var1": "ansible"}
        },
        # "Group2": {}
    }

    mi = MyInventory(temphosts_dict)
    # print(mi.INVENTORY.get_groups_dict())

    # for group, hosts in mi.INVENTORY.get_groups_dict().items():
    #     print(group, hosts)

    host = mi.INVENTORY.get_host("192.168.200.10")
    print(mi.VARIABLE_MANAGER.get_vars(host=host))

if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```

![img](https://gitee.com/gtdong/images/raw/master/blog/1448094-20190410101955130-26326171.png)











# Python调用ansible API系列（五）综合使用

如何把动态生成资产信息、执行playbook以及自定义结果结合起来用呢？

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
通过ansible API动态生成ansible资产信息但不产生实际的hosts文件
主机信息都可以通过数据库获得，然后生成指定格式，最后调用这个类来
生成主机信息。
"""

import sys
# 用于读取YAML和JSON格式的文件
from ansible.executor.playbook_executor import PlaybookExecutor
from ansible.parsing.dataloader import DataLoader
# 用于存储各类变量信息
from ansible.vars.manager import VariableManager
# 用于导入资产文件
from ansible.inventory.manager import InventoryManager
# 操作单个主机信息
from ansible.inventory.host import Host
# 操作单个主机组信息
from ansible.inventory.group import Group
# 状态回调，各种成功失败的状态
from ansible.plugins.callback import CallbackBase
from collections import namedtuple


class PlaybookCallResultCollector(CallbackBase):
    """
    playbook的callback改写，格式化输出playbook执行结果
    """
    CALLBACK_VERSION = 2.0

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.task_ok = {}
        self.task_unreachable = {}
        self.task_failed = {}
        self.task_skipped = {}
        self.task_status = {}

    def v2_runner_on_unreachable(self, result):
        """
        重写 unreachable 状态
        :param result:  这是父类里面一个对象，这个对象可以获取执行任务信息
        """
        self.task_unreachable[result._host.get_name()] = result

    def v2_runner_on_ok(self, result, *args, **kwargs):
        """
        重写 ok 状态
        :param result:
        """
        self.task_ok[result._host.get_name()] = result

    def v2_runner_on_failed(self, result, *args, **kwargs):
        """
        重写 failed 状态
        :param result:
        """
        self.task_failed[result._host.get_name()] = result

    def v2_runner_on_skipped(self, result):
        self.task_skipped[result._host.get_name()] = result

    # def v2_playbook_on_stats(self, stats):
    #     hosts = sorted(stats.processed.keys())
    #     for h in hosts:
    #         t = stats.summarize(h)
    #         self.task_status[h] = {
    #             "ok": t["ok"],
    #             "changed": t["changed"],
    #             "unreachable": t["unreachable"],
    #             "skipped": t["skipped"],
    #             "failed": t["failed"]
    #         }


class MyInventory:
    def __init__(self, hostsresource):
        """
        初始化函数
        :param hostsresource: 主机资源可以有2种形式
        列表形式: [{"ip": "172.16.48.171", "port": "22", "username": "root", "password": "123456"}]
        字典形式: {
                    "Group1": {
                        "hosts": [{"ip": "192.168.200.10", "port": "1314", "username": "root", "password": None}],
                        "vars": {"var1": "ansible"}
                    },
                    "Group2": {}
                }
        """
        self._hostsresource = hostsresource
        self._loader = DataLoader()
        self._hostsfilelist = ["temphosts"]
        """
        sources这个我们知道这里是设置hosts文件的地方，它可以是一个列表里面包含多个文件路径且文件真实存在，在单纯的执行ad-hoc的时候这里的
        文件里面必须具有有效的hosts配置，但是当通过动态生成的资产信息的时候这个文件必须存在但是它里面可以是空的，如果这里配置成None那么
        它不影响资产信息动态生成但是会有一个警告，所以还是要配置一个真实文件。
        """
        self._inventory = InventoryManager(loader=self._loader, sources=self._hostsfilelist)
        self._variable_manager = VariableManager(loader=self._loader, inventory=self._inventory)

        self._dynamic_inventory()

    def _add_dynamic_group(self, hosts_list, groupname, groupvars=None):
        """
        动态添加主机到指定的主机组

        完整的HOSTS文件格式
        [test1]
        hostname ansible_ssh_host=192.168.1.111 ansible_ssh_user="root" ansible_ssh_pass="123456"

        但通常我们都省略hostname，端口也省略因为默认是22，这个在ansible配置文件中有，除非有非22端口的才会配置
        [test1]
        192.168.100.10 ansible_ssh_user="root" ansible_ssh_pass="123456" ansible_python_interpreter="/PATH/python3/bin/python3"

        :param hosts_list: 主机列表 [{"ip": "192.168.100.10", "port": "22", "username": "root", "password": None}, {}]
        :param groupname:  组名称
        :param groupvars:  组变量，格式为字典
        :return:
        """
        # 添加组
        my_group = Group(name=groupname)
        self._inventory.add_group(groupname)

        # 添加组变量
        if groupvars:
            for key, value in groupvars.items():
                my_group.set_variable(key, value)

        # 添加一个主机
        for host in hosts_list:
            hostname = host.get("hostname", None)
            hostip = host.get("ip", None)
            if hostip is None:
                print("IP地址为空，跳过该元素。")
                continue
            hostport = host.get("port", "22")
            username = host.get("username", "root")
            password = host.get("password", None)
            ssh_key = host.get("ssh_key", None)
            python_interpreter = host.get("python_interpreter", None)

            try:
                # hostname可以不写，如果为空默认就是IP地址
                if hostname is None:
                    hostname = hostip
                # 生成一个host对象
                my_host = Host(name=hostname, port=hostport)
                # 添加主机变量
                self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_host", value=hostip)
                self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_port", value=hostport)
                if password:
                    self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_pass", value=password)
                self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_user", value=username)
                if ssh_key:
                    self._variable_manager.set_host_variable(host=my_host, varname="ansible_ssh_private_key_file", value=ssh_key)
                if python_interpreter:
                    self._variable_manager.set_host_variable(host=my_host, varname="ansible_python_interpreter", value=python_interpreter)

                # 添加其他变量
                for key, value in host.items():
                    if key not in ["ip", "hostname", "port", "username", "password", "ssh_key", "python_interpreter"]:
                        self._variable_manager.set_host_variable(host=my_host, varname=key, value=value)

                # 添加主机到组
                self._inventory.add_host(host=hostname, group=groupname, port=hostport)

            except Exception as err:
                print(err)

    def _dynamic_inventory(self):
        """
        添加 hosts 到inventory
        :return:
        """
        if isinstance(self._hostsresource, list):
            self._add_dynamic_group(self._hostsresource, "default_group")
        elif isinstance(self._hostsresource, dict):
            for groupname, hosts_and_vars in self._hostsresource.items():
                self._add_dynamic_group(hosts_and_vars.get("hosts"), groupname, hosts_and_vars.get("vars"))

    @property
    def INVENTORY(self):
        """
        返回资产实例
        :return:
        """
        return self._inventory

    @property
    def VARIABLE_MANAGER(self):
        """
        返回变量管理器实例
        :return:
        """
        return self._variable_manager


class AnsibleRunner(object):
    def __init__(self, hostsresource):
        Options = namedtuple("Options", [
            "connection", "remote_user", "ask_sudo_pass", "verbosity", "ack_pass",
            "module_path", "forks", "become", "become_method", "become_user", "check",
            "listhosts", "listtasks", "listtags", "syntax", "sudo_user", "sudo", "diff"
        ])
        self._options = Options(connection='smart', remote_user=None, ack_pass=None, sudo_user=None, forks=5, sudo=None,
                          ask_sudo_pass=False,
                          verbosity=5, module_path=None, become=None, become_method=None, become_user=None, check=False,
                          diff=False,
                          listhosts=None, listtasks=None, listtags=None, syntax=None)
        self._passwords = dict(sshpass=None, becomepass=None)  # 这个可以为空，因为在hosts文件中
        self._loader = DataLoader()
        myinven = MyInventory(hostsresource=hostsresource)
        self._inventory = myinven.INVENTORY
        self._variable_manager = myinven.VARIABLE_MANAGER

    def run_playbook(self, playbook_path, extra_vars=None):
        """
        执行playbook
        :param playbook_path: playbook的yaml文件路径
        :param extra_vars: 额外变量
        :return: 无返回值
        """
        try:
            if extra_vars:
                self._variable_manager.extra_vars = extra_vars
            playbook = PlaybookExecutor(playbooks=[playbook_path], inventory=self._inventory, variable_manager=self._variable_manager, loader=self._loader,
                                        options=self._options, passwords=self._passwords)
            # 配置使用自定义callback
            self._callback = PlaybookCallResultCollector()
            playbook._tqm._stdout_callback = self._callback
            # 执行playbook
            playbook.run()
        except Exception as err:
            print(err)

    def get_playbook_result(self):
        """
        获取playbook执行结果
        :return:
        """
        result_raw = {"ok": {}, "failed": {}, "unreachable": {}, "skipped": {}, "status": {}}
        for host, result in self._callback.task_ok.items():
            result_raw["ok"][host] = result._result

        for host, result in self._callback.task_failed.items():
            result_raw["failed"][host] = result._result

        for host, result in self._callback.task_unreachable.items():
            result_raw["unreachable"][host] = result._result

        for host, result in self._callback.task_skipped.items():
            result_raw["skipped"][host] = result._result

        for host, result in self._callback.task_status.items():
            result_raw["status"][host] = result._result

        return result_raw


def main():
    temphosts_list = [{"ip": "172.16.48.250", "port": "22", "username": "root", "password": "12qwaszx!@QW"}]

    temphosts_dict = {
        "Group1": {
            "hosts": [{"ip": "192.168.200.10", "port": "1314", "username": "root", "password": None}],
            "vars": {"var1": "ansible"}
        },
        # "Group2": {}
    }

    # mi = MyInventory(temphosts_list)
    # for group, hosts in mi.INVENTORY.get_groups_dict().items():
    #     print(group, hosts)
    # host = mi.INVENTORY.get_host("192.168.200.10")
    # print(mi.VARIABLE_MANAGER.get_vars(host=host))

    ar = AnsibleRunner(temphosts_list)
    ar.run_playbook("/Users/rex.chen/PycharmProjects/IDCMigration/AnsibleStudy/f1.yml")
    print(ar.get_playbook_result())

if __name__ == "__main__":
    try:
        main()
    finally:
        sys.exit()
```
