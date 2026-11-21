# Synology L0CK3D Ransomware and Decryption / 群晖 L0CK3D 勒索恢复案例

[中文](#中文) · [English](#english) · [文档导航 / Documentation](#documentation)

## 中文

这是一次群晖 NAS 遭遇 L0CK3D 勒索后的脱敏技术复盘。案例在已有恢复密钥的前提下，记录如何应对存储空间不足、文件属性、运行中的服务和历史恢复残留，并说明逐文件恢复、验证及授权清理的方法。

- **已核实**：选定范围内，已识别的密文计数归零，获得授权的固定清单中 364 项全部清理。这些结果仅涵盖已检查的范围。
- **未执行**：将恢复数据通过 SMB 迁移到目标设备 NAS-B、核验迁移后的备份，以及将源设备恢复出厂设置。

本仓库提供文档，不提供可直接运行的解密器。脚本目录仅列出公开别名和用途，没有原始脚本、恢复密钥或原始日志。个人信息、真实路径、IP 地址、账户、设备名和原始文件名均已省略，必要处用 `*` 遮盖。NAS-A 和 NAS-B 分别是源设备和目标设备的公开代称。

## English

This redacted case study documents recovery from L0CK3D ransomware on a Synology NAS. The recovery key was already available. The case covers limited storage, file attributes, running services, and remnants of earlier recovery attempts. It describes the methods used to restore and verify files individually and to carry out authorized cleanup.

- **Verified:** The count of identified encrypted files fell to zero within the selected scope, and cleanup was completed for all 364 items on the authorized list. These findings cover only the scope that was inspected.
- **Pending:** The SMB transfer of recovered data to NAS-B, verification of that backup, and the factory reset of the source NAS were not performed.

This repository provides documentation only. It does not contain a runnable decryptor. The script catalog lists public aliases and their purposes. Original scripts, recovery keys, and raw logs are omitted. Personal information, actual paths, IP addresses, account names, device names, and original filenames are omitted or masked with `*`. NAS-A and NAS-B are public aliases for the source and destination devices.

<a id="documentation"></a>

## 文档导航 / Documentation

以下章节以中文编写，链接附英文标题。 The chapters below are written in Chinese, with English titles listed here.

| 中文 | English |
| --- | --- |
| [A · 背景、目标与证据范围](docs/A-context.md) | Context, objectives, and evidence scope |
| [B · 恢复过程与版本演进](docs/B-history.md) | Recovery timeline and version history |
| [C · 恢复原理与逐文件设计](docs/C-recovery-design.md) | Recovery design and per-file transactions |
| [D · 并发、续跑与验证](docs/D-concurrency-and-validation.md) | Concurrency, resuming work, and validation |
| [E · 收尾清理与结果](docs/E-cleanup-and-results.md) | Cleanup and verified results |
| [F · 脚本与核心模块目录](docs/F-script-catalog.md) | Script and core module catalog |
| [G · 迁移、备份核验与恢复出厂方案](docs/G-migration-and-reset.md) | Migration, backup verification, and factory reset plan |
| [H · 排查经验与改进建议](docs/H-lessons.md) | Lessons learned |
| [I · 脱敏规则与发布范围](docs/I-publication-and-redaction.md) | Redaction and publication scope |
| [J · 参考资料与证据说明](docs/J-references.md) | References and evidence notes |
| [K · 密钥获取、内存条件与代码示例](docs/K-memory-key-recovery.md) | Memory key recovery, evidence limits, and code examples |
| [公开发布检查记录](PUBLICATION_CHECK.md) | Publication checklist |
