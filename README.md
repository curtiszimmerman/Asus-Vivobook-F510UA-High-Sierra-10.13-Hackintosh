# Asus Vivobook F510UA
This build running on MacOs X 10.13.4
This is a fork of https://github.com/tctien342/Asus-Vivobook-S510UA-High-Sierra-10.13-Hackintosh. I have edited the DSDT to work with F510UA. The laptops are so similar that the rest of the colver folder works out of the box.

![Alt text](https://ivanov-audio.com/wp-content/uploads/2014/01/Hackintosh-Featured-Image.png)

# Info
	1. The Apple kexts AppleIntelLpssI2C.kext, AppleIntelLpssI2CController.kext, AppleHPM.kext must be deleted or the system with KP.
	2. aDummyHDA.kext must be installed to /S/L/E for sound patching to work. This is easy to do by using the Kext Utility.app in the Tools folder.

# System specification
    1.Name:           Asus Vivobook F510UA
    2.CPU:            Intel Core i5-8250U
    3.Graphic:        Intel UHD620
    4.Wifi:           Intel Dual Band Wireless-AC  - with bluetooth
    5.Card Reader:    Realtek_CardReader(RTL8411B_RTS5226_RTS5227) - Not confirmed
    6.Camera:         AzureWave VGA
    7.Audio:          Conexant Audio CX8050
    8.Touchpad:       ELAN1300
    9.Bios Version:   N/A

# Tested working
    1. Video UHD620 with hardware acceleration
    2. Audio Speakers
    3. Sleep and wake
    4. Trackpad - including 2 and 4 finger gestures
    5. Keyboard Including Fn keys
    6. Screen brightness control

# Know problems
    1.  HDMI Audio - Not Tested
    2.  Wifi card problem - hackintosh not support Intel card yet
    3.  Bluetooh card
    4. Card reader not tested

# Using DSDT Patch
    1. Asus N55SL/Vivobook for battery
    2. Fix _WAK Arg0 v2
    3. USB _PRW 0x6D Skylake (insant wake) for sleep fix
    4. Shutdown fix v2
    5. Rename _DSM methods to XDSM
    6. All Asus FN patch in necessary folder

# New VoodooI2C
    -Git: [VoodooI2C ELAN1300](https://github.com/tctien342/VoodooI2C)

# I2C PATCH
    1. Replace all scope _SB.PCI0.I2C1 with this:
    ```
        Scope (_SB.PCI0.I2C1)
        {
            Device (ETPD)
            {
                Name (_ADR, One)  // _ADR: Address
                Name (ETPH, Package (0x16)
                {
                    "ELAN1200", 
                    "ELAN1201", 
                    "ELAN1203", 
                    "ELAN1200", 
                    "ELAN1201", 
                    "ELAN1300", 
                    "ELAN1301", 
                    "ELAN1300", 
                    "ELAN1301", 
                    "ELAN1000", 
                    "ELAN1200", 
                    "ELAN1200", 
                    "ELAN1200", 
                    "ELAN1200", 
                    "ELAN1200", 
                    "ELAN1203", 
                    "ELAN1203", 
                    "ELAN1201", 
                    "ELAN1300", 
                    "ELAN1300", 
                    "ELAN1200", 
                    "ELAN1300"
                })
                Name (FTPH, Package (0x05)
                {
                    "FTE1001", 
                    "FTE1200", 
                    "FTE1200", 
                    "FTE1300", 
                    "FTE1300"
                })
                Name (SBFG, ResourceTemplate ()
                {
                    GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,
                        "\\_SB.PCI0.GPI0", 0x00, ResourceConsumer, ,
                        )
                        {   // Pin list
                            0x0055
                        }
                })
                Method (_HID, 0, NotSerialized)  // _HID: Hardware ID
                {
                    If (And (TPDI, 0x04))
                    {
                        Return (DerefOf (Index (ETPH, TPHI)))
                    }

                    If (And (TPDI, 0x10))
                    {
                        Return (DerefOf (Index (FTPH, TPHI)))
                    }

                    Return ("ELAN1010")
                }

                Name (_CID, "PNP0C50")  // _CID: Compatible ID
                Name (_UID, One)  // _UID: Unique ID
                Name (_S0W, 0x03)  // _S0W: S0 Device Wake State
                Method (_DSM, 4, NotSerialized)  // _DSM: Device-Specific Method
                {
                    If (LEqual (Arg0, ToUUID ("3cdff6f7-4267-4555-ad05-b30a3d8938de") /* HID I2C Device */))
                    {
                        If (LEqual (Arg2, Zero))
                        {
                            If (LEqual (Arg1, One))
                            {
                                Return (Buffer (One)
                                {
                                    0x03                                           
                                })
                            }
                            Else
                            {
                                Return (Buffer (One)
                                {
                                    0x00                                           
                                })
                            }
                        }

                        If (LEqual (Arg2, One))
                        {
                            Return (One)
                        }
                    }
                    Else
                    {
                        Return (Buffer (One)
                        {
                            0x00                                           
                        })
                    }
                }

                Method (_STA, 0, NotSerialized)  // _STA: Status
                {
                    Return (0x0F)
                }

                Method (_CRS, 0, Serialized)  // _CRS: Current Resource Settings
                {
                    Name (SBFB, ResourceTemplate ()
                    {
                        I2cSerialBusV2 (0x0015, ControllerInitiated, 0x00061A80,
                            AddressingMode7Bit, "\\_SB.PCI0.I2C1",
                            0x00, ResourceConsumer, , Exclusive,
                            )
                    })
                    Return (ConcatenateResTemplate (SBFB, SBFG))
                }
            }
        }
    ```

    2. Add Window10 Patch:
    ```
        # Windows 10 DSDT Patch for VoodooI2C
        # Allows I2C controllers and devices to be discovered by OS X.
        # Based off patches written by RehabMan
        into_all method code_regex If\s+\([\\]?_OSI\s+\(\"Windows\s2015\"\)\) replace_matched begin If(LOr(_OSI("Darwin"),_OSI("Windows 2015"))) end;
    ```

    3. Add GPIO Controller Enable
    ```
        # GPI0 Status patch
        # Ensures that OS X can enumerate the GPI0 controller
        # Written and maintained by Alexandre Daoud

        into method label _STA parent_label GPI0 replace_content begin

        Return (0x0F)

        end;
    ```


