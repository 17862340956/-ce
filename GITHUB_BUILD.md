# GitHub Actions 编译

作者：杏仁 / xingren。

1. 在 GitHub 创建仓库，把本包解压后的全部内容放到仓库根目录，包括隐藏目录 `.github` 和文件 `.gitattributes`。
2. 确认仓库中存在 `.github/workflows/build.yml`，以及根目录的 `build.sh`、`Makefile`、`cpuinfo_spoof.c`。
3. 打开仓库 **Actions → PKG110 SM8650 6.1 KO r1 → Run workflow**，选择分支后运行。工作流使用手动触发；请将工作流提交到默认分支。
4. 运行结束后，在页面底部 **Artifacts** 下载 `pkg110-ko-r1-运行编号`；若构建失败，下载 `pkg110-diagnostics-r1-运行编号` 查看 `build_logs/build.log`。

不要把整个 ZIP 当成单个文件上传，否则 GitHub 不会识别其中的工作流。Web 上传时隐藏目录容易遗漏，推荐使用 Git 客户端提交解压目录内容。

本包包含工作流和构建入口，未在你的 GitHub 仓库实际运行。现有 report.py 依赖目标 CRC/KCFI 等证据字段；目标画像目前尚未补齐这些字段，因此报告步骤可能失败。现有布局头中仍含待目标 BTF 核实的基线偏移。构建产物需经过进一步目标适配验证。
