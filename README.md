# nodepm
NOne-Desktop-Environment Power Manager

```
Usage: nodepm ARGV

ARGV:
need root permission:

 init					init the nodepm daemon
 daemon					mainly for testing and debug, please run init argument instead if you use systemd as init

need daemon running, do not need root permission:

 backlight				change or show backlight of screen, need daemon running
   backlight up				turn up backlight
   backlight down			turn up backlight
   backlight set VALUE			set backlight to given VALUE which should be between 0 and 1
   backlight get			get current backlight
 timer					change or show mode of input timing monitor mode
   timer on				set timing mode on
   timer off				set timing mode off
   timer switch				switch timing mode between on and off
   timer status				show current mode of input timing monitor 
 battery				show status of battery and ac power

bash variables:
all these bash variables can be modified in /etc/nodepm/env.conf. adding them before CMDLINE is also possible but not recommended:
 
 Bat_Function				the function to get battery and ac power status, now supports 'sys' and 'acpi', 'sys' as default
 RefreshTime				refresh time of daemon loop, 2 as default
 Config_Mode				'double' to switch config between battery mode and ac mode, 'one' for a single config. configs can be found under /etc/nodepm. 'double' as default
 Gpu_Backlight				'all' to take over all the gpu backlight files, otherwise any directory under /sys/class/backlight can be use as value here. 'all' as default
 Backlight_Unit				backlight unit when turning up and down which should be between 0 and 1, 0.075 as default, recommended between 0.05 and 0.2
 Backlight_Idle				default backlight to be set when idle ( Notice: not screen off! ) , also the expected lower limit of idling backlight brightness. it should be between 0 and 1, 0.25 as default
 Inhibit_List				what to inhibit by 'systemd-inhibit', 'default' to let nodepm set it smartly, 'all' to inhibit all, otherwise please read 'man systemd-inhibit'. 'default' as default
 Debug_Mode				debug mode, 1 to turn on, 0 for off as default
 Multirun_Mode				0 to ban run more than one nodepm daemon at the same time, 1 to igore current nodepm daemon and 2 to remove and reset current run file. 0 as default
 BatteryDaemon				'on' to run battery daemon, 'off' to skip it. when 'on', /etc/nodepm/*/battery/conf will also be read. 'on' as default. Notice 'on' is needed for Config_Mode 'double'
 LidDaemon				'on' to run lid daemon, 'off' to skip it. when 'on', /etc/nodepm/*/lid/conf will also be read. 'on' as default 
 TimerDaemon				'on' to run timing daemon, 'off' to skip it. when 'on', /etc/nodepm/*/timer/conf will also be read. 'on' as default 
 BacklightDaemon			'on' to run backlight daemon, 'off' to skip it. when 'on', /etc/nodepm/*/backlgiht/conf will also be read. 'on' as default
 Wake_Fifo				'on' to start a daemon to hear a specific fifo to wake from idle, 'off' to skip it, 'on' as default
 
variables below , please just read them in /etc/nodepm but do not change:
 Ac_Status				status of ac power
 Battery_Status				status of battery power
 Battery_Capacity			capacity of battery
 Battery_Time				time prediction of battery power

config files:
all files here are bash scripts

 /etc/nodepm/init.exec			bash script to source and run when daemon inits
 /etc/nodepm/loop.exec			bash script to source and run in every daemon loop
 /etc/nodepm/env.conf			universal config for nodepm, sourced before daemon inits
 
 /etc/nodepm/bat/*			config in use under Config_Mode 'double' and ac power off
 /etc/nodepm/ac/*			config in use under Config_Mode 'double' and ac power on
 /etc/nodepm/one/*			config in use under Config_Mode 'one'
   backlight/
    conf				config soruced when BacklightDaemon='on'
    exec				bash script to source and run when backlight is changed
   battery/
    conf				config soruced when BatteryDaemon='on'
    ac					bash script to source and run when switching to ac power. only the one under /etc/nodepm/ac is valid.
    bat					bash script to source and run when switching to battery power. only the one under /etc/nodepm/bat is valid.
    */					directories under battery/ are those scipts to run at specific power status
     exec				bash script to source and run under certain condition
     conf				config for when to source and run script
      VALUES:
       Once				Once='yes' (default) to run only once when condition once trigers, Once='no' to run always when condition true
       UpperLimit			set the upper limit percent of battery capacity to run the script, 100 as default
       LowerLimit			set the lower limit percent of battery capacity to run the script, 0 as default
   lid/
    conf				config soruced when LidDaemon='on'
    open				bash script to source and run when detect lid open
    close				bash script to source and run when detect lid close
   timer/
    conf				config soruced when TimerDaemon='on'
    wakeup				bash script to source and run when detect new input after waiting for some time
    */					directories under timer/ are those scipts to run at specific power status
     exec				bash script to source and run under certain condition
     conf				config for when to source and run script
      VALUES:
       Once				Once='yes' (default) to run only once when condition once trigers, Once='no' to run always when condition true
       TimeLimit			set the lower time limit of idle time waited to run the script

internal functions:
can be used in bash scripts in /etc/nodepm

 INTERNAL_user_run CMDLINE		run CMDLINE as users rather than root
 INTERNAL_backlight_up			turn up backlight
 INTERNAL_backlight_down		turn down backlight
 INTERNAL_backlight_save		save current backlight
 INTERNAL_backlight_off			save current backlight and turn off baclight
 INTERNAL_backlight_idle [BRIGHTNESS]	save current backlight and turn it down to [BRIGHTNESS], if [BRIGHTNESS] is absent then use value of Backlight_Idle. Notice: [BRIGHTNESS] should be no less than Backlight_Idle
 INTERNAL_backlight_wake		wake up screen from backlight off or idle

temporary files in /run/nodepm:
whetehr these files are availible deciding on configuration

these files can be both read and edited:
 input_monitor_mode			mode of timer daemon, can be 0 r 1

these files can only be read but please do not edit:
 ac					status of ac power 
 bat					status of battery 
 battery				same as "/run/nodepm/"bat
 lid					status of lid
 backlight_current			backlight of current screen
 input_monitor_wait_time 		time that timer daemon has waited
 can be edited

these files can be written into or edit but cannot be read (fifo files) :
 input_monitor_wake			write this file to wake timer daemon for user 
 backlight_fifo				write this file to control backlight

others are just temporary files for program itself, please not read or edit them
```

Updates:

0.4:

  introduce nodepm-user program
  
  (with nodepm-user) making it possible to be faster to wakeup from idle under X11
