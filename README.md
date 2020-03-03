# LYWSD03MMC

TLSR8251 SoC, LYWSD03MMC [firmware here](https://github.com/custom-components/sensor.mitemp_bt/issues/7#issuecomment-570829260)

Advertisements example ([full hcidump](lywsd03mmcdump.txt), [without duplicates](lywsd03mmcdump_noduplicates.txt)):

```
Dump duration - 30 minutes

Temperature: 22.2 to 21.8
Humidity:     36  to  37

---------------------------------------------------------------------------------------------------
len  AD  Xiaomi Frame Device  Frame     MAC (LE)        ----------------PAYLOAD--------------  RSSI
    type  UUID   ctrl  type    cnt                          cypher      ext.cnt     MAC tag
---------------------------------------------------------------------------------------------------
0F   16  95 FE  30 58  5B 05   BF   6B 87 2F 38 C1 A4   08                                      E3
1A   16  95 FE  58 58  5B 05   C0   6B 87 2F 38 C1 A4   63 02 92 E6 37  21 00 00  E5 E9 32 F7   E2
0F   16  95 FE  30 58  5B 05   C1   6B 87 2F 38 C1 A4   08                                      E3
1A   16  95 FE  58 58  5B 05   C2   6B 87 2F 38 C1 A4   11 B4 AF 05 28  21 00 00  8B C7 85 E9   E3
0F   16  95 FE  30 58  5B 05   C3   6B 87 2F 38 C1 A4   08                                      E2
1A   16  95 FE  58 58  5B 05   C4   6B 87 2F 38 C1 A4   E3 28 24 BC 5F  21 00 00  BE 1E 41 1A   E2
0F   16  95 FE  30 58  5B 05   C5   6B 87 2F 38 C1 A4   08                                      E2
1A   16  95 FE  58 58  5B 05   C6   6B 87 2F 38 C1 A4   9C 02 27 6B E4  21 00 00  D5 5E 58 EF   E2
0F   16  95 FE  30 58  5B 05   C7   6B 87 2F 38 C1 A4   08                                      DC
19   16  95 FE  58 58  5B 05   C8   6B 87 2F 38 C1 A4      8E 95 EE 1F  21 00 00  BF 6A C6 94   DC -- apparantly, battery
0F   16  95 FE  30 58  5B 05   C9   6B 87 2F 38 C1 A4   08                                      E2
1A   16  95 FE  58 58  5B 05   CA   6B 87 2F 38 C1 A4   4F DB 8C 38 30  21 00 00  6E 64 71 E6   DC
0F   16  95 FE  30 58  5B 05   CB   6B 87 2F 38 C1 A4   08                                      E2
1A   16  95 FE  58 58  5B 05   CC   6B 87 2F 38 C1 A4   D6 02 B0 D8 A4  21 00 00  03 7B C1 69   E2
0F   16  95 FE  30 58  5B 05   CD   6B 87 2F 38 C1 A4   08                                      E2

*Frame counter + "ext. counter" - 32bit word, ext.cnt starts from 00 00 00 and increases by 1 when frame cnt. overflows
```

More examples: [more](https://github.com/custom-components/sensor.mitemp_bt/issues/7#issuecomment-568723038), [raw hcidump](https://github.com/custom-components/sensor.mitemp_bt/issues/7#issuecomment-566897865), [filtered esp32log](https://github.com/custom-components/sensor.mitemp_bt/issues/7#issuecomment-573395064)

### "08 payload" frame:

```
Frame ctrl - 58 30

 0101      10       0            0       0      0        1         1         0          000

version   auth    binding   registered  mesh   data  capability   MAC        is       reserved
  (5)     mode   valid.req?     flag    flag   flag     flag      flag    encrypted


Capability - 08

  00    0        01          000
rsrvd  I/O   bondability     not
       cap.    "front"       used
```

### "Data" frame:

```
Frame ctrl - 58 58

 0101      10       0            0       0      1        0         1         1          000

version   auth    binding   registered  mesh   data  capability   MAC        is       reserved
  (5)     mode   valid.req?     flag    flag   flag     flag      flag    encrypted

```

### Encryption:

```
AES 128 CCM (CBC+MAC)

Data: regular xiaomi data object
Key: "bind_key" or "beaconkey"
AAD: seems like fixed to 0x11
MAC: 4 bytes
nonce: device MAC, device type, frame cnt, ext. cnt
```

decrypted example:

```shell
#... 85 75 17 E2 AA  00 00 00  18 B2 A9 DA   E2
b' 06 10  02  56 01'
    hum       34,2%

#... AC 84 32 95     00 00 00  0E 58 B5 7F   E3
b' 0a 10  01  64'
   batt )    100%
```

* Probably in the example above (dump 30 minutes) 2-3 measurements for temperature and hunidity and 1 frame with battery info

## Characteristics snapshot

![adafruit bluefruit snapshot](characteristics_snapshot.png)

## some HTTPS API responses

while adding sensor to MiHome app:

```json
{"mac":"A4:C1:38:AB:DD:EB","model":"miaomiaoce.sensor_ht.t2","token":"12byteshex","did":"blt.3.11usobjecls00"}
{"did":"blt.3.11usobjecls00","token":"another12bytes","props":[{"type":"prop","key":"bind_key","value":"16bytes"},{"type":"prop","key":"smac","value":"A4:C1:38:AB:DD:EB"}]}
```

## MiHome RN-plugin

[miaomiaoce.sensor_ht.t2](https://github.com/wiecosystem/Bluetooth/files/4230785/miaomiaoce.sensor_ht.t2.zip)
