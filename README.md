# OPD2407 / MT6897 / 6.1.128 适配包 r1

作者：xingren。设备画像来自用户提供的 kernel_profile_20261031_191000_10354.tar.gz。本包是独立于 PHP110 的目标源码及外部模块构建包，尚未包含这台设备实际编译或加载通过的 KO。

## 目标与依据

- 设备：OnePlus OPD2407，device 属性 OP615AL1，平台 MT6897。
- 当前用户空间：Android 15；内核分支：android14-6.1。
- 运行 release：6.1.128-android14-11-o-g024c24b34f5e。
- 编译器基线：clang-r487747c，画像版本输出为 clang 17.0.2。
- 配置 SHA256：78030136b5d95f3cec4444bf72f4141083c5eca315cb9eed7fb407968b89faee。
- 原始 BTF SHA256：9372cccd4e885660697069315eed8d1a25cb327a3959054f2c51febf8d8def67。
- BTF 选择性解码保存在 TARGET_BTF.json，厂商样本的216项符号 CRC 和25项 KCFI ID 证据保存在 TARGET_PROFILE.json。它们是部分目标证据，构建后按实际交集重新计算覆盖。
- 采集包41项清单中40项匹配，仅 SUMMARY.txt 的摘要不符；配置、BTF和两份模块样本均验证通过。

## 文件放置与 Actions

全部文件直接放在仓库根目录，唯一需要的子目录是 GitHub 规定的 .github/workflows。编译自检不依赖工作流的具体文件名、README 或 .gitattributes。

在上传本包的新提交上运行 **OPD2407 MT6897 6.1 KO r1**。历史运行的 Re-run jobs 仍使用历史提交。日志应先出现：

```text
BUILD_KIT=opd2407-ddk-r1
KIT_VERIFIED=opd2407-ddk-r1 files=16
```

使用固定 wrapper 拉取预制 android14-6.1 DDK，再选定同一个镜像 ID 完成编译器检查、清理与构建。只进行构建工具/生成文件准备及一个外部模块构建。modpost 的符号解析补丁会在临时容器内恢复，使用已核对的 prepare0 目标重建，保留源码、工具时间及哈希检查。

成功产物在 dist；失败时下载 opd2407-diagnostics-r1-*，其中保留完整日志、原始KO、生成的.mod.c、构建配置、Module.symvers 和失败报告。没有通过报告校验的文件不会作为 verified 产物发布。

## Linux 本地构建

需要 Bash、Docker daemon、Python3、sha256sum。建议在独立工作副本中执行：

```sh
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install -r requirements.txt
python3 verify_kit.py
bash build.sh
```

成功后保留现有 dist；重复构建请先保存产物并使用新的工作副本。

## 设备端

将生成的 dist 内容放在同一目录，以 ROOT 执行：

```sh
sh run.sh
```

默认是 inspect：加载只读检查实例，核对型号、平台、release、完整运行配置和BTF指纹，并检查UTS、根CPU属性、拓扑、频率和CPUINFO原对象。不发布替换指针，生成日志包。输出 INSPECT_OK 后，启用原项目的 Kirin9020 展示预设使用：

```sh
sh run.sh full
```

full 会卸载本版本未修改、未固定驻留的 inspect 实例，再执行7项指针替换和1项UTS展示修改，共8项。已经发布修改的实例会固定驻留，切换到其它模式使用重启后的干净状态。full 需要 resetprop，加载前检查该依赖。

额外支持 uts、cpus、topology、cpufreq、cpuinfo、combo 单独模式。运行后返回的日志包包含 insmod 输出、状态、dmesg 和相应读取结果。

## 本次实际适配

1. 原始内存访问偏移来自这台设备的 BTF：UTS name=0、device.id=764；inode/dentry/superblock链和proc私有字段也按目标偏移读取，不借用PHP110结构地址。
2. 对传入内核的kprobe/path及克隆的proc_ops、设备/频率属性、seq_operations和new_utsname，使用目标精确大小与偏移的编译期断言。
3. 移除本设备未暴露的die_id计划；保留core/package两项，full指针计数为7。CPUINFO去掉5.10专用arch_freq_prepare_all依赖，公开MT6897 6.1源码的open路径直接seq_open。
4. 保留CFI/KCFI及MODVERSIONS。报告解析全部10个已发布回调的DWARF参数与返回值、KCFI ID，并单独标明哪些ID有原生同型样本依据。
5. 实际KO每个CRC条目对照当次DDK Module.symvers；与厂商样本存在交集的CRC必须全部一致，module_layout必须匹配；样本未覆盖的符号逐项列出，不补造、不替换CRC。
6. 17个目标偏移/大小常量随KO保存并由报告再对照BTF；对ro_after_init范围边界缺失或异常直接退出预检，避免静默取消只读范围排除。

预设展示为12行CPUINFO、0-11文本掩码、统一频率列表以及Kirin9020属性；真实设备仍为8核，CPU8-11目录、实际调度/亲和性、MIDR、CPU指令和硬件频率保持原设备行为。目标显示release为6.1.128-HUAWEI-Kirin9020。这是原项目的展示预设，不是新增真实处理器或完整硬件模拟。

## 验证范围

本机没有可用Docker/Linux内核编译环境。交付前完成源码/接口复查、BTF及厂商样本校验、脚本与C语法解析、报告函数及模拟Android运行流程回归、根目录清单与归档验证；实际DDK编译与设备加载尚待Actions产物和设备日志确认。

即使报告 validation=passed，仍需看 target_crc_unverified_symbols 和 target_kcfi_unverified_callbacks。样本没有提供完整目标 Module.symvers，BTF也不证明运行时符号地址和页面权限；因此不把部分CRC/KCFI匹配写成完整OEM ABI证明。

公开基线参考（并未证明与安装固件逐字节一致）：

- https://github.com/OnePlusOSS/android_kernel_oneplus_mt6897/tree/11c181cc0a2a927ecc79036b6e2bf7fd4430feeb
- https://github.com/OnePlusOSS/android_kernel_oneplus_mt6897/blob/11c181cc0a2a927ecc79036b6e2bf7fd4430feeb/fs/proc/cpuinfo.c
- https://github.com/Ylarod/ddk/tree/1149888f44873f275019d03ee0dbb7dddf5a3055
