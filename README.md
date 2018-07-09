# 安卓性能监控小工具

#### 流量监控
- 获取被测试App的UID：
<br>原理：执行adb Shell读取/data/system/packages.list，并通过apk的<PKGNAME>找到对应的UID
<br>例子：cat /data/system/packages.list | grep com.gift.android
<br>输出值：com.gift.android 10055 0 /data/data/com.gift.android default 1028,1015,1023,3003,3002,3001
<br>其中10055即为App的UID

- 获得App的接收流量（tcp_rcv）
<br>原理：执行adb Shell读取/proc/uid_stat/<UID>/tcp_rcv,其中<UID>为被测试App的UID
<br>例子：cat /proc/uid_stat/10055/tcp_rcv
<br>输出值：1566512
<br>1566512即为接收流量的值

- 获得App的发送流量（tcp_snd）
<br>原理：执行adb Shell读取/proc/uid_stat/<UID>/tcp_snd,其中<UID>为被测试App的UID
<br>例子：cat /proc/uid_stat/10055/tcp_snd
<br>输出值：486847
<br>486847即为发送流量的值

- 总体的流量数值计算
<br>采集到前后两次流量数值后，即可计算得到某段时间内耗费的总流量。
<br>例子
<br>root@android:/ # cat /proc/uid_stat/10055/tcp_rcv
<br>1566512
<br>root@android:/ # cat /proc/uid_stat/10055/tcp_snd
<br>486847
<br>--- 在app上执行某些操作----
<br>root@android:/ # cat /proc/uid_stat/10055/tcp_rcv
<br>1978338
<br>root@android:/ # cat /proc/uid_stat/10055/tcp_snd
<br>548128
<br>计算：该端时间内消耗总流量 = (548128 + 1978338) - (486847 + 1566512)  bytes

- 其他流量测法，通过 tcpdump 抓包，再通过 wireshake 直接读取包信息来获得流量。


#### 总内存和进程内存监控
- 总内存获取
<br>原理：adb shell读取/proc/meminfo
<br>例子： cat  /proc/meminfo
<br>MemFree:          560352 kB
<br>Buffers:          136576 kB
<br>Cached:           418460 kB
<br>Active:           626888 kB
<br>Inactive:         314720 kB
<br>Active(anon):     354828 kB
<br>Inactive(anon):    39584 kB
<br>Active(file):     272060 kB
<br>Inactive(file):   275136 kB
<br>Dirty:                52 kB
<br>Writeback:             0 kB
<br>Mapped:           135096 kB
<br>Slab:              23068 kB

- 被测试App进程内存
<br>原理：adb shell执行 dumpsys meminfo <PID>其中<PID>为被测进程的PID
<br>例子：dumpsys meminfo 15045<br>
|             | Pss Total | Private Dirty | Private Clean | Swapped Dirty | Heap Size | Heap Alloc | Heap Free |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| Native Heap |   5061    |     5028      |       0       |       0       |   16480   |   13207    |    124    |
| Dalvik Heap |   28780   |     40972     |       0       |       0       |   41156   |   40069    |   1087    |
 
#### 总CPU和进程CPU监控
- CPU获取
<br>原理：dumpsy meminfo取进程内存时额外CPU开销大，会导致dumpsys cpu不准；系统自带top命令单次获取数据时间长且精度不够，利用busybox获取CPU使用情况(在获取内存数据前先取CPU)，执行adb Shell命令：busybox top -b -n 1
<br>例子：
<br>adb push busybox /data/local/tmp
<br>adb shell chmod 755 /data/local/tmp/busybox

#### 当前时刻显示的Activity
<br>原理：adb shell执行，dumpsys window w|grep mFocusedApp|busybox awk '{print $5}'|busybox tr -d '}'



#### 当前时刻的时间
<br>原理：adb shell执行，date +%Y/%m/%d" "%H:%M:%S

#### 系统启动后运行时间
<br>原理：adb shell执行，busybox awk -F. 'NR==1{print $1}' /proc/uptime  依此为基础来分析执行监控的时刻及准确的获取数据间隔


