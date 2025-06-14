# Syncthing 自定义构建

这个仓库提供基于当前代码的 Syncthing 自定义构建，支持多平台和多架构。

## 特性

- 🔧 编译当前仓库代码（支持自定义修改）
- 🏗️ 多平台构建（Linux、Windows、macOS、FreeBSD）
- 📦 自动发布到 GitHub Releases
- ✅ 文件完整性校验
- 🏷️ 灵活的版本标签管理

## 使用方法

1. 修改 Syncthing 源码（可选）
2. 进入 Actions 页面手动触发构建
3. 或推送代码到 main/master 分支自动构建
4. 在 Releases 页面下载编译好的二进制文件

## 支持的平台

- **Linux**: amd64, arm64, arm, 386
- **Windows**: amd64, arm64, 386  
- **macOS**: amd64 (Intel), arm64 (Apple Silicon)
- **FreeBSD**: amd64, arm64

## 触发构建的方式

### 1. 手动触发
- 进入 Actions 页面
- 选择 "Build Syncthing Custom"
- 点击 "Run workflow"
- 可选择指定自定义 release 标签名

### 2. 自动触发
- **代码推送**: 推送到 main/master 分支时自动构建

## 下载和使用

1. 访问仓库的 Releases 页面
2. 下载对应您系统的二进制文件
3. Linux/macOS 用户添加执行权限：`chmod +x syncthing-*`
4. 运行程序

## 文件校验

每个 Release 都包含 `sha256sum.txt` 文件，用于验证下载文件的完整性：

```bash
# Linux/macOS
sha256sum -c sha256sum.txt

# Windows (PowerShell)
Get-FileHash syncthing-windows-amd64.exe -Algorithm SHA256
```

## 源码信息

- **基于**: 官方 Syncthing 项目
- **官方仓库**: https://github.com/syncthing/syncthing
- **许可证**: Mozilla Public License 2.0

## 注意事项

⚠️ 这是非官方的自动构建版本，仅供学习和个人使用  
⚠️ 建议从官方渠道下载正式发布的版本  
⚠️ 使用前请验证文件完整性

---

**构建时间**: 每次构建约需 15-30 分钟  
**存储周期**: Artifacts 保留 7 天，Releases 长期保存