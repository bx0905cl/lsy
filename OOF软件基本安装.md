##### 基本安装

 这个 shell 命令 ./configure && make && make install' 旨在配置、构建和安装该软件包。以下是更详细的说明，这些说明是通用的；请参阅 'README' 文件以获取特定于此软件包的说明。有些软件包提供了 `INSTALL' 文件，但并非都实现了下面文档中描述的所有功能。在给定软件包中的可选功能的缺失并不一定是一个错误。有关 GNU 软件包的更多建议，请参阅 *note Makefile Conventions: (standards)Makefile Conventions。

‘configure' shell 脚本试图猜测在编译期间使用的各种与系统相关的变量的正确值。它使用这些值在软件包的每个目录中创建一个 'Makefile'。它还可能创建一个或多个包含系统相关定义的'.h'文件。最后，它创建一个名为 'config.status' 的 shell 脚本，您可以在将来运行以重新创建当前的配置，并创建一个包含编译器输出的文件 'config.log'（主要用于调试 `configure'）。

它还可以使用一个可选的文件（通常称为 'config.cache'，通过 '--cache-file=config.cache' 或简单地 `-C' 启用），该文件保存其测试结果以加速重新配置。默认情况下禁用缓存，以防止意外使用过时的缓存文件。

如果您需要执行不寻常的操作来编译软件包，请尝试找出 'configure' 如何检查是否需要执行这些操作，并将差异或说明发送到 'README' 中给出的地址，以便考虑在下一个发布版中使用。如果您正在使用缓存，并且在某个时候 `config.cache' 包含您不想保留的结果，您可以删除或编辑它。

文件 'configure.ac'（或 'configure.in'）用于由名为 'autoconf' 的程序创建 'configure'。如果您想更改它或使用更新版本的 'autoconf' 重新生成 'configure'，则需要 `configure.ac'。

编译此软件包的最简单方法是：

1. 进入包含软件包源代码的目录，然后键入 `./configure' 以为您的系统配置软件包。

运行 `configure' 可能需要一些时间。在运行时，它会打印一些消息，说明它正在检查哪些功能。

1. 键入 `make' 以编译软件包。
2. 可选地，键入 `make check' 以运行任何随软件包一起提供的自测试，通常使用刚刚构建的未安装的二进制文件。
3. 键入 'make install' 以安装程序以及任何数据文件和文档。当安装到由 root 拥有的前缀时，建议将软件包配置和构建为常规用户，并仅以 root 特权执行 `make install' 阶段。
4. 可选地，键入 'make installcheck' 以重复任何自测试，但这次使用其最终安装位置中的二进制文件。此目标不安装任何内容。以常规用户身份运行此目标，特别是如果先前的 `make install' 需要 root 特权，则验证安装是否完成正确。
5. 您可以通过键入 'make clean' 从源代码目录中删除程序二进制文件和对象文件。要同时删除 'configure' 创建的文件（以便可以为不同类型的计算机编译软件包），请键入 'make distclean'。还有一个 `make maintainer-clean' 目标，但那主要是为软件包的开发人员准备的。如果您使用它，您可能需要获取所有其他程序以重新生成分发的文件。
6. 通常，您还可以键入 `make uninstall' 以再次删除已安装的文件。实际上，并非所有软件包都测试了卸载是否正确工作，尽管这是 GNU 编码标准要求的。
7. 一些软件包，特别是那些使用 Automake 的软件包，提供 'make distcheck'，开发人员可以使用它来测试所有其他目标，如'make install' 和 `make uninstall' 是否正常工作。通常不由最终用户运行此目标。

##### 编译器和选项

一些系统在编译或链接时需要一些 'configure' 脚本不知道的特殊选项。运行 `./configure --help' 以获取有关一些相关环境变量的详细信息。

您可以通过在命令行或环境中设置变量，为配置参数提供 `configure' 的初始值。以下是一个示例：./configure CC=c99 CFLAGS=-g LIBS=-lposix

有关更多详细信息，请参阅 *Note Defining Variables::。

#####  为多种架构进行编译

您可以同时为多种计算机类型编译该软件包，方法是将每种架构的目标文件放置在各自的目录中。为此，您可以使用 GNU ’make'。‘cd' 到您希望放置目标文件和可执行文件的目录，并运行 ’configure' 脚本。‘configure' 自动检查 ‘configure' 所在的目录和 `..' 中的源代码。这被称为 "VPATH" 构建。

