# K．密钥获取：现场条件、内存材料与验证

[返回总目录](../README.md)

## K.1 先说明能确认到哪一步

本案例使用的是受影响设备自身进程保留的 **48 字节会话材料**。材料先保存为私有文件，后续恢复程序从文件加载。它包含 `IV16 + AES-key32`，用于解开文件尾部的每文件密钥记录。没有取得攻击者的 RSA 私钥，也没有证据表明这份材料能恢复另一台受害设备。

本章补查了早期会话中读回的盘点报告、进程保留记录、启动参数以及仍保存的恢复源码。原始采集任务文件和证据归档曾出现在当时的目录列表中；**这次没有取得最初采集脚本的完整正文、完整内存映射集或初始逆向记录**。因此，下面明确区分现场事实、机制解释和新编写的示例。不会把示例宣称为当年逐字执行过的脚本。

所有真实 PID、启动计数、指针地址、摘要、设备地址、任务名、目录名和密钥字节均以 `*` 遮盖或用 A、B、C 代替。技术尺寸和算法名称保留。

| 证据 | 可确认的内容 | 不能由此推断的内容 |
| --- | --- | --- |
| 早期目录列表 | 存在采集任务 A、证据目录 B 和归档 C | 不能仅凭归档大小宣称采集完整 |
| 早期盘点报告 | 会话材料长度为 48 字节；记录了内存指针 `*`、私有输出文件 A 和 SHA-256 `*` | 未复得采集正文，无法断言具体使用哪个系统调用、扫描了哪些映射 |
| 后续恢复入口 | 持续从同一私有材料文件加载，按长度、权限及固定摘要核对 | 文件名相同本身不能证明内容相同 |
| 进程身份与暂停记录 | 保存材料的进程 P 仍存在；后续处置保留其停止状态，恢复程序另行运行 | 不能补写成“已经证明最初采集时每个线程都停止” |
| 恢复源码与样本验证记录 | 材料拆为 IV 和 AES 密钥，解开尾部记录，再执行内容恢复与验证 | 不能由一次格式检查宣布整台设备都使用同一会话材料 |

## K.2 当时为什么还有获取机会

进入现存恢复记录时，NAS-A 仍在运行，目标进程 P 尚未退出。处置随后一直保留 P 的内存状态：没有把它当作普通恢复程序终止，也没有为修复空间和并发问题而重启设备。后续恢复脚本还绑定 P 的启动计数及可执行文件摘要，防止把 PID 被重用后的其他进程认作原进程。

这种条件给会话材料保全留下了机会。停止执行的进程仍可能保有堆和全局变量；终止、重新执行或重启设备会失去原来的进程地址空间。材料也可能已被清零或覆盖，所以“未重启”不能保证必然取到密钥。

可由现存记录确认的顺序是：

1. **保有现场**：目标进程及其内存状态仍在，存在早期采集任务及证据归档记录。
2. **材料落盘**：早期盘点报告明确记录 48 字节候选、指针、输出文件和摘要。
3. **验证并使用**：恢复程序读取私有材料，解析文件尾部，在样本上验证恢复结果，再进入后续批量流程。
4. **继续保全**：暂停旧恢复程序、处理后台服务或优化并发时，单独保留 P；实际恢复已不要求每个文件再次读取 P 的内存。

最初发现 P 的人工判断、第一条暂停命令、首次读取内存所用接口及第一份样本的完整记录不在本次复得的材料内。这里保留这些缺口，不用其他团队的案例替代本案事实。

## K.3 从内存到可用材料，具体做什么

机制上需要先确定**目标进程、样本版本和材料位置**，再读取候选，最后验证。不是在所有内存里随意截取 48 字节。

### 定位：全局指针与实际缓冲区有区别

