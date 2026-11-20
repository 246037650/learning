## . 先检查蓝牙适配器

```
# 查看蓝牙硬件，一般是 hci0
hciconfig -a
# 或者
lsusb
```
└─$ hciconfig -a
hci0:   Type: Primary  Bus: USB
        BD Address: 04:7F:0E:90:11:01  ACL MTU: 1021:9  SCO MTU: 255:4
        UP RUNNING
        RX bytes:742 acl:0 sco:0 events:53 errors:0
        TX bytes:2163 acl:0 sco:0 commands:53 errors:0
        Features: 0xbf 0xee 0xcd 0xfe 0xdb 0xff 0x7b 0x87
        Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV3
        Link policy: RSWITCH SNIFF
        Link mode: PERIPHERAL ACCEPT
        Name: 'wlan0'
        Class: 0x00010c
        Service Classes: Unspecified
        Device Class: Computer, Laptop
        HCI Version: 5.0 (0x9)  Revision: 0x202
        LMP Version: 5.0 (0x9)  Subversion: 0x2
        Manufacturer: Barrot Technology Limited (2279)
如果没起来，启动蓝牙服务：

```
sudo systemctl start bluetooth
sudo systemctl enable bluetooth
```

## 2. bluetoothctl（推荐，同时扫经典蓝牙 + BLE 低功耗）

```
bluetoothctl
```

进入交互 shell，依次输入：

```
power on
scan on
```

- `scan on`：同时扫 BR/EDR 经典蓝牙 + BLE
- `scan le`：只扫 BLE（充电桩、手环、Beacon）
- `scan bredr`：只扫传统蓝牙耳机、串口模块
- 看到设备 MAC、名称后：

```
devices
info AA:BB:CC:DD:EE:FF
```
[bluetoothctl]> devices
Device 5E:60:BA:5F:22:4B 5E-60-BA-5F-22-4B
Device 98:5F:41:73:66:43 LAPTOP-7BRDOUN6
Device 90:E2:FC:2E:E5:A0 BYDPAD-202611160001
Device 6B:79:E5:14:F1:D8 6B-79-E5-14-F1-D8

在 bluetoothctl 里直接 `info MAC` 看 `AddressType`来确定是 BR/EDR 还是 BLE：
90:E2:FC:2E:E5:A0 (public)是 BR/EDR 经典蓝牙，6B:79:E5:14:F1:D8 (random)是 BLE 低功耗。
```

[bluetoothctl]> info 90:E2:FC:2E:E5:A0
Device 90:E2:FC:2E:E5:A0 (public)
        Name: BYDPAD-202611160001
        Alias: BYDPAD-202611160001
        Class: 0x00240420 (2360352)
        Icon: audio-card
        Paired: no
        Bonded: no
        Trusted: no
        Blocked: no
        Connected: no
        WakeAllowed: no
        LegacyPairing: no
        CablePairing: no
        UUID: Phonebook Access Client   (0000112e-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
        UUID: Handsfree                 (0000111e-0000-1000-8000-00805f9b34fb)
        UUID: Generic Audio             (00001203-0000-1000-8000-00805f9b34fb)
        UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
        UUID: Message Notification Se.. (00001133-0000-1000-8000-00805f9b34fb)
        UUID: Headset                   (00001108-0000-1000-8000-00805f9b34fb)
        UUID: Human Interface Device... (00001124-0000-1000-8000-00805f9b34fb)
        UUID: PnP Information           (00001200-0000-1000-8000-00805f9b34fb)
        UUID: Vendor specific           (ec884348-cd41-40a2-9727-575d50bf1fd3)
        UUID: Vendor specific           (ffcacade-afde-cade-defa-cade00000000)
停止扫描：

```
scan off
exit
```
连接是
[bluetoothctl]> pair 90:E2:FC:2E:E5:A0
Attempting to pair with 90:E2:FC:2E:E5:A0
[CHG] Device 90:E2:FC:2E:E5:A0 Connected: yes
Request confirmation
[agent] Confirm passkey 383312 (yes/no): yes
[CHG] Device 90:E2:FC:2E:E5:A0 Bonded: yes
[CHG] Device 90:E2:FC:2E:E5:A0 WakeAllowed: yes