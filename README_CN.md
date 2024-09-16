# Android-OpenSSH

这是一个为 Android 系统定制移植的 OpenSSH 版本，名为 `android-openssh`。它的目标是在 Android 设备上提供 SSH 服务器（`sshd`）和客户端（`ssh`）功能。该项目已经针对在 Android 上运行 OpenSSH 所面临的独特挑战进行了修改。本文件将指导您了解项目的功能、已知问题以及如何修改代码以自定义 shell 路径。

## 功能

- **SSH 客户端 (`ssh`):** 正确工作，允许与远程服务器的安全连接。
- **`sshd` 的可定制 Shell:** 本版本允许您定义一个自定义的 shell 路径，以适应 Android 独特的文件系统结构。
- **支持 OpenSSL 3.3.2:** 利用 OpenSSL 的最新版本来提供安全通信。
- **部分支持 `sshd`:** 虽然服务器可以绑定到端口并处理一些连接请求，但在用户认证阶段存在已知问题（例如“Broken Pipe”错误）。

## 修改

### 1. `auth.c` 中的自定义 Shell 路径

本项目的主要修改是为 `sshd` 提供指定自定义 shell 路径的选项。在许多 Android 系统上，默认的 shell（`/system/bin/sh`）可能无法与 `sshd` 正常工作。默认情况下，此代码假定 shell 是 `/system/bin/sh`，但您可以将其修改为您设备上存在的任何其他 shell 可执行文件（例如，`/system/bin/echo`）。

#### 如何修改 Shell 路径

1. **定位** `auth.c` 中的以下代码：
    ```c
    if (options.chroot_directory == NULL || strcasecmp(options.chroot_directory, "none") == 0) {
        // 直接将路径分配给 shell。
        char *shell = "/system/bin/sh"; // 需要在这里进行修改
        logit("Using shell: %s", shell);
        ...
    }
    ```

2. **编辑** `char *shell` 行，使其指向您希望 `sshd` 使用的 shell 可执行文件。例如：
    ```c
    char *shell = "/system/bin/sh"; // 更改为您的期望 shell 路径
    ```

3. **解释:** `shell` 变量分配了 `sshd` 创建会话时将使用的 shell 可执行文件的路径。默认情况下，它指向 `/system/bin/sh`。然而，由于 Android 上的特定环境，使用像 `/system/bin/sh` 这样的其他可执行文件可能是必要的，但是在Termux环境上，Shell 可以是 Termux 的特定 Shell 如`/data/data/com.termux/files/usr/bin/bash`。以避免问题。确保可执行文件具有适当的权限，并且可以被 `sshd` 进程使用。

### 2. 已知问题和解决方法

#### Broken Pipe 和 Segmentation Fault

当使用默认或修改后的 shell 路径运行 `sshd` 时，您可能会遇到以下错误：
```
mm_log_handler: write: Broken pipe
Segmentation fault
```
这个问题发生在连接过程中，表明可能存在与 shell 执行或环境设置相关的问题。以下是一些排查和理解问题的步骤：

1. **验证 Shell 路径:** 确保在 `auth.c` 中指定的路径正确，并且指向一个可执行文件。
2. **权限检查:** 确保 shell 文件具有被 `sshd` 执行所需的权限。
3. **使用调试模式:** 使用 `-d` 标志运行 `sshd` 以捕获详细的调试信息，这有助于确定问题发生的位置。

尽管 `sshd` 存在这个问题，但客户端工具（`ssh`）可以正常工作，并可用于连接到其他 SSH 服务器。

### 3. 编译和安装

要在您的 Android 设备上编译和安装这个版本的 OpenSSH，请按照以下步骤操作：

1. **配置源代码:** 使用 `./configure` 和可选标志来调整构建环境。例如，要指定 SSH 编译后的目录：
    ```sh
    ./configure --prefix=/system
    ```

2. **构建项目:**
    ```sh
    make
    ```

3. **安装二进制文件:** 
    ```sh
    make install
    ```
4. **安装至指定位置(可选):**
    ```sh
    make install DESTDIR=android
    ```
    这将暂时安装至本地目录的 `android` 文件夹内。

### 4. 解决的问题

在将 OpenSSH 移植到 Android 的过程中，已经识别并解决了几个问题：

- **权限问题:** Android 的文件系统限制了某些操作，因此进行了调整以确保 OpenSSH 可以使用可用的权限运行。
- **`setgroups` 问题:** 解决了最初阻止 `sshd` 启动的 `setgroups` 问题。
- **Shell 兼容性:** 引入了一种方法来修改 `sshd` 使用的 shell 路径，以提高与 Android 环境的兼容性。

## 贡献

欢迎贡献和反馈！请确保您所做的任何修改都记录在本 `README.md` 中，以便其他用户能够从您的更改中受益。

如果您需要提出问题或建议请在issues或这里创建一个issue
https://github.com/sansjtw1/android-openssh/issues

## 许可证

本项目在 [OpenSSH(openbsd) 版权政策](https://www.openbsd.org/policy.html) 下发布。在使用或修改此代码之前，请务必阅读OpenSSH版权政策。