# Firmware and Software Configuration 

The keyboard is designed to work out of the box with software on your computer such as AutoHotKey, which enables powerful macros to be designed as the computer can make context specific decisions regarding what to do, unlike a traditional keyboard.

## Theory of Operation
Hub16 works by 'wrapping' a normal key press with a modifier key, just like CTRL + C or SHIFT + C works on the `c` key on your computer. By using an obscure key such as F24, we can get the same functionality out of our Macro Pad without impeding on the functionality of our standard keyboard (assuming your keyboard does not have a F24 key!). 

When you press a key on the Hub16, it first presses and holds down a modifier key (F24 by default) before pressing and releasing a normal alpha key. It then releases the modifier key and returns to the idle state. As F24 + `a-z` is not a keyboard shortcut nothing occurs, however by using software on the host computer we can detect the F24 press and run the appropriate script. 

## Default Configuration 
```
------------------  ENC1:Clockwise: q
|  ENC1   ENC2   |  ENC1:Anticlockwise: r
| a   b   c   d  |  ENC1:Button: s
| e   f   g   h  |  ENC2:Clockwise: t
| i   j   k   l  |  ENC2:Anticlockwise: u
| m   n   o   p  |  ENC2:Button: v
------------------
```

The default configuration is shown above, with double tapping the bottom right key (`p`) resulting in a layer change to below, where keyboard reset, along with LED control is located. You can return to the base layer by double tapping the bottom right key for a second time. A list of available keycodes can be found [here](https://beta.docs.qmk.fm/using-qmk/simple-keycodes/keycodes).

__NOTE__ some rotary encoders output flipped signals (clockwise instead of counterclockwise), if after assembly your encoders appear to be sending the wrong signals, comment in line 84 in [config.h](../Firmware/hub16/config.h), or alter the setting in your [software](../Software).

```
_______, RGB_MOD, RGB_RMOD,  RGB_TOG,
RGB_VAD, RGB_VAI,  RGB_HUD,  RGB_HUI,
RGB_SAD, RGB_SAI,  _______,  _______, 
_______, _______,  RESET,    TD(TOGGLE_LAYER)
```
__NOTE__ if your board resets without going into bootloader mode upon pressing the above reset key, you will need to either reset the board using the physical reset button on the back of the PCB instead, or reflash the bootloader following [these instructions](Documents/firmware-install.md). 

## Firmware Configuration 

If the above configuration is not to your liking, you can alter the code in `Firmware/hub16/keymaps/default/keymap.c` (although I recommend making your own keymap folder) to make the keyboard do whatever you want. 

To build your new configuration, you will need to do the following:

* Download QMK: `git clone https://github.com/qmk/qmk_firmware --recursive`
* Follow the [build instructions](https://docs.qmk.fm/#/getting_started_build_tools) for your OS to install the toolchain.

The important sections of the code are outlined below:

| Line(s) | Description |  
| --- | ----------- |  
| 19 | Default modifier key |  
| 32:33 | behaviour of layer shift key |  
| 35:51| Keyboard layout and layers|  
| 56:74| Encoder rotation behaviour|  

If you want to use it without all the strange modifier key stuff, I'd suggest modifying the `no_mod` keymap, as it has all of the modifier code removed. 

If you are wondering how all the layouts / layers / code stuff works, I highly recommend the [QMK Documentation](https://docs.qmk.fm/#/).

## VIA Support 

If you would rather use a GUI to configure the board, flash the board with the [VIA](https://caniusevia.com/) keymap (`make hub16:via:flash` or `hub16_via.bin` for QMK Toolbox), and then you will be able to configure the board in VIA.

Unfortunately the rotary encoders cannot be configured from within VIA, and are set to volume / media controls out of the box. Encoder behaviour can be altered in the `/hub16/keymap/via/keymap.c` file and after flashing the board will be updated, whilst allowing the remaining keys to be programmed in VIA.

## Software Configuration 

Use of the software requires understanding of whatever scripting language is being used, such as AutoHotKey, or Python if AutoKey is being used on Linux. I have provided my scripts as inspiration, and am more than happy to answer questions on how to implement features if required.

Taran from Linus Tech Tips has a great [video](https://youtu.be/GZEoss4XIgc?t=346) which covers the whole process, from building the firmware from source to his uniquely designed macros. 

## Software Installation 

### Windows
* Install [AHK](https://www.autohotkey.com/)
    * Copy the example script from `Software/hub16.ahk` into a folder of your choice. 
    	* I highly recommend placing a shortcut to that file in `%APPDATA%/Roaming/Microsoft/Windows/Start Menu/Programs/Start-up` to ensure it runs at boot.
    * Customise the script to your hearts content! 

### macOS
* Install [Brew](https://brew.sh/).
* Open Terminal and install [Karabiner-Elements](https://karabiner-elements.pqrs.org/) with ```brew cask install karabiner-elements```.
*  __NOTE__ recent versions of firmware have an updated VID:PID (to enable VIA support), which Karabiner uses to identify the Hub16 keyboard instead of F24. 
   *  Due to this, pick the correct configuration file for the VID:PID pair of your Hub16. 
   *  If firmware on your board is updated, it may require changing the VID:PID in the settings.
* Place example config `Software/karabiner-0x*.json` in your local `~.config/karabiner` folder.

### Linux
* [AutoKey](https://github.com/autokey/autokey) can be used in Linux (tested Ubuntu 19.10) to do the keyboard remapping. 
* Install AutoKey from [here](https://github.com/autokey/autokey/wiki/Installing#debian-and-derivatives) as the apt-get version old and no longer functioning.
* Getting it to work isn't as easy as other OS's, but following the below instructions got it working on my machines.
	* We will remap F24 to `Hyper_R`, and then map `Hyper_R` to `mod3` which AutoKey can then detect.   
	* In a terminal, enter `modmap` to confrim `mod3` and `Hyper_R` are unassigned.
	* Create a file in your home directory with the name `.Xmodmap` and the following contents

		```
		keycode 202 = Hyper_R
		add mod3 = Hyper_R
		```
	* Update xmodmap with ```xmodmap ~/.Xmodmap``` and then confirm that ```mod3``` has been updated. Then add it to your ```~./bashrc```, as this will allow the keyboard to work **only after a shell is opened**. I tried for hours to get it working at startup with no luck, if you can figure out a way please let me know! 
	
	* Load the AutoKey files found in ```Firmware/autokey-hub16``` and it should work! Note: you may need to make a new folder from within Autokey, then copy your files in as it is particular about locating new files.
	* You will also need to configure AutoKey to start at boot, which can be done from `Edit -> Preferences`
* For volume and media control as configured in my example code, [playerctl](https://github.com/altdesktop/playerctl) and ```alsa-utils``` must be installed

* Bugs / Strange behaviour
	* As mentioned above, I have been unable to get it working at boot, so you need to open a new shell every time the computer is turned on, or a USB HID device is plugged / unplugged. This includes whenever the firmware is updated.
	* If the last thing done on the computer is using your normal keyboard before the Hub16, you will need to either press the macro twice, or click on your screen before running it. 
	* I don't understand why any of these occur, if you know / have a fix please let me know or send a pull request. 

* Alternate solution
	* If the above bugs are too annoying, you can simply hard code the key presses into the keyboard which skips the software layer, but at the trade off of not being able to dynamically switch between keys depending on the active window. 
		* To do this, uncomment the commented ```process_record_user``` function in `Firmware/hub16/keymaps/no_mod/keymap.c`, and place your required key combinations in that function. 
		* By using layers on the keyboard (similar to the LED control layer) you could have different layers on the keyboard for different applications each sending different key combinations, which would allow you to manually alter the keyboard behaviour as you move between software applications. 
		* If you have any questions or issues with getting this to work, please reach out and I'll lend a hand. 