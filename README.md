# Android-OpenSSH

This is a custom version of OpenSSH ported to Android, named `android-openssh`. It aims to provide SSH server (`sshd`) and client (`ssh`) functionalities on Android systems. The project has been modified to address some of the unique challenges of running OpenSSH on Android. This document will guide you through the features, known issues, and how to modify the code for a customized shell path.

## Features

- **SSH Client (`ssh`):** Works correctly, allowing secure connections to remote servers.
- **Customizable Shell for `sshd`:** This version allows you to define a custom shell path to accommodate Android's unique file system structure.
- **Supports OpenSSL 3.3.2:** Utilizes the latest version of OpenSSL to provide secure communications.
- **Partially Supports `sshd`:** While the server can bind to ports and handle some connection requests, there are known issues (such as "Broken Pipe" errors) during the user authentication phase.

## Modifications

### 1. Custom Shell Path in `auth.c`

The primary modification in this project is the option to specify a custom shell path for `sshd`. On many Android systems, the default shell (`/system/bin/sh`) may not work properly with `sshd`. By default, this code assumes the shell is `/system/bin/sh`, but you can modify it to any other shell executable present on your device (e.g., `/system/bin/echo`).

#### How to Modify the Shell Path

1. **Locate** the following code in `auth.c`:
    ```c
    if (options.chroot_directory == NULL || strcasecmp(options.chroot_directory, "none") == 0) {
        // Assign the path directly to the shell.
        char *shell = "/system/bin/sh"; // It needs to be revised here
        logit("Using shell: %s", shell);
        ...
    }
    ```

2. **Edit** the `char *shell` line to point to the shell executable you want `sshd` to use. For example:
    ```c
    char *shell = "/system/bin/echo"; // Change this to your desired shell path
    ```

3. **Explanation:** The `shell` variable assigns the path of the shell executable that `sshd` will use when creating a session. By default, it points to `/system/bin/sh`. However, due to the specific environment on Android, using other executables like `/system/bin/echo` might be necessary to avoid issues. Make sure the executable has proper permissions and is usable by the `sshd` process.

### 2. Known Issues and Workarounds

#### Broken Pipe and Segmentation Fault

When running `sshd` with the default or modified shell paths, you may encounter the following error:
```
mm_log_handler: write: Broken pipe
Segmentation fault
```
This issue occurs during the connection process, indicating potential problems with shell execution or environment setup. Here are some steps to troubleshoot and understand the problem:

1. **Verify Shell Path:** Ensure that the path specified in `auth.c` is correct and points to an executable file.
2. **Permissions Check:** Make sure the shell file has the necessary permissions to be executed by `sshd`.
3. **Use Debug Mode:** Run `sshd` with the `-d` flag to capture detailed debugging information, which can help identify where the problem occurs.

Despite this problem with `sshd`, the client-side tool (`ssh`) works correctly and can be used for connecting to other SSH servers.

### 3. Compilation and Installation

To compile and install this version of OpenSSH on your Android device:

1. **Configure the Source:** Use `./configure` with optional flags to adjust the build environment. For example, to specify the SSH configuration directory:
    ```sh
    ./configure --sysconfdir=/data/data/com.kalinr/openssh/etc
    ```

2. **Modify `config.h` for Custom Shell (Optional):** If you haven't already modified `auth.c`, you can define the shell path directly in `config.h` using:
    ```c
    #define SSHD_SHELL "/system/bin/echo"
    ```
    Note: This approach is less flexible and recommended only if you want a hardcoded shell path.

3. **Build the Project:**
    ```sh
    make
    ```

4. **Install the Binaries:** Copy the compiled binaries (`sshd`, `ssh`, etc.) to the desired locations on your Android device.

### 4. Issues Resolved

During the porting of OpenSSH to Android, several issues were identified and addressed:

- **Permission Issues:** Android's file system restricts certain operations, so adjustments were made to ensure OpenSSH runs with available permissions.
- **`setgroups` Issue:** Resolved the `setgroups` issue that initially prevented `sshd` from starting.
- **Shell Compatibility:** Introduced a method to modify the shell path used by `sshd` to improve compatibility with the Android environment.

## Contributing

Contributions and feedback are welcome! Please ensure that any modifications you make are documented in this `README.md` so that other users can benefit from your changes.

## License

This project is released under the [OpenSSH License](https://www.openssh.com/license.html). Please review the license before using or modifying this code.