对同类已分析样本，初始化代码生成会话材料，并通过全局指针引用堆上的缓冲区。静态分析给出的是指针槽位及其含义；运行时必须读取槽位、解析指针，再在该进程的可读映射中寻找实际材料。PIE、加载基址、字长和版本变化都会影响地址。即使非 PIE 程序的槽位固定，堆指针也不能跨进程照抄。[公开逆向说明](https://github.com/soimy/c3rb3r-rescue/blob/main/skills/dsm-ransomware-rescue/references/reverse-decrypt.md)

本案盘点报告保存了一个指针值，但没有复得产生它的完整分析过程。因此公开稿不提供“通用地址”，也不声称已重现原来的定位步骤。

### 读取：可走活进程，也可走已保存映射

Linux 的 `/proc/PID/maps` 描述虚拟地址区间，`/proc/PID/mem` 按进程虚拟地址读取内存，访问受内核权限检查约束。授权管理员可以在核对进程身份及线程停止状态后，通过这两个接口保全相关范围。[Linux maps 文档](https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html)、[Linux mem 文档](https://man7.org/linux/man-pages/man5/proc_pid_mem.5.html)

另一种方式是先保存进程映射及每段的地址、长度、读取错误和摘要，再离线解析。映射文件的偏移必须按采集元数据换算；进程虚拟地址不能直接当作整机 RAM 镜像中的文件偏移。本案当前证据不足以判断最初采用的是直接导出还是先采集后离线提取。

只导出已知位置的 48 字节可以保全一个候选，不能替代完整内存取证，也不会自动找到材料位置。权限不足时应停止排查，不能为取证而重启 NAS、运行样本或扩大暴露面。

### 验证：候选通过哪些关口

本地恢复实现明确使用材料的后 32 字节作为 AES-256 密钥、前 16 字节作为 CBC IV；解开尾部前 48 字节，得到 `file-key32 + nonce8 + nonce8`。随后用每文件 key 和 nonce 按已确认的 ChaCha8 区间布局恢复载荷。尺寸、模式和保留字段也必须符合支持的格式。

两份 nonce 相等只是筛选条件，**不能认证整个候选**。尤其是 IV 错误时，CBC 第一块可能损坏，而位于后面两块中的重复 nonce 仍然相等。必须再用独立原件、可信历史摘要或足够强的完整格式检查验证实际恢复内容。由同一错误材料执行一次解密再加密，也可能往返成功。[材料层次与验证限制](https://github.com/soimy/c3rb3r-rescue/blob/main/docs/key-sharing.md)

## K.4 可公开的代码：已知地址候选导出示例

下面是本次新编写的 **A 示例**，不是未复得的原始采集脚本。它只读取已由分析人员确认的**材料缓冲区地址**，不自动找进程、不搜索任意密钥、不暂停或恢复进程、不解密文件。实际材料和参数均不包含在仓库里。

运行条件：Linux、受授权的读取权限、已核对的 PID / 启动计数 / 可执行文件摘要 / 缓冲区地址、目标所有线程已处于 `T` 或 `t`、独立救援存储上预建的当前用户私有目录。运行前先保全必要的完整证据。输出目录必须是绝对路径、无符号链接、权限严格为 `0700`，输出文件 A 必须不存在。生产执行前还需在相同 DSM 环境审核；本次只使用合成数据测试核心检查，没有在 NAS 上运行此示例。

```python
# A: reviewed-address candidate export, Python 3.8+, Linux only.
import argparse
import hashlib
import os
from pathlib import Path
import stat


def identity(root, expected_start, expected_exe):
    raw = (root / "stat").read_text()
    fields = raw[raw.rfind(")") + 2:].split()
    if fields[19] != expected_start:  # /proc stat field 22
        raise RuntimeError("process identity changed")
    h = hashlib.sha256()
    with (root / "exe").open("rb") as f:
        for block in iter(lambda: f.read(1024 * 1024), b""):
            h.update(block)
    if h.hexdigest() != expected_exe.lower():
        raise RuntimeError("executable mismatch")
    tasks = sorted((root / "task").iterdir())
    if not tasks:
        raise RuntimeError("no thread inventory")
    for task in tasks:
        rows = (task / "status").read_text().splitlines()
        states = [s.split()[1] for s in rows if s.startswith("State:")]
        if states not in (["T"], ["t"]):
            raise RuntimeError("a thread is not stopped")
    return tuple(t.name for t in tasks)


def readable(maps, address, length):
    for row in maps.splitlines():
        span, perms, *_ = row.split()
        lo, hi = (int(x, 16) for x in span.split("-"))
        if perms.startswith("r") and lo <= address and address + length <= hi:
            return True
    return False


def export_candidate(pid, expected_start, expected_exe, address, outdir):
    if pid <= 0 or address < 0:
        raise RuntimeError("invalid process or address")
    directory = Path(outdir)
    if not directory.is_absolute() or directory.resolve() != directory:
        raise RuntimeError("use an absolute, non-symlink output directory")
    dfd = os.open(str(directory), os.O_RDONLY | os.O_DIRECTORY | os.O_NOFOLLOW)
    try:
        st = os.fstat(dfd)
        if st.st_uid != os.geteuid() or stat.S_IMODE(st.st_mode) != 0o700:
            raise RuntimeError("output directory must be private and owned")
        root = Path("/proc") / str(pid)
        before = identity(root, expected_start, expected_exe)
        maps = (root / "maps").read_text()
        if not readable(maps, address, 48):
            raise RuntimeError("candidate outside a readable mapping")
        with (root / "mem").open("rb", buffering=0) as mem:
            first = os.pread(mem.fileno(), 48, address)
            second = os.pread(mem.fileno(), 48, address)
        if len(first) != 48 or first != second:
            raise RuntimeError("short or unstable candidate read")
        after = identity(root, expected_start, expected_exe)
        if before != after or maps != (root / "maps").read_text():
            raise RuntimeError("process mappings or threads changed")
        flags = os.O_WRONLY | os.O_CREAT | os.O_EXCL | os.O_NOFOLLOW
        fd = os.open("A", flags, 0o600, dir_fd=dfd)
        with os.fdopen(fd, "wb") as f:
            os.fchmod(f.fileno(), 0o600)
            if f.write(first) != 48:
                raise RuntimeError("short output write")
            f.flush()
            os.fsync(f.fileno())
        os.fsync(dfd)
    finally:
        os.close(dfd)
    print("Saved 48-byte candidate; independent validation is required.")


if __name__ == "__main__":
    p = argparse.ArgumentParser()
    p.add_argument("--pid", type=int, required=True)
    p.add_argument("--start", required=True)
    p.add_argument("--exe-sha256", required=True)
    p.add_argument("--address", type=lambda x: int(x, 0), required=True)
    p.add_argument("--out-dir", required=True)
    a = p.parse_args()
    export_candidate(a.pid, a.start, a.exe_sha256, a.address, a.out_dir)
```

该示例会把敏感候选保存到本地 A 文件，但不会把候选、地址或摘要打印到输出中，也没有网络请求。输出失败留下的部分文件应按失败证据保留，不能当作有效材料。两次读取一致和前后身份检查不能排除其他有权限的程序同时修改内存；执行时仍需控制现场。Python 对象的释放也不等于密钥字节已从内存安全擦除。

## K.5 原恢复实现的核心片段

下面按现存 Go 实现缩写了 **B 片段**。`material` 和 `record` 都是运行时输入；它只说明字节顺序，不包含真实密钥。完整调用者还必须检查尾部总长、文件声明尺寸、算法、模式及加密区间。不要单独用这个片段宣布文件已恢复。

```go
// B: abbreviated footer unwrap; caller checks metadata and file contents.
if len(material) != 48 || len(record) != 48 {
    return errors.New("invalid input length")
}
block, err := aes.NewCipher(material[16:48])
if err != nil {
    return err
}
plain := make([]byte, 48)
cipher.NewCBCDecrypter(block, material[:16]).CryptBlocks(plain, record)
if !bytes.Equal(plain[32:40], plain[40:48]) {
    return errors.New("candidate rejected")
}
// plain[:32] is the per-file key; plain[32:40] is its nonce.
// A passing comparison is only a filter, not authentication.
```

片段需要 Go 标准库 `bytes`、`crypto/aes`、`crypto/cipher` 和 `errors`。它展示的是现存恢复实现的逻辑，不是最初提取内存的代码。

## K.6 条件、失败点与保密边界

- **进程已退出或设备已重启**：不能按旧 PID 和地址恢复过去的进程内存。检查之前保存的镜像、映射集、候选和独立备份；没有这些材料时，本章不保证可恢复。
- **地址或版本不符**：拒绝读取候选。PID、全局槽位、堆指针、镜像偏移是不同概念，不能相互替代。
- **模式不符或多次加密**：保留文件，重新确认格式与材料批次；一个候选不一定适用于所有文件。
- **nonce 相等但内容不符**：候选仍未验证。尤其要考虑 IV 错误、同名文件版本变化和未覆盖真实加密区间的过小样本。
- **内存采集不完整**：记录不可读范围和短读；缺页不能补零后当作真实内容，进程映射也不包括寄存器和所有已释放物理页。
- **保密**：不上传 A 文件、完整内存、包含地址和摘要的原始采集报告、密钥截图或私有样本。提取成功后的材料只在私有环境加载，公开仓库保留方法与脱敏验证结论。

本章没有对 NAS 发起新的内存采集，也没有重新运行勒索样本。若日后复得原始采集正文或完整证据，可以补充其经脱敏的准确调用链，并继续保留本次证据缺口的说明。

## English overview

The early inventory recorded a 48-byte candidate, a process-memory pointer, a private output file, and a digest. Later recovery code used that private material as `IV16 + AES-key32` to unwrap per-file records. The relevant process remained present and was preserved in a stopped state during later recovery work.

The complete original acquisition script and the initial address-analysis record were not recovered for this publication update. We therefore do not claim that a particular syscall, memory-capture route, or first-stop sequence has been independently reconstructed. Section K.4 provides a newly written example for exporting a candidate from an already reviewed address. Section K.5 summarizes the retained recovery implementation.

A repeated footer nonce is only a candidate filter. A wrong IV can corrupt part of the per-file key while leaving that filter satisfied. Independent whole-file verification is still required. Real key bytes, process identifiers, addresses, digests, private paths, and original filenames remain masked or omitted.

[返回总目录](../README.md)
