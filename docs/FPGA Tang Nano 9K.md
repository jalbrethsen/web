#### Open IDE

```
/home/justin/packages/gowin_ide/IDE/bin/gw_ide
```

#### Flash picotiny example

```
sudo ~/packages/openFPGALoader/build/openFPGALoader -b tangnano9k -f ~/packages/gowin_ide/TangNano-9K-example/picotiny/picotiny.fs
screen /dev/ttyUSB1 115200
```

#### Future idea

Make something like the esp32 turnkey modules so this can be easily integrated into other IC designs.