对于非 GNU ’make'，最好在源代码目录中一次为一种架构编译软件包。在为一种架构安装完软件包后，在为另一种架构重新配置之前使用 `make distclean'。

在 MacOS X 10.5 及更高版本系统上，您可以通过向编译器指定多个 ‘-arch' 选项，但仅向预处理器指定单个 `-arch' 选项，创建适用于多个系统类型的库和可执行文件，这被称为 "fat" 或 "universal" 二进制文件。像这样：

./configure CC="gcc -arch i386 -arch x86_64 -arch ppc -arch ppc64" \           

​                    CXX="g++ -arch i386 -arch x86_64 -arch ppc -arch ppc64" \           

​                    CPP="gcc -E" CXXCPP="g++ -E"

不能保证在所有情况下都能产生可工作的输出，如果遇到问题，您可能需要逐个构建每个架构，并使用 `lipo' 工具组合结果。

##### 安装路径

默认情况下，’make install' 将软件包的命令安装在 ‘/usr/local/bin' 下，包含文件安装在 ’/usr/local/include' 下，依此类推。您可以通过在 ‘configure' 中提供选项 ’--prefix=PREFIX' 来指定除了 `/usr/local' 之外的安装前缀，其中 PREFIX 必须是绝对文件名。

您可以为与体系结构相关的文件和与体系结构无关的文件指定单独的安装前缀。如果您向 ‘configure' 传递选项 `--exec-prefix=PREFIX'，则软件包将使用 PREFIX 作为安装程序和库的前缀。文档和其他数据文件仍然使用常规前缀。

此外，如果您使用不同的目录布局，您可以使用 ’--bindir=DIR' 之类的选项为特定类型的文件指定不同的值。运行 ‘configure --help' 查看您可以设置的目录列表以及它们包含哪些类型的文件。通常，这些选项的默认值是用 ’${prefix}' 表示的，因此仅指定 `--prefix' 将影响未明确提供的所有其他目录规范。

影响安装位置的最便携方法是将正确的位置传递给 ‘configure'；然而，许多软件包提供以下两种方法之一或两种方法，以在不必重新配置或重新编译的情况下通过变量赋值传递给 `make install' 命令行来更改安装位置。

第一种方法涉及为每个受影响的目录提供一个覆盖变量。例如，’make install prefix=/alternate/directory' 将选择所有以 ‘${prefix}' 表示的目录配置变量的备用位置。在 ‘configure' 期间指定的任何未在 `${prefix}' 表示的目录都必须在整个安装的安装时进行覆盖。为每个目录变量提供 makefile 变量覆盖的方法是 GNU 编码标准要求的，并且理想情况下不会导致重新编译。但是，一些平台对共享库的语义有已知限制，使用此方法时可能需要重新编译，特别是在使用 GNU Libtool 的软件包中可能更为明显。

第二种方法涉及提供 ’DESTDIR' 变量。例如，‘make install DESTDIR=/alternate/directory' 将在所有安装名称之前添加 ’/alternate/directory'。‘DESTDIR' 覆盖的方法不是 GNU 编码标准要求的，并且在具有驱动器号的平台上无法工作。另一方面，它在避免重新编译问题方面效果更好，即使在 ’configure' 时某些目录选项未按 `${prefix}' 表示。

##### 可选功能

如果软件包支持，您可以通过在 ‘configure' 中使用选项 ’--program-prefix=PREFIX' 或 `--program-suffix=SUFFIX' 来使程序在其名称上具有额外的前缀或后缀。

一些软件包关注 ‘configure' 的 ’--enable-FEATURE' 选项，其中 FEATURE 表示软件包的可选部分。它们还可能关注 ‘--with-PACKAGE' 选项，其中 PACKAGE 是类似于 ’gnu-as' 或 ‘x'（用于 X Window 系统）的东西。’README' 应该提到软件包识别的任何 ‘--enable-' 和 `--with-' 选项。

对于使用 X Window 系统的软件包，’configure' 通常可以自动找到 X 包含和库文件，但如果不能，您可以使用 ‘configure' 选项 ’--x-includes=DIR' 和 `--x-libraries=DIR' 来指定它们的位置。

