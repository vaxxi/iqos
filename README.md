# IQOS

Poking the IQOS vaping device.

![IQOS device](https://github.com/vaxxi/iqos/raw/master/iqos.jpg)


## Initial probing of the device

### dmesg

```
[   21.406903] usb 3-3: new full-speed USB device number 2 using xhci_hcd
[   21.538848] usb 3-3: New USB device found, idVendor=2759, idProduct=0003
[   21.538853] usb 3-3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[   21.538871] usb 3-3: Product: Zurich FPD 4.x Charger
[   21.538874] usb 3-3: Manufacturer: Philip Morris Products S.A.
[   21.538876] usb 3-3: SerialNumber: AAAABBBBCCCCDDDD
[   31.538246] hid-generic 0003:2759:0003.0004: timeout initializing reports
[   31.538444] hid-generic 0003:2759:0003.0004: hiddev0,hidraw3: USB HID v1.01 Device [Philip Morris Products S.A. Zurich FPD 4.x Charger] on usb-0000:00:14.0-3/input0
```

The Vendor ID is not found in the [USB vendor database](https://usb-ids.gowdy.us/read/UD?restrict=2), (serial number is altered for privacy, it's obviously supposed to be a string unique to your device). There's nothing online about the device
string except [a thread on a Japanese forum](https://translate.google.ro/translate?hl=en&sl=ja&u=http://yomogi.2ch.net/test/read.cgi/smoking/1461584144/&prev=search).

### lsusb

```
Bus 003 Device 002: ID 2759:0003  
```

Nothing extra listed here.

### /sys/kernel/debug/usb/devices

```
T:  Bus=03 Lev=01 Prnt=01 Port=02 Cnt=01 Dev#=  4 Spd=12   MxCh= 0
D:  Ver= 2.00 Cls=00(>ifc ) Sub=00 Prot=00 MxPS= 8 #Cfgs=  1
P:  Vendor=2759 ProdID=0003 Rev= 2.00
S:  Manufacturer=Philip Morris Products S.A.
S:  Product=Zurich FPD 4.x Charger
S:  SerialNumber=AAAABBBBCCCCDDDD
C:* #Ifs= 1 Cfg#= 1 Atr=c0 MxPwr=500mA
I:* If#= 0 Alt= 0 #EPs= 2 Cls=03(HID  ) Sub=00 Prot=00 Driver=usbhid
E:  Ad=81(I) Atr=03(Int.) MxPS=  64 Ivl=1ms
E:  Ad=01(O) Atr=03(Int.) MxPS=  64 Ivl=1ms
```

Raw output from the kernel interface. 

### lsusb -v

```
Bus 003 Device 004: ID 2759:0003  
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0         8
  idVendor           0x2759 
  idProduct          0x0003 
  bcdDevice            2.00
  iManufacturer           1 Philip Morris Products S.A.
  iProduct                2 Zurich FPD 4.x Charger
  iSerial                 3 AAAABBBBCCCCDDDD
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           41
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          4 Standard configuration
    bmAttributes         0xc0
      Self Powered
    MaxPower              500mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      0 No Subclass
      bInterfaceProtocol      0 None
      iInterface              5 Zurich FPD 4.x Charger HID
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.01
          bCountryCode            0 Not supported
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength      36
         Report Descriptors: 
           ** UNAVAILABLE **
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x01  EP 1 OUT
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               1
Device Status:     0x0001
  Self Powered
```

Verbose output from `lsusb`. The USB standard is quite complicated; simply
said, our device has two one way "communication ports" (the two endpoints
listed above), one for data input at address 0x81 and one for data output at
address 0x01.

## is there anybody out there

Install `python-usb`:

```
apt-get install python-usb
```

Run the attached `hiqos.py`. The device seems to alternatively return 2
strings (further execution will keep repeating these two strings):

```
array('B', [63, 1, 0, 211, 225, 222, 150, 83, 118, 235, 189, 189, 219, 114, 27, 77, 182, 236, 156, 116, 151, 102, 175, 87, 23, 245, 108, 236, 28, 211, 188, 195, 187, 223, 197, 243, 237, 150, 246, 190, 115, 106, 127, 107, 255, 63, 233, 129, 140, 9, 243, 219, 172, 155, 189, 238, 87, 94, 4, 27, 0, 0, 0, 16])
array('B', [63, 1, 0, 6, 251, 210, 223, 205, 123, 255, 175, 249, 250, 125, 245, 190, 109, 123, 239, 126, 11, 59, 253, 250, 45, 251, 151, 124, 138, 70, 57, 197, 221, 175, 228, 99, 188, 109, 232, 117, 164, 159, 55, 247, 242, 29, 127, 237, 7, 155, 166, 153, 14, 215, 166, 126, 238, 241, 247, 122, 222, 55, 127, 103])
```

Sadly, the strings make no sense when converted to ASCII. First three bytes
are identical, the rest are different.


[1] https://github.com/walac/pyusb/blob/master/docs/tutorial.rst

[2] http://www.beyondlogic.org/usbnutshell/usb4.shtml

[3] http://www.usbmadesimple.co.uk/ums_5.htm

[4] http://newae.com/files/Hackaday_USSSSSBTalkingUSBFromPython_OFlynn.pdf

