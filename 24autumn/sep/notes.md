
1. fc.exe 是 Windows 操作系统中的一个命令行工具，全称为 "File Compare"。它用于比较两个文件的内容，输出它们之间的差异。该命令可以用于文本文件或二进制文件，常用于开发和调试过程中，帮助用户识别文件的变化或错误。

2. `.clang-format` 是一个用于配置 Clang Format 工具的文件，它可以自动格式化 C、C++、Objective-C 和其他支持的语言代码。使用 `.clang-format` 文件可以保持代码的一致性和可读性。

    ### 使用步骤：

    1. **创建 `.clang-format` 文件**：
    在你的项目根目录中创建一个名为 `.clang-format` 的文件。你可以手动创建，或者使用命令行工具生成。

    2. **配置格式化规则**：
    在 `.clang-format` 文件中，你可以设置多种格式化选项。以下是一些常见的配置项：

    ```yaml
    Language: Cpp
    BasedOnStyle: Google
    IndentWidth: 4
    UseTab: Never
    AllowShortBlocksOnASingleLine: true
    ColumnLimit: 120
    ```

    可以根据需要调整这些选项，更多配置选项可以参考 [Clang Format documentation](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)。

    3. **使用 Clang Format 格式化代码**：
    确保你已安装 Clang Format。可以通过命令行执行以下命令格式化文件：

    ```bash
    clang-format -i <file.cpp>
    ```

    选项 `-i` 表示直接修改文件，去掉 `-i` 则会将格式化结果输出到标准输出。

    4. **集成到编辑器**：
    许多现代代码编辑器和 IDE（如 Visual Studio Code、CLion 等）都支持 Clang Format，可以通过插件或内置功能自动格式化代码。

    ### 示例 `.clang-format` 文件：

    ```yaml
    BasedOnStyle: Google
    IndentWidth: 2
    ColumnLimit: 100
    AllowShortIfStatementsOnASingleLine: true
    ```