一些软件包提供了配置 ‘make' 执行的详细程度的能力。对于这些软件包，运行 ’./configure --enable-silent-rules' 将默认设置为最小输出，可以使用 ‘make V=1' 覆盖；而运行 ’./configure --disable-silent-rules' 将默认设置为详细输出，可以使用 `make V=0' 覆盖。

##### 特定系统

在 HP-UX 上，默认的 C 编译器不兼容 ANSI C。如果没有安装 GNU CC，建议使用以下选项来使用 ANSI C 编译器：./configure CC="cc -Ae -D_XOPEN_SOURCE=500"

如果这不起作用，请安装 HP-UX 的预构建的 GCC 二进制文件。

HP-UX 上的 ‘make' 会更新与其先决条件具有相同时间戳的目标，这使得它在涉及生成的文件（如 ’configure'）时通常无法使用。请改用 GNU `make'。

在 OSF/1（也称为 Tru64）上，某些版本的默认 C 编译器无法解析其 ‘<wchar.h>' 头文件。选项 `-nodtk' 可以用作一种解决方法。因此，如果没有安装 GNU CC，建议尝试： ./configure CC="cc"

如果这不起作用，请尝试：./configure CC="cc -nodtk"

在 Solaris 上，请不要将 ’/usr/ucb' 提前放在 ‘PATH' 中。此目录包含多个无效的程序；这些程序的工作变体可在 ’/usr/bin' 中找到。因此，如果需要在 ‘PATH' 中包含 ’/usr/ucb'，请将其放在 `/usr/bin' 之后。

在 Haiku 上，为所有用户安装的软件放在 ‘/boot/common' 中，而不是 `/usr/local'。建议使用以下选项：./configure --prefix=/boot/common

##### 共享默认值

如果您希望为 ’configure' 脚本设置默认值以供共享，可以创建一个名为 ‘config.site' 的站点 shell 脚本，为诸如 ’CC'、‘cache_file' 和 ’prefix' 等变量提供默认值。‘configure' 会查找 ’PREFIX/share/config.site'（如果存在），然后查找 ‘PREFIX/etc/config.site'（如果存在）。或者，您可以将 ’CONFIG_SITE' 环境变量设置为站点脚本的位置。请注意：并非所有的 `configure' 脚本都会寻找站点脚本。

##### 定义变量

在站点 shell 脚本中未定义的变量可以在传递给 ‘configure' 的环境中进行设置。然而，一些软件包可能在构建过程中再次运行 configure，并且这些变量的定制值可能会丢失。为了避免这个问题，您应该在 ’configure' 命令行中设置它们，使用 `VAR=value'。例如：./configure CC=/usr/local2/bin/gcc

将使用指定的 `gcc' 作为 C 编译器（除非在站点 shell 脚本中被覆盖）。

不幸的是，由于 Autoconf 的限制，这种技术对于 `CONFIG_SHELL' 不起作用。在限制解除之前，您可以使用这个解决方法：CONFIG_SHELL=/bin/bash ./configure CONFIG_SHELL=/bin/bash

##### configure' 调用

`configure' 识别以下选项以控制其操作方式。

‘--help' ’-h' 打印 `configure' 的所有选项摘要，并退出。

‘--help=short' ’--help=recursive' 打印此软件包独有的 ‘configure' 选项摘要，并退出。’short' 变体列出仅在顶层使用的选项，而 `recursive' 变体列出在任何嵌套软件包中也存在的选项。

‘--version' ’-V' 打印用于生成 `configure' 脚本的 Autoconf 版本，并退出。

‘--cache-file=FILE' 启用缓存：使用 FILE（传统上为 ’config.cache'）中的测试结果并保存。FILE 默认为 `/dev/null' 以禁用缓存。

‘--config-cache' ’-C' `--cache-file=config.cache' 的别名。

’--quiet' ‘--silent' ‘-q' 不要打印说明正在进行哪些检查的消息。要抑制所有正常输出，请将其重定向到 `/dev/null'（任何错误消息仍将显示）。

’--srcdir=DIR' 在目录 DIR 中查找软件包的源代码。通常，`configure' 可以自动确定该目录。

’--prefix=DIR' 将 DIR 用作安装前缀。有关详细信息，请参阅 *note Installation Names::，其中包括用于微调安装位置的其他选项。‘--no-create' `-n' 运行 configure 检查，但在创建任何输出文件之前停止。

‘configure' 还接受一些其他不太常用的选项。运行 `configure --help' 以获取更多详细信息。