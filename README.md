# PACMAN(8) - Pacman 手册（中文翻译）
**版本：Pacman 7.0.0（2024-09-09）**  

## 名称
pacman - 包管理器工具

## 概述
pacman &lt;操作&gt; [选项] [目标]

## 描述
Pacman 是一个包管理工具，用于跟踪 Linux 系统上安装的包。它支持依赖关系、包组、安装与卸载脚本，并能够同步本地机器与远程仓库以自动升级包。Pacman 的包格式是压缩的 tar 格式。

自 3.0.0 版本以来，pacman 已成为 [libalpm(3)](https://man.archlinux.org/man/libalpm.3.en) 的前端，该库是 "Arch Linux 包管理" 库。该库允许编写其他前端（例如 GUI 前端）。

调用 pacman 需要指定一个操作，以及可能的选项和目标。目标通常是包名、文件名、URL 或搜索字符串。目标可以作为命令行参数提供。此外，如果 stdin 不是从终端读取并且传递了单个连字符（-）作为参数，则将从 stdin 读取目标。

## 操作

### -D, --database
对包数据库进行操作。此操作允许你修改 pacman 数据库中已安装包的某些属性。它还允许检查数据库的内部一致性。详见数据库选项部分。

### -Q, --query
查询包数据库。此操作允许你查看已安装的包及其文件，以及关于各个包的元信息（依赖关系、冲突、安装日期、构建日期、大小）。可以针对本地包数据库运行，也可以用于单个包文件。如果命令行中未提供包名，则将查询所有已安装的包。此外，可以对包列表应用各种筛选器。详见查询选项部分。

### -R, --remove
从系统中删除包。也可以指定组进行删除，在这种情况下，组中的每个包都将被删除。指定包的文件将被删除，数据库将被更新。大多数配置文件将保存为 .pacsave 扩展名，除非使用 --nosave 选项。详见删除选项部分。

### -S, --sync
同步包。包直接从远程仓库安装，包括运行这些包所需的所有依赖。例如，pacman -S qt 将下载并安装 qt 及其所有依赖包。如果一个包名在多个仓库中存在，可以明确指定要安装的仓库：pacman -S testing/qt。你还可以指定版本要求：pacman -S "bash&gt;=3.2"。需要使用引号，否则 shell 会将 &gt; 解释为文件重定向。

除了包之外，也可以指定组。例如，如果 gnome 是一个定义的包组，那么 pacman -S gnome 将提供一个提示，允许你从编号列表中选择要安装的包。包的选择是使用以空格和（或）逗号分隔的包编号列表指定的。可以通过指定第一个和最后一个包编号并用连字符（-）分隔来选择连续的包。通过在编号或编号范围前加上 ^ 来排除包。

Pacman 还处理提供其他包的包。例如，pacman -S foo 将首先查找 foo 包。如果未找到 foo，将搜索提供与 foo 相同功能的包。如果找到了任何包，将安装它。如果找到多个提供 foo 的包，将提供选择提示。

你也可以使用 pacman -Su 来升级所有过期的包。详见同步选项部分。当升级时，pacman 会进行版本比较以确定哪些包需要升级。该行为如下：

- 字母数字：
  1.0a &lt; 1.0b &lt; 1.0beta &lt; 1.0p &lt; 1.0pre &lt; 1.0rc &lt; 1.0 &lt; 1.0.a &lt; 1.0.1
- 数字：
  1 &lt; 1.0 &lt; 1.1 &lt; 1.1.1 &lt; 1.2 &lt; 2.0 &lt; 3.0.0

此外，版本字符串可以定义一个 epoch 值，除非 epoch 值相等，否则该值将覆盖任何版本比较。其格式为 epoch:version-rel。例如，2:1.0-1 始终大于 1:3.6-1。

### -T, --deptest
检查依赖关系；这在如 makepkg 之类的脚本中很有用，用于检查已安装的包。此操作将检查每个指定的依赖并返回系统上当前未满足的依赖列表。此操作不接受其他选项。示例用法：pacman -T qt "bash&gt;=3.2"。

### -U, --upgrade
升级或添加包并从同步仓库安装所需的依赖。可以指定 URL 或文件路径。这是一个 "移除然后添加" 的过程。详见升级选项部分；另请参阅处理配置文件以了解 pacman 如何处理配置文件。

### -F, --files
查询文件数据库。此操作允许你查找拥有某些文件的包或显示某些包拥有的文件。仅搜索同步数据库中的包。详见文件选项部分。

### -V, --version
显示版本并退出。

### -h, --help
显示给定操作的语法。如果未提供操作，则显示一般语法。

## 选项

### -b, --dbpath &lt;path&gt;
指定一个替代的数据库位置（默认是 /var/lib/pacman）。除非你清楚自己在做什么，否则不要使用此选项。  
**注意**：如果指定，这将是一个绝对路径，根路径不会自动添加到前面。

### -r, --root &lt;path&gt;
指定一个替代的安装根目录（默认是 /）。不应该使用此选项将软件安装到 /usr/local 而不是 /usr。  
**注意**：如果数据库路径或日志文件未在命令行或 [pacman.conf(5)](https://man.archlinux.org/man/pacman.conf.5.en) 中指定，则它们的默认位置将在此根路径中。  
**注意**：此选项不适合在挂载的访客系统上执行操作。请参阅 --sysroot。

### -v, --verbose
输出诸如 Root、配置文件、数据库路径、缓存目录等信息。

### --arch &lt;arch&gt;
指定一个替代的体系架构。

### --cachedir &lt;dir&gt;
指定一个替代的包缓存位置（默认是 /var/cache/pacman/pkg）。可以指定多个缓存目录，pacman 将按照传递的顺序依次尝试。  
**注意**：这是一个绝对路径，根路径不会自动添加到前面。

### --color &lt;when&gt;
指定何时启用颜色显示。有效选项为 always、never 或 auto。  
- always 强制开启颜色；
- never 强制关闭颜色；
- auto 只在输出到 tty 时自动启用颜色。

### --config &lt;file&gt;
指定一个替代的配置文件。

### --debug
显示调试消息。在报告错误时，建议使用此选项。

### --gpgdir &lt;dir&gt;
指定用于 GnuPG 验证包签名的文件目录（默认是 /etc/pacman.d/gnupg）。该目录应包含两个文件：pubring.gpg 和 trustdb.gpg。  
- pubring.gpg 保存所有打包者的公钥。
- trustdb.gpg 包含所谓的信任数据库，指定密钥是可信的和被认证的。  
**注意**：这是一个绝对路径，根路径不会自动添加到前面。

### --hookdir &lt;dir&gt;
指定一个替代的包含 hook 文件的目录（默认是 /etc/pacman.d/hooks）。可以指定多个 hook 目录，后面的目录中的 hooks 将优先于前面的目录中的 hooks。  
**注意**：这是一个绝对路径，根路径不会自动添加到前面。

### --logfile &lt;file&gt;
指定一个替代的日志文件。无论安装根目录设置为何，此路径都是绝对路径。

### --noconfirm
跳过所有 "你确定吗？" 的提示。如果你想从脚本中运行 pacman，除非你明确知道自己在做什么，否则不建议使用此选项。

### --confirm
取消之前 --noconfirm 的效果。

### --disable-download-timeout
禁用默认的下载低速限制和超时设置。如果你在使用代理和（或）安全网关时下载文件有问题，请使用此选项。

### --sysroot &lt;dir&gt;
指定一个替代的系统根目录。该路径将添加到所有其他配置目录和以 file:// 开头的任何仓库服务器前。传递为目标的任何路径或 URL 都不会被修改。此选项允许正确操作挂载的访客系统。

### --disable-sandbox
禁用在 Linux 系统上应用的默认沙盒机制。如果在运行不支持该功能的 Linux 内核时，因使用 landlock 下载文件而出现故障，此选项会非常有用。

## 事务选项（适用于 -S, -R 和 -U）

### -d, --nodeps
跳过依赖版本检查，但仍检查包名。通常情况下，pacman 会检查包的依赖字段，以确保所有依赖项已安装并且系统中没有包冲突。指定该选项两次可跳过所有依赖检查。

### --assume-installed &lt;package=version&gt;
添加一个虚拟包 "package"（版本 "version"）到事务中以满足依赖关系。这样可以禁用特定的依赖检查而不影响所有依赖检查。要禁用所有依赖检查，请参阅 --nodeps 选项。

### --dbonly
仅添加/删除数据库条目，文件不受影响。

### --noprogressbar
在下载文件时不显示进度条。此选项在脚本调用 pacman 并捕获输出时非常有用。

### --noscriptlet
如果存在安装脚本，不执行它。除非你清楚自己在做什么，否则不要使用此选项。

### -p, --print
仅打印目标，而不执行实际操作（同步、删除或升级）。使用 --print-format 指定目标显示的格式。默认的格式字符串是 "%l"，它显示 -S 的 URL，-U 的文件名，以及 -R 的 pkgname-pkgver。

### --print-format &lt;format&gt;
指定一个类似 printf 的格式来控制 --print 操作的输出。可用的属性有：
- %a：架构
- %b：构建日期
- %d：描述
- %e：包基础
- %f：文件名
- %g：base64 编码的 PGP 签名
- %h：sha256sum
- %m：md5sum
- %n：包名
- %p：打包者
- %v：包版本
- %l：位置
- %r：仓库
- %s：大小
- %C：checkdepends
- %D：依赖
- %G：组
- %H：冲突
- %L：许可证
- %M：makedepends
- %O：可选依赖
- %P：提供
- %R：替换

此选项隐含了 --print。

## 升级选项（适用于 -S 和 -U）

### -w, --downloadonly
从服务器检索所有包，但不安装/升级任何内容。

### --asdeps
非显式地安装包，换句话说，将其安装原因伪装为依赖。这对 makepkg 和其他从源码构建工具很有用，它们需要在构建包之前安装依赖。

### --asexplicit
显式地安装包，换句话说，将其安装原因伪装为显式安装。如果你希望将依赖标记为显式安装，以避免在执行 --recursive 移除操作时被删除，此选项非常有用。

### --ignore &lt;package&gt;
指示 pacman 忽略包的升级，即使有可用的升级。可以通过逗号分隔指定多个包。

### --ignoregroup &lt;group&gt;
指示 pacman 忽略组中所有包的升级，即使有可用的升级。可以通过逗号分隔指定多个组。

### --needed
不重新安装已经是最新版本的目标包。

### --overwrite &lt;glob&gt;
绕过文件冲突检查并覆盖冲突文件。如果要安装的包包含已安装的文件并与 glob 匹配，此选项将导致所有这些文件被覆盖。使用 --overwrite 不会允许使用文件覆盖目录，或安装包含冲突文件和目录的包。可以通过逗号分隔指定多个模式。此选项可以指定多次。模式可以被否定，即匹配的文件将不会被覆盖，通过在它们前面加上感叹号。后续的匹配将覆盖先前的匹配。以感叹号或反斜杠开头的文字需要转义。

## 查询选项（适用于 -Q）

### -c, --changelog
查看包的变更日志（如果存在）。

### -d, --deps
将输出限制或过滤为作为依赖项安装的包。此选项可与 -t 结合使用，列出真正的孤立包（即作为依赖项安装但不再被任何已安装包需要的包）。

### -e, --explicit
将输出限制或过滤为显式安装的包。此选项可与 -t 结合使用，列出未被其他包依赖的显式安装包。

### -g, --groups
显示属于指定组的所有包。如果未指定组名，则列出所有分组包。

### -i, --info
显示指定包的信息。如果查询的是包文件而不是本地数据库，可以使用 -p 选项。传递两个 --info 或 -i 标志还将显示备份文件及其修改状态的列表。

### -k, --check
检查指定包拥有的所有文件是否存在于系统中。如果未指定包或未提供过滤标志，则检查所有已安装的包。指定此选项两次将执行更详细的文件检查（包括权限、文件大小和修改时间），对于包含所需 mtree 文件的包尤为适用。

### -l, --list
列出指定包拥有的所有文件。命令行上可以指定多个包。

### -m, --foreign
将输出限制或过滤为未在同步数据库中找到的包。通常这些是手动下载并使用 --upgrade 安装的包。

### -n, --native
将输出限制或过滤为在同步数据库中找到的包。这是 --foreign 的反向过滤。

### -o, --owns &lt;file&gt;
搜索拥有指定文件的包。路径可以是相对路径或绝对路径，可以指定一个或多个文件。

### -p, --file
表示命令行中提供的包是一个文件，而不是数据库中的条目。文件将被解压缩并查询。此选项在与 --info 和 --list 结合使用时非常有用。

### -q, --quiet
对某些查询操作显示较少的信息。当 pacman 的输出在脚本中处理时，这非常有用。search 只会显示包名，而不显示版本、组和描述信息；owns 只会显示包名，而不是显示 "文件由包拥有" 的消息；group 只显示包名，忽略组名；list 只显示文件，忽略包名；check 只显示包名和丢失的文件对；一个空查询只显示包名，而不是包名和版本。

### -s, --search &lt;regexp&gt;
在每个本地安装的包中搜索名称或描述中与 regexp 匹配的内容。如果包含多个搜索词，则仅返回描述中匹配所有搜索词的包。

### -t, --unrequired
将输出限制或过滤为仅打印当前未被任何已安装包必需或可选依赖的包。指定此选项两次以包含那些作为可选依赖但不被其他包直接需要的包。

### -u, --upgrades
将输出限制或过滤为本地系统上已过时的包。仅使用包版本来查找过时的包；此处不检查替代包。此选项在同步数据库使用 -Sy 刷新后效果最佳。

## 删除选项（适用于 -R）

### -c, --cascade
删除所有目标包，以及依赖其中一个或多个目标包的所有包。此操作是递归的，必须谨慎使用，因为它可能会删除许多可能需要的包。

### -n, --nosave
指示 pacman 忽略文件的备份设计。在文件从系统中删除时，数据库通常会检查是否应将文件重命名为 .pacsave 扩展名。

### -s, --recursive
删除指定的每个目标及其所有依赖项，前提是 (A) 这些依赖项未被其他包依赖；(B) 它们不是由用户显式安装的。此操作是递归的，类似于反向的 --sync 操作，有助于保持系统干净且无孤立包。如果你想忽略条件 (B)，可以将此选项传递两次。

### -u, --unneeded
删除不被任何其他包依赖的目标。这在删除某个组时（不使用 -c 选项）非常有用，以避免破坏任何依赖关系。

## 同步选项（适用于 -S）

### -c, --clean
从缓存中删除不再安装的包，以及当前未使用的同步数据库，以释放磁盘空间。当 pacman 下载包时，它会将包保存在缓存目录中。此外，同步数据库也会被保存，且即使它们已从 [pacman.conf(5)](https://man.archlinux.org/man/pacman.conf.5.en) 配置文件中删除，仍然不会被删除。使用一个 --clean 标志仅删除不再安装的包；使用两个标志则会删除缓存中的所有文件。无论哪种情况，你都可以选择是否删除包和（或）未使用的已下载数据库。

如果你使用共享的网络缓存，请参见 [pacman.conf(5)](https://man.archlinux.org/man/pacman.conf.5.en) 中的 CleanMethod 选项。

### -g, --groups
显示每个指定包组的所有成员。如果没有提供组名，则列出所有组；将标志传递两次以查看所有组及其成员。

### -i, --info
显示指定同步数据库包的信息。传递两个 --info 或 -i 标志还会显示所有依赖于此包的仓库中的包。

### -l, --list
列出指定仓库中的所有包。命令行上可以指定多个仓库。

### -q, --quiet
对某些同步操作显示较少的信息。当 pacman 的输出在脚本中处理时，这非常有用。search 只会显示包名，而不会显示仓库、版本、组和描述信息；list 只显示包名，忽略数据库和版本；group 只显示包名，忽略组名。

### -s, --search &lt;regexp&gt;
此操作将在同步数据库中搜索每个包的名称或描述，匹配正则表达式 regexp。如果包含多个搜索词，只有描述中匹配所有搜索词的包才会被返回。

### -u, --sysupgrade
升级所有过时的包。每个当前安装的包都会被检查，如果存在较新的包则进行升级。系统会提供所有要升级的包的报告，操作不会在没有用户确认的情况下继续。依赖项会在此级别自动解决，并在必要时安装/升级。

将此选项传递两次以启用包降级；在这种情况下，pacman 将选择同步数据库中与本地版本不匹配的包版本。当用户从测试仓库切换到稳定仓库时，这可能会很有用。

你还可以手动指定其他目标，以便 -Su foo 将执行系统升级，并在同一操作中安装/升级 foo 包。

### -y, --refresh
从 [pacman.conf(5)](https://man.archlinux.org/man/pacman.conf.5.en) 中定义的服务器下载主包数据库（repo.db）的新副本。通常在每次使用 --sysupgrade 或 -u 时应使用此选项。传递两个 --refresh 或 -y 标志将强制刷新所有包数据库，即使它们看起来是最新的。

## 数据库选项（适用于 -D）

### --asdeps &lt;package&gt;
将指定的包标记为非显式安装的包；换句话说，将它们的安装理由设置为依赖项安装。

### --asexplicit &lt;package&gt;
将指定的包标记为显式安装的包；换句话说，将它们的安装理由设置为显式安装。这在你希望保留一个最初作为另一个包的依赖项安装的包时非常有用。

### -k, --check
检查本地包数据库的内部一致性。这将检查所有必需的文件是否存在，已安装的包是否具有所需的依赖项，是否存在冲突，并且不会有多个包拥有同一个文件。指定此选项两次将对同步数据库进行检查，以确保所有指定的依赖项可用。

### -q, --quiet
在数据库操作成功完成后抑制消息输出。

## 文件选项（适用于 -F）

### -y, --refresh
从服务器下载新的包文件数据库（repo.files）。使用两次以强制刷新，即使数据库是最新的。

### -l, --list
列出查询包所拥有的文件。

### -x, --regex
将每个查询视为正则表达式。

### -q, --quiet
对某些文件操作显示较少的信息。当 pacman 的输出被脚本处理时，这非常有用，然而，你可能更愿意使用 --machinereadable。

### --machinereadable
以机器可读的输出格式打印每个匹配项。格式为 repository\0pkgname\0pkgver\0path\n，其中 \0 表示空字符，\n 表示换行符。

## 处理配置文件

Pacman 使用与 RPM 相同的逻辑来确定对指定为备份的文件采取的操作。在升级期间，每个备份文件会使用三个 MD5 哈希来确定所需的操作：一个用于安装的原始文件，一个用于即将安装的新文件，另一个用于系统上实际存在的文件。比较这三个哈希后，可能出现以下场景：

1. **original=X, current=X, new=X**  
   所有三个文件都相同，因此覆盖不会有问题。安装新文件。

2. **original=X, current=X, new=Y**  
   当前文件与原始文件相同，但新文件不同。由于用户从未修改过该文件，且新文件可能包含改进或错误修复，因此安装新文件。

3. **original=X, current=Y, new=X**  
   两个包版本包含完全相同的文件，但系统上的文件已被修改。保持当前文件不变。

4. **original=X, current=Y, new=Y**  
   新文件与当前文件相同。安装新文件。

5. **original=X, current=Y, new=Z**  
   所有三个文件都不同，因此以 .pacnew 扩展名安装新文件并警告用户。用户必须手动将所需的更改合并到原始文件中。

6. **original=NULL, current=Y, new=Z**  
   包之前未安装，并且文件已经存在于系统中。以 .pacnew 扩展名安装新文件并警告用户。用户必须手动将所需的更改合并到原始文件中。

## 示例

1. pacman -Ss ne.hack  
   在包数据库中搜索正则表达式 ne.hack。

2. pacman -S gpm  
   下载并安装 gpm 包及其依赖项。

3. pacman -U /home/user/ceofhack-0.6-1-x86_64.pkg.tar.gz  
   从本地文件安装 ceofhack-0.6-1 包。

4. pacman -Syu  
   更新包列表并随后升级所有包。

5. pacman -Syu gpm  
   更新包列表、升级所有包，然后如果 gpm 未安装则安装它。

## 配置
请参见 [pacman.conf(5)](https://man.archlinux.org/man/pacman.conf.5.en) 以获取有关使用 pacman.conf 文件配置 pacman 的更多详细信息。

## 参见
[alpm-hooks(5)](https://man.archlinux.org/man/alpm-hooks.5.en)，[libalpm(3)](https://man.archlinux.org/man/libalpm.3.en)，[makepkg(8)](https://man.archlinux.org/man/makepkg.8.en)，[pacman.conf(5)](https://man.archlinux.org/man/pacman.conf.5.en)。

更多关于 pacman 及其相关工具的当前信息，请访问 [pacman 官网](https://archlinux.org/pacman/)。

## Bugs
Bugs？你一定是在开玩笑；这个软件里没有 Bug！但是如果我们错了，请在 [GitLab 的议题](https://gitlab.archlinux.org/pacman/pacman/-/issues) 上报告它们，并提供诸如你的命令行、Bug 的性质，甚至包数据库等具体信息。

## 作者
当前维护者：

- Allan McRae &lt;allan@archlinux.org&gt;
- Andrew Gregory &lt;andrew.gregory.8@gmail.com&gt;
- Morgan Adamiec &lt;morganamilo@archlinux.org&gt;

过去的主要贡献者：

- Judd Vinet &lt;jvinet@zeroflux.org&gt;
- Aurelien Foret &lt;aurelien@archlinux.org&gt;
- Aaron Griffin &lt;aaron@archlinux.org&gt;
- Dan McGee &lt;dan@archlinux.org&gt;
- Xavier Chantry &lt;shiningxc@gmail.com&gt;
- Nagy Gabor &lt;ngaba@bibl.u-szeged.hu&gt;
- Dave Reisner &lt;dreisner@archlinux.org&gt;
- Eli Schwartz &lt;eschwartz@archlinux.org&gt;

有关其他贡献者，请在 pacman.git 仓库中使用 git shortlog -s。
