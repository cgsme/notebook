# df

Linux df（disk free） 命令用于显示目前在 Linux 系统上的**文件系统磁盘使用情况**统计。

## 语法

```bash
df [选项]... [FILE]...
```

## 参数说明

```bash
-a, --all 包含所有的具有 0 Blocks 的文件系统
--block-size={SIZE} 使用 {SIZE} 大小的 Blocks
-h, --human-readable 使用人类可读的格式(预设值是不加这个选项的...)
-H, --si 很像 -h, 但是用 1000 为单位而不是用 1024
-i, --inodes 列出 inode 资讯，不列出已使用 block
-k, --kilobytes 就像是 --block-size=1024
-l, --local 限制列出的文件结构
-m, --megabytes 就像 --block-size=1048576
--no-sync 取得资讯前不 sync (预设值)
-P, --portability 使用 POSIX 输出格式
--sync 在取得资讯前 sync
-t, --type=TYPE 限制列出文件系统的 TYPE
-T, --print-type 显示文件系统的形式
-x, --exclude-type=TYPE 限制列出文件系统不要显示 TYPE
-v (忽略)
--help 显示这个帮手并且离开
--version 输出版本资讯并且离开
```

## 示例

显示文件系统的磁盘使用情况统计：

```bash
# df 
Filesystem     1K-blocks    Used     Available Use% Mounted on 
/dev/sda6       29640780 4320704     23814388  16%     / 
udev             1536756       4     1536752    1%     /dev 
tmpfs             617620     888     616732     1%     /run 
none                5120       0     5120       0%     /run/lock 
none             1544044     156     1543888    1%     /run/shm 
```

第一列指定文件系统的名称，第二列指定一个特定的文件系统1K-块1K是1024字节为单位的总内存。用和可用列正在使用中，分别指定的内存量。

使用列指定使用的内存的百分比，而最后一栏"安装在"指定的文件系统的挂载点。

df也可以显示磁盘使用的文件系统信息：

```bash
# df test 
Filesystem     1K-blocks    Used      Available Use% Mounted on 
/dev/sda6       29640780    4320600   23814492  16%       / 
```

用一个-i选项的df命令的输出显示inode信息而非块使用量。

```bash
df -i 
Filesystem      Inodes    IUsed    IFree     IUse% Mounted on 
/dev/sda6      1884160    261964   1622196   14%        / 
udev           212748     560      212188    1%         /dev 
tmpfs          216392     477      215915    1%         /run 
none           216392     3        216389    1%         /run/lock 
none           216392     8        216384    1%         /run/shm 
```

显示所有的信息:

```bash
# df --total 
Filesystem     1K-blocks    Used    Available Use% Mounted on 
/dev/sda6       29640780 4320720    23814372  16%     / 
udev             1536756       4    1536752   1%      /dev 
tmpfs             617620     892    616728    1%      /run 
none                5120       0    5120      0%      /run/lock 
none             1544044     156    1543888   1%      /run/shm 
total           33344320 4321772    27516860  14% 
```

我们看到输出的末尾，包含一个额外的行，显示总的每一列。

-h选项，通过它可以产生可读的格式df命令的输出：

```bash
# df -h 
Filesystem      Size  Used   Avail Use% Mounted on 
/dev/sda6       29G   4.2G   23G   16%     / 
udev            1.5G  4.0K   1.5G   1%     /dev 
tmpfs           604M  892K   603M   1%     /run 
none            5.0M     0   5.0M   0%     /run/lock 
none            1.5G  156K   1.5G   1%     /run/shm 
```

我们可以看到输出显示的数字形式的'G'（千兆字节），"M"（兆字节）和"K"（千字节）。

这使输出容易阅读和理解，从而使显示可读的。请注意，第二列的名称也发生了变化，为了使显示可读的"大小"。 
