# git基本命令
## 本地回退版本
### 根据commit id进行回退
在git平台查看commit id，以github为例：
![](images/2019-06-14-12-19-33.png)
执行操作
```ruby
E:\project\zhirui\zhirui-wms-user-pad>git reset --hard dbda016
HEAD is now at dbda016 提交
E:\project\zhirui\zhirui-wms-user-pad>git reset --hard ddc486
HEAD is now at ddc4868 提交
E:\project\zhirui\zhirui-wms-user-pad>git reset --hard refs/heads/master
HEAD is now at ddc4868 提交
```
### 根据当前版本回退到前N个历史版本
```ruby
E:\project\zhirui\zhirui-wms-user-pad>git reset --hard refs/heads/master^^
HEAD is now at 04cb694 手机端数量汇报仓储任务
E:\project\zhirui\zhirui-wms-user-pad>git reset --hard refs/heads/master^^
HEAD is now at a7b48a1 Merge branch 'master' of http://192.168.1.230:7000/r/zhirui-wms-user-pad
E:\project\zhirui\zhirui-wms-user-pad>git reset --hard refs/heads/master^^
HEAD is now at dbda016 提交
```

