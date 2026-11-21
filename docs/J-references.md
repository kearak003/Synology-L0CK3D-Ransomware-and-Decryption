# J．资料来源与证据说明

[返回总目录](../README.md)

## J.1 本案私有证据分组

| 证据别名 | 内容 | 支持的主要结论 |
| --- | --- | --- |
| 证据 E01 | 早期空间诊断、存储属性检查、单文件重建记录 | 普通复制和属性继承的影响；样本共享与延迟回收现象 |
| 证据 E02 | 恢复核心、启动器、合成自检和本地验证记录 | 实现的机制与各层验证能力；不单独证明已部署 |
| 证据 E03 | 指定真实样本和固定批次的运行记录 | 那些批次的已完成、保留和失败原因 |
| 证据 E04 | 多轮后继运行、进程检查和性能修正记录 | 来源采样、锁、权限、资源和报告写入等工程问题 |
| 证据 E05 | 后期正常结束的优先范围收尾报告 | 约 87 分钟、已确认通知和过程目录的处理数量，以及残留边界 |
| 证据 E06 | 最后一次定向只读核验报告 | 35 个过程目录的细分、327 个链接自身元信息、2 个内容命中的通知 |
| 证据 E07 | 固定清单删除报告与逐项日志 | 授权清单 364 项全部删除且最终复核不存在 |
| 证据 E08 | 用户后续迁移选择和已核对的产品操作资料 | SMB 迁移方案；没有提供迁移或重置完成证明 |

证据原件、文件名、摘要与逐项路径不在公开包中。私有索引用于本地复核，公开读者不能把本页当作可独立重放的数字取证报告。

## J.2 公开技术资料

### 文件系统共享与数据回收

- [Btrfs：去重机制与兼容条件](https://btrfs.readthedocs.io/en/latest/Deduplication.html)：说明共享重复区块、内核逐字节比较和存储属性兼容问题。它支持通用原理，不证明本案每个文件具备共享条件。
- [Linux 手册：FICLONE 与 FICLONERANGE](https://man7.org/linux/man-pages/man2/ioctl_ficlonerange.2.html)：说明独立文件间共享数据与写时复制语义。具体 NAS 内核和文件系统驱动仍需实际验证。

### 事件响应与恢复思路

- [Microsoft Incident Response 勒索事件响应方法](https://learn.microsoft.com/en-us/security/ransomware/incident-response-playbook-dart-ransomware-approach)：用于范围确认、遏制与调查的总体思路，不是本案的 NAS 解密工具。
- [公开社区救援项目](https://github.com/soimy/c3rb3r-rescue)：提供保全、内存材料和副本恢复相关实践。社区工具不代表商业保证，也不证明本案原地同池恢复策略是其推荐方式。

本案后期与社区常见建议的主要差异，是在空间约束下继续使用原存储池，并在明确授权和逐项验证后删除旧数据。独立介质上的密文副本恢复能减少这类事务与空间耦合，不能因为本案已经这样做过，就忽略更简单的独立输出方案。

### 数据迁移与系统重建

- [群晖：挂载远程 CIFS 共享](https://kb.synology.com/en-global/DSM/help/FileStation/mountremotevolume?version=7)。
- [群晖：共享文件夹及权限](https://kb.synology.com/en-af/DSM/help/DSM/AdminCenter/file_share_create?version=7)。
- [群晖：共享文件夹同步](https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/application_backupserv_sharedfoldersync?version=7)。
- [rsync 官方手册](https://download.samba.org/pub/rsync/rsync.1.html)。
- [群晖：系统重置](https://kb.synology.com/index.php/en-us/DSM/help/DSM/AdminCenter/system_restoredefaults?version=7)。

产品界面、支持能力和文档会随版本变化。本文中的迁移章节需要结合实际 NAS-B 路径、权限、可用空间和软件版本固定操作，不能把示意路径直接投入生产使用。

## J.3 使用这些资料的方式

公开资料用于解释通用语义和操作入口，本案结果以私有执行证据为准。其他勒索家族的解密器、不同版本的文件格式或普通 Linux 上的测试，不能直接替代当前样本与当前 NAS 的验证。

本文未转载第三方源码、原始取证材料或大段外部文章。后续若发布实际代码，需要另行核对其来源与许可。
