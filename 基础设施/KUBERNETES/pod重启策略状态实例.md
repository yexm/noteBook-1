# Pod重启策略状态实例

- Pod状态Running，运行一个container。Container成功终止。
    - 日志记录completion event
    - 如果重启策略为：
        - Always: 重启Container；Pod phase 保持Running
        - OnFailure： Pod phase转换为Succeeded
        - Never： Pod phase转换为Succeeded

- Pod状态Running，运行一个container。Container异常终止。
    - 日志记录failure event
    - 如果重启策略为：
        - Always: 重启Container；Pod phase 保持Running
        - OnFailure： 重启Container；Pod phase 保持Running
        - Never： Pod phase转换为Failed

- Pod状态Running，运行两个container。一个Container异常终止。
    - 日志记录failure event
    - 如果重启策略为：
        - Always: 重启Container；Pod phase 保持Running
        - OnFailure： 重启Container；Pod phase 保持Running
        - Never： 不重启Container， Pod phase 保持Running
    - 如果container1 不是running的，container2 退出：
        - Always: 重启Container；Pod phase 保持Running
        - OnFailure： 重启Container；Pod phase 保持Running
        - Never： Pod phase 变为Faild

- Pod状态Running并且有一个containerOOM
    - Container terminates in failure.
    - 日志记录OOM event
    - 如果重启策略为：
        - Always: 重启Container；Pod phase 保持Running
        - OnFailure： 重启Container；Pod phase 保持Running
        - Never： 报告failure事件， Pod phase 变为Failed

- Pod状态Running，磁盘异常
    - 杀死所有container
    - 报告appropriate事件
    - Pod phase变为Failed
    - 如果pod在controller控制下，将在其他位置重启

- Pod状态Running，所在node节点被剥离
    - Node Controller等待超时时间
    - Node Controller设置Pod phase为Failed
    - 如果pod在controller控制下，将在其他位置重启