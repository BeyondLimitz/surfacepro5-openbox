# surfacepro5-openbox
a customized debian 12 configuration for SurfacePro 5 Openbox
# Debian Openbox Configuration

Building a custom graphical interface over a Debian 12 headless base is an excellent way to reclaim hardware overhead on a 8GB Microsoft Surface Pro.
For your specific hardware target, Openbox is the highly recommended choice over Fluxbox. While both are incredibly light (~2–8MB idle overhead), Openbox holds two decisive advantages for a Microsoft Surface Pro installation: 

1. Pixel Density & Scaling: The Microsoft Surface Pro relies on high-DPI (HiDPI) screens. Openbox UI sizing, fonts, padding, and margins are parsed cleanly using simple multipliers or standard theme tools (obconf), making scaling much easier to manage. Fluxbox themes utilize static XPixMaps (.xpm) for parts of the window decorations, which can break or look miniscule on a modern HiDPI display without significant pixel-by-pixel editing.

2. Modular Architecture: Since Fluxbox forces its own panel, tray, and tabbing engine on you, it resists customization. Because you want to build a bespoke desktop environment piece-by-piece, Openbox gives you a perfectly blank canvas with zero forced tooling. 

## The Microsoft Surface Blueprint

To build a streamlined, ultra-lightweight environment around Openbox on Debian 12, deploy these carefully picked, modern building blocks:
• Display Server: xorg + xinit (Skipping a heavy display manager like GDM/LightDM entirely lets you boot straight to CLI and run startx to spin up your desktop).
• Panel / Taskbar: tint2 (Highly configurable, parses system icons well, and lets you toggle text layout sizes easily for high resolution displays).
• File Manager: pcmanfm (Provides optional desktop background/icon management if initialized with pcmanfm --desktop, keeping memory footprint negligible).
• Application Launcher: rofi or dmenu (Dynamic, lightning-fast search menus that completely replace sluggish traditional desktop application listings).
• Composite Manager: picom (Vital for smooth screen rendering, avoiding video tearing, and enabling subtle window shadows).
• Notification Daemon: dunst (Extremely fast, low-footprint daemon driven by a simple text file).

## Step-by-Step Base Installation Workflow

Once your headless Debian 12 environment is up, execute this workflow to build your desktop environment layer:

### 1. Install Core GUI Packages

```bash
sudo apt update && sudo apt install --no-install-recommends \
    xorg xinit openbox obconf tint2 pcmanfm rofi dunst picom \
    feh lxappearance xterm
```
**Use code with caution.**
(The --no-install-recommends flag is critical to prevent Debian from pulling in massive recommended dependency chains).

### 2. Populate Local Configurations

```bash
# Create target configuration paths
mkdir -p ~/.config/openbox
mkdir -p ~/.config/tint2

# Copy default configurations into your user profile
cp /etc/xdg/openbox/rc.xml ~/.config/openbox/
cp /etc/xdg/openbox/menu.xml ~/.config/openbox/
cp /etc/xdg/openbox/autostart ~/.config/openbox/
```
Use code with caution.

### 3. Configure Your Autostart Script

Open ~/.config/openbox/autostart in your preferred text editor and append your component daemons:
```bash
# Set screen background wallpaper
feh --bg-scale /path/to/your/wallpaper.jpg &

# Start notification tool
dunst &

# Start the compositor for anti-tearing
picom --vsync &

# Launch your custom panel
tint2 &
```
Use code with caution.

### 4. Configure Direct Boot to X11

Create or open ~/.xinitrc in your home directory and insert exactly one line:
```bash
exec openbox-session
```
Use code with caution.
Now, logging into your headless terminal and execution of startx will immediately throw you into a lightning-fast, tailored workstation idling below 350MB of RAM total system usage.

**Crucial Considerations for Surface Pro Deployment**

• Linux Surface Kernel: The stock Debian kernel may lack optimization for specialized Surface hardware. If you run into issues with the touchscreen, battery status indicators, or hardware buttons, integrate the custom Linux Surface Kernel Project repository to instantly ensure complete hardware compatibility.
• Touch Input & On-Screen Keyboards: Since you're targeting touch hardware, keep in mind that Openbox doesn't inherently listen to touch gestures out of the box. If you expect tablet-mode functionality, install onboard for an on-screen keyboard and map a rofi keybinding to toggle it easily.

Here is how to configure precise HiDPI scaling and implement your keyboard shortcuts for Rofi, Volume, and Brightness inside your Openbox setup on the Microsoft Surface Pro.

## 1. Establishing the HiDPI Scaling Foundations

The Microsoft Surface Pro relies on a high-density screen. Since Openbox does not scale dynamically like a heavy DE, you must feed the correct pixel scaling factors directly to X11, GTK applications, and Openbox window elements. 
A scale factor of 200% (2x) or a custom DPI value around 144 to 192 generally offers the cleanest layout on Surface displays. 

## A. Configure Global X11 Font Scaling

Create or open the user configuration file ~/.Xresources: 
```bash
nano ~/.Xresources
```
Use code with caution.
Add the following parameters to scale system font layouts globally: 
```ini
Xft.dpi: 192
Xft.autohint: 0
Xft.lcdfilter: lcddefault
Xft.hintstyle: hintslight
Xft.hinting: 1
Xft.antialias: 1
Xft.rgba: rgb
Xcursor.size: 32
```
Use code with caution.
To load this file on startup, add xrdb -merge ~/.Xresources to the top of your ~/.config/openbox/autostart script. 

## B. Force Toolkits to Scale (GTK & Qt)

Open your system environment profiles or add these variables to your ~/.config/openbox/environment file so that your standard applications render with double pixel sizing: 
```bash
export GDK_SCALE=2
export GDK_DPI_SCALE=0.5
export QT_SCALE_FACTOR=2
```
Use code with caution.
(Setting GDK_DPI_SCALE=0.5 ensures that GTK text does not accidentally double-scale alongside the rest of the application layout). 

# Hardware Management

## 2. Mapping Rofi, Volume, and Brightness Shortcuts

Openbox manages all system keybindings natively via the <keyboard> block within ~/.config/openbox/rc.xml. 

To install the lightweight tools required to process the multimedia triggers on Debian 12:
```bash
sudo apt install --no-install-recommends alsa-utils brightnessctl
```
Use code with caution.
Open your keybinding configuration file:
```bash
nano ~/.config/openbox/rc.xml
```
Use code with caution.
Scroll down until you locate the <keyboard> tag, and insert the following blocks inside it: 

## A. Application Launcher Shortcut (Rofi)

This maps Super + Space (the Windows/Surface key + spacebar) to open your applications launcher. By passing the -dpi 192 flag, Rofi will render clearly on your high-resolution screen. 
```xml
  <!-- Launch Rofi Application Menu -->
  <keybind key="W-space">
    <action name="Execute">
      <command>rofi -show drun -dpi 192</command>
    </action>
  </keybind>
```
Use code with caution.

## B. Native Audio Volume Controls

These intercept your Surface Pro's audio keys to modify sound parameters through ALSA: 
Add keybindings for volume up, volume down, and mute toggles using amixer -q set Master commands within your <keyboard> configurations. 

## C. Hardware Screen Brightness Controls

These binds allow the dedicated brightness hardware keys to adjust your screen backlight illumination via brightnessctl. 

## 3. Activating Your New Settings

To apply your new configurations without restarting your system, force Openbox to reload its configuration profile directly from your live session: 
```bash
openbox --reconfigure
```
Use code with caution.

# HiDPI Configuration

Here is how to solve the next two hurdles for your Surface Pro 5 deployment.
Because Openbox calculates window borders using fixed pixels rather than scalable vectors, default themes look paper-thin on the Surface Pro 5's 2736×1824 display. Additionally, since Openbox lacks a built-in touch input layer for X11, you must pair it with dedicated background daemons to handle tapping, scrolling, and multi-finger gestures cleanly.

1. Choosing and Installing a Pre-Scaled HiDPI Openbox Theme

While you can try to find community themes on platforms like GitHub, many are unmaintained. Instead, you can use modern theme engines that ship with dedicated HiDPI/Retina padding options natively.
The cleanest approach is to deploy a modern, flat theme engine that generates high-DPI scaling directly.
• Option A: The Celestial Theme (Highly Recommended)
The Celestial GTK/Openbox Theme is a modern, flat design theme that explicitly bundles a standalone HiDPI and XHiDPI Openbox layout profile, ensuring thick, grab-friendly titlebars and window borders on high-resolution displays.
• Option B: BunsenLabs Beam HiDPI
The Bunsen-Beam HiDPI Theme is specifically targeted at minimalist Openbox setups running on high-density screens, multiplying window control elements precisely to match 2x pixel layouts.

## Installing and Activating a Theme

To easily apply your theme across both your windows and your GTK apps, pull down your chosen theme directory into your local profile and manage them using standard configuration utilities: 
```bash
# Install the missing Openbox engine plugins for your GUI utility
sudo apt install --no-install-recommends lxappearance-obconf

# Create your local themes directory if it doesn't exist
mkdir -p ~/.themes

# Clone or drop your chosen HiDPI theme folder into ~/.themes
# (Example using Celestial theme repository setup)
git clone https://github.com/zquestz/celestial-gtk-theme /tmp/celestial
cp -r /tmp/celestial/themes/* ~/.themes/

# Launch the visual customization wrapper
lxappearance
```

Inside the lxappearance tool, navigate to the Window Border tab (which embeds the obconf panel). Choose the HiDPI variant of your theme. This instantly scales title bars, minimizing window alignment errors when tapping with fingers. 

2. Configuring Touchscreen Tapping and Multi-Finger Gestures

Since you are operating a headless Debian 12 base under an X11-driven Openbox environment, touch registers as basic mouse movements out of the box. To bring smooth kinetic scrolling and multi-touch gestures to life without loading a bloated Desktop Environment, use Touchégg. Touchégg operates as a silent background daemon that translates multi-touch inputs into standard X11 keystrokes or actions. 
A. Install Touchégg and Input Utilities

```bash
sudo apt update && sudo apt install --no-install-recommends touchegg xdotool xinput
```
Use code with caution.

B. Handle Click Tapping (The Libinput Layer)

X11 relies on the libinput driver to parse the Surface Pro 5 touchscreen. To make sure single taps and double taps behave correctly, verify that tapping properties are flagged as active.
Add this instruction directly to your ~/.config/openbox/autostart script to force libinput to recognize natural touchscreen tapping:
```bash
# Find your touchscreen ID using 'xinput list' if needed, though libinput usually applies this globally
xinput set-prop "Intel Precise Touch Screen" "libinput Tapping Enabled" 1 2>/dev/null || true
```
Use code with caution.

C. Setting up Multi-Finger Gestures

Touchégg relies on an XML configuration layout to translate finger gestures (swipes, pinches) into active desktop behaviors. Create a customized layout profile: 
```bash
mkdir -p ~/.config/touchegg
nano ~/.config/touchegg/touchegg.conf
```
Use code with caution.
Paste the following baseline script to optimize your layout for Openbox window execution:
```xml
<touchégg>
  <settings>
    <property name="animation_delay">150</property>
    <property name="action_delay">200</property>
  </settings>
  
  <application name="All">
    <!-- Pinch with 2 fingers to zoom (Simulates Ctrl+ and Ctrl-) -->
    <gesture type="PINCH" fingers="2" direction="IN">
      <action type="SEND_KEYS">
        <modifiers>Control_L</modifiers>
        <keys>minus</keys>
        <repeat>true</repeat>
      </action>
    </gesture>
    
    <gesture type="PINCH" fingers="2" direction="OUT">
      <action type="SEND_KEYS">
        <modifiers>Control_L</modifiers>
        <keys>plus</keys>
        <repeat>true</repeat>
      </action>
    </gesture>

    <!-- Swipe up with 3 fingers to trigger Rofi application launcher -->
    <gesture type="SWIPE" fingers="3" direction="UP">
      <action type="RUN_COMMAND">
        <command>rofi -show drun -dpi 192</command>
        <repeat>false</repeat>
      </action>
    </gesture>

    <!-- Swipe left/right with 3 fingers to switch Openbox virtual desktops -->
    <gesture type="SWIPE" fingers="3" direction="LEFT">
      <action type="SEND_KEYS">
        <modifiers>Control_L</modifiers>
        <keys>Left</keys>
        <repeat>false</repeat>
      </action>
    </gesture>

    <gesture type="SWIPE" fingers="3" direction="RIGHT">
      <action type="SEND_KEYS">
        <modifiers>Control_L</modifiers>
        <keys>Right</keys>
        <repeat>false</repeat>
      </action>
    </gesture>
  </application>
</touchégg>
```
Use code with caution.

D. Start the Daemon on Boot

To make sure gestures run seamlessly every time you execute startx, append the background process initialization call to the bottom of your Openbox initialization file:
```bash
nano ~/.config/openbox/autostart
```
Use code with caution.
Add the execution call right above your panel options:
```bash
# Initialize gesture processing daemon
touchegg &
```
Use code with caution.

# TLP Power Optimization

To optimize battery life on your Microsoft Surface Pro 5 without introducing the overhead of a heavy desktop environment, you should focus on command-line utilities that run silently in the background.
For a custom Debian 12 setup, you have two primary optimization paths. You must choose only one, as running them together will cause severe conflicts.

## Path 1: auto-cpufreq (Highly Recommended for Surface Hardware)

While TLP is the traditional choice for Linux laptops, community consensus among Surface Linux users on Reddit indicates that auto-cpufreq yields much better battery savings on Microsoft Surface devices. 
Instead of statically changing system parameters, it actively monitors your battery state, CPU usage, and temperatures. It dynamically throttles or boosts your CPU frequencies depending on whether you are actively compiling code or sitting idle reading a document. 

1. Install Dependencies & Build auto-cpufreq

Since you are on a lightweight headless Debian base, installing from source or via the official installer script ensures no bloated packages are added.
```bash
# Install required build tools and python utilities
sudo apt install --no-install-recommends git python3-pip python3-setuptools dmidecode

# Clone and run the official installer
git clone https://github.com/AdnanHodzic/auto-cpufreq.git /tmp/auto-cpufreq
cd /tmp/auto-cpufreq && sudo ./auto-cpufreq-installer
```
Use code with caution.
Choose the option to install the daemon service during the prompt so it launches automatically on boot. 

2. Monitor Live Stats

To ensure your Surface Pro 5's Intel CPU is scaling its governors correctly down to power-saving states when unplugged, run: 
```bash
sudo auto-cpufreq --stats
```
Use code with caution.

## Path 2: TLP (The Traditional, Granular Alternative)

If you prefer structural, rule-based power management (e.g., forcing absolute power rules for your Wi-Fi card, audio hardware, and USB ports), use TLP. 

1. Install TLP on Debian

```bash
sudo apt install --no-install-recommends tlp tlp-rdw
```
Use code with caution.

2. Configure for High Aggression on Battery

Open the master configuration file to enforce aggressive power savings when unplugged: 
```bash
sudo nano /etc/tlp.conf
```
Use code with caution.
Uncomment and adjust the following parameters to ensure your Surface maximizes its idle state:
```ini
# Force powersave governor on battery, allow performance on AC
CPU_SCALING_GOVERNOR_ON_AC=performance
CPU_SCALING_GOVERNOR_ON_BAT=powersave

# Set Intel energy performance preference
CPU_ENERGY_PERF_POLICY_ON_AC=performance
CPU_ENERGY_PERF_POLICY_ON_BAT=power

# Enable aggressive Wi-Fi and Audio power-saving modes
WIFI_PWR_ON_BAT=on
SOUND_POWER_SAVE_ON_BAT=1
```
Use code with caution.
Start and enable the systemd service:
```bash
sudo tlp start
```
Use code with caution.

### Essential Utility: Intel PowerTOP Diagnosis

Regardless of whether you choose auto-cpufreq or TLP, you should use Intel's native PowerTOP to diagnose what background tasks are draining your battery. 
```bash
sudo apt install --no-install-recommends powertop
```
Use code with caution.

### Perform a Baseline Calibration

To train the kernel on your Surface Pro 5's specific screen power draw and battery discharge behavior, run a calibration: 
```bash
sudo powertop --calibrate
```
Use code with caution.
(Your screen will flicker and change brightness states during this test; let it run uninterrupted until completion.) 

### Apply Automated Tunables

Launch sudo powertop and navigate to the Tunables tab using the Tab key. You will see a list of hardware modules marked as "Bad" (consuming too much power) or "Good". To force auto-optimization across all flags on startup, you can add this line to your ~/.config/openbox/autostart script: 
```bash
sudo powertop --auto-tune &
```


# Tint2 Toolbar config

To keep your custom Debian 12 desktop ultra-lean, you can configure both tracking features directly inside your Tint2 panel.
The clean approach is to use Tint2's native battery subsystem paired with a lightweight Executor (a feature that prints the stdout of any rapid shell script directly onto the panel).

## 1. Building the Dynamic CPU Governor Script

To get the active CPU power state, we need a lightweight script that queries the kernel file path or reads auto-cpufreq. Create a script inside your home directory:
```bash
mkdir -p ~/.config/tint2/scripts
nano ~/.config/tint2/scripts/gov_status.sh
```
Paste the following shell script. It checks if auto-cpufreq or standard cpufreq is running and outputs a compact, glanceable tag:
```bash
#!/bin/bash

# Fetch governor from the first CPU core
if [ -f /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor ]; then
    GOV=$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor)
else
    GOV="unknown"
fi

# Abbreviate names to save panel space
case "$GOV" in
    "powersave")   echo "🔋 POW" ;;
    "performance") echo "⚡ PERF" ;;
    "schedutil")   echo "⚙️ SCHED" ;;
    *)             echo "💻 $GOV" ;;
esac
Use code with caution.
Save and exit (Ctrl+O, Ctrl+X), then make the script executable:
bash
chmod +x ~/.config/tint2/scripts/gov_status.sh
```

## 2. Injecting the Elements into Your Tint2 Config

Open your main Tint2 configuration file. If you copied the default template earlier, it will be located at ~/.config/tint2/tint2rc.
```bash
nano ~/.config/tint2/tint2rc
```

### Step A: Enable the Items in the Panel layout

Find the panel_items block near the top of the file. You must add B (for native Battery) and E1 (for Executor 1) to the sequence where you want them to appear (e.g., alongside the clock C or system tray S).
```ini
# Example layout string: Taskbar, System Tray, Battery, Executor 1, Clock
panel_items = TSCB E1
```

### Step B: Define the Native Battery Component

Scroll to the bottom of the file or find an existing # Battery block, and configure it to look crisp on your HiDPI screen:  
```ini
#-------------------------------------
# BATTERY
#-------------------------------------
battery = 1
battery_hide = never
battery_low_status = 15
battery_low_cmd = notify-send "Surface Battery Critical!"
bat1_font = Sans Bold 11
bat2_font = Sans Bold 11
battery_font_color = #FFFFFF 100
battery_padding = 6 4 6 4
battery_background_id = 0
battery_text_format = 🔋 %p%
```

### Step C: Define the CPU Governor Executor

Directly below your battery configuration, add the executor block to run your custom shell script every few seconds:
```ini
#-------------------------------------
# EXECUTOR 1 (CPU Governor)
#-------------------------------------
execp = 1
execp_command = ~/.config/tint2/scripts/gov_status.sh
execp_interval = 4
execp_has_icon = 0
execp_font = Sans Bold 11
execp_font_color = #FFFFFF 100
execp_padding = 6 4 6 4
execp_background_id = 0
```

## 3. Applying and Testing

Save the configuration file. To force Tint2 to rebuild itself instantly without breaking your active X11 desktop environment session, reload the panel daemon:
```bash
killall -SIGUSR1 tint2
```

Your taskbar will flash momentarily and reappear displaying your exact battery percentage alongside an active profile tag (🔋 POW or ⚡ PERF). When you unplug your Surface Pro 5, you will watch auto-cpufreq dynamically update the status bar string in real-time as it safely drops your system governors. 


## Tint2 config part 2

Here is a complete, production-ready tint2rc configuration optimized for your high-DPI Surface Pro 5 display. It implements a clean, semi-transparent matte-black aesthetic with clear typography, padding for touch targets, and integrates both your battery tracker and CPU governor script.

## 1. The Minimalist, Semi-Transparent Config

Open your Tint2 configuration file:
```bash
nano ~/.config/tint2/tint2rc
```
Delete its contents and replace them entirely with the following configuration:
```ini
#-------------------------------------
# TINT2 CONFIGURATION
#-------------------------------------

# BACKGROUNDS
# Background 1: Panel Background (Semi-transparent dark charcoal)
background_color = #1a1a1a 80
border_width = 0
border_color = #000000 0
background_style = none

# Background 2: Active task/button indicator
background_color_active = #ffffff 15
border_width_active = 1
border_color_active = #ffffff 30

# PANEL
panel_monitor = all
panel_position = bottom center horizontal
panel_size = 100% 42
panel_margin = 0 0
panel_padding = 10 0 10
panel_background_id = 1
wm_menu = 1
panel_dock = 0
panel_pivot_strut = 0
panel_window_name = tint2
panel_layer = top
panel_shrink = 0
autohide_height = 1

# PANEL ITEMS ORDER
# T = Taskbar, S = System Tray, B = Battery, E1 = CPU Gov, C = Clock
panel_items = TSCB E1

# TASKBAR
taskbar_mode = single_desktop
taskbar_hide_if_empty = 0
taskbar_padding = 4 0 4
taskbar_background_id = 0
taskbar_active_background_id = 2
taskbar_name = 1
taskbar_hide_inactive_tasks = 0
taskbar_hide_different_monitors = 0
taskbar_always_show_all_monitors = 1
taskbar_name_padding = 6 3
taskbar_name_background_id = 0
taskbar_name_font = Sans Bold 11
taskbar_name_font_color = #ffffff 100

# TASKS
task_text = 1
task_icon = 1
task_centered = 0
task_urgent_cmd = 
task_font = Sans 11
task_font_color = #cccccc 80
task_active_font_color = #ffffff 100
task_padding = 10 4 6
task_max_width = 180

# SYSTEM TRAY
systray = 1
systray_padding = 6 0 6
systray_background_id = 0
systray_sort = ascending
systray_icon_size = 22
systray_icon_ascent = 0

# CLOCK
time1_format = %H:%M
time1_font = Sans Bold 11
time1_font_color = #ffffff 100
time2_format = %A, %b %d
time2_font = Sans 9
time2_font_color = #aaaaaa 100
clock_padding = 8 0 8
clock_background_id = 0

# NATIVE BATTERY
battery = 1
battery_hide = never
battery_low_status = 15
battery_low_cmd = notify-send "Surface Battery Critical!"
bat1_font = Sans Bold 11
bat2_font = Sans Bold 11
battery_font_color = #ffffff 100
battery_padding = 8 0 8
battery_background_id = 0
battery_text_format = 🔋 %p%

# EXECUTOR 1: CPU GOVERNOR WIDGET
execp = 1
execp_command = ~/.config/tint2/scripts/gov_status.sh
execp_interval = 4
execp_has_icon = 0
execp_font = Sans Bold 11
execp_font_color = #ffffff 100
execp_padding = 8 0 8
execp_background_id = 0

# TOOLTIPS
tooltip_show_timeout = 0.5
tooltip_hide_timeout = 0.1
tooltip_padding = 4 4
tooltip_background_id = 1
tooltip_font = Sans 10
tooltip_font_color = #ffffff 100
```

## 2. Design Adjustments Made for Your Setup

• HiDPI Adjustments: The total height is pushed to 42px, and font sizes are set to 11pt bold. This keeps elements scale-proportionate and leaves large targets for touch inputs.
• True Transparency: Built-in composition transparency is handled via background_color = #1a1a1a 80. The 80 value indicates an 80% opacity fill over an underlying desktop canvas.
• Anti-Aliased Borders: Border styles are disabled on the container frame to stop pixel jaggedness on high-density displays.

## 3. Load the Changes Instantly

To see your new translucent panel system take effect over your current X11 workspace layer, reload the tint2 runtime process:
```bash
killall -SIGUSR1 tint2
```

(Note: If the background turns completely black instead of semi-transparent, ensure that picom is actively running in the background, as X11 requires a compositor to render channel alpha values).


# Configuring Desktop/Workspace and Screen Rotation


Here is the second phase of optimizing your Microsoft Surface Pro 5 desktop environment.
Because Openbox does not possess built-in virtual workspace management tools or touch-rotation tracking layers, you must implement them via Openbox’s native XML engine and a custom iio-sensor-proxy pipeline.

## Part 1: Configuring Seamless Openbox Workspaces

To cycle through workspaces fluidly without heavy desktop animations, configure dedicated key combinations and trackpad shortcuts. Open your configuration profile:
```bash
nano ~/.config/openbox/rc.xml
```

### A. Set the Total Number of Workspaces

Search for the <desktops> tag block and adjust the number of workspaces to your preferred layout (e.g., 4):
```xml
  <desktops>
    <number>4</number>
    <firstdesk>1</firstdesk>
    <names>
      <name>1: Web</name>
      <name>2: Dev</name>
      <name>3: Sys</name>
      <name>4: Chat</name>
    </names>
    <popupTime>800</popupTime>
  </desktops>
```

### B. Map Workspace Navigation Hotkeys

Scroll down into your <keyboard> section and map Super + [1-4] to jump directly to specific workspaces, and Ctrl + Alt + Left/Right to cycle through them:
```xml
  <!-- Direct Jump to Workspace (Super + Number) -->
  <keybind key="W-1"><action name="GoToDesktop"><to>1</to></action></keybind>
  <keybind key="W-2"><action name="GoToDesktop"><to>2</to></action></keybind>
  <keybind key="W-3"><action name="GoToDesktop"><to>3</to></action></keybind>
  <keybind key="W-4"><action name="GoToDesktop"><to>4</to></action></keybind>

  <!-- Cycle Workspaces (Ctrl + Alt + Arrow Keys) -->
  <keybind key="C-A-Right"><action name="GoToDesktop"><to>next</to><wrap>yes</wrap></action></keybind>
  <keybind key="C-A-Left"><action name="GoToDesktop"><to>previous</to><wrap>yes</wrap></action></keybind>
```

### C. Enable Desktop Scrolling

To switch virtual workspaces by sliding your finger down the edge of your touch panel or scrolling anywhere on an open desktop patch, insert this into your <context name="Desktop"> subsection inside the <mouse> configuration section:
```xml
    <context name="Desktop">
      <mousebind button="Up" action="Click">
        <action name="GoToDesktop"><to>previous</to><wrap>yes</wrap></action>
      </mousebind>
      <mousebind button="Down" action="Click">
        <action name="GoToDesktop"><to>next</to><wrap>yes</wrap></action>
      </mousebind>
    </context>
```
Save and refresh Openbox to register your changes immediately:
```bash
openbox --reconfigure
```

## Part 2: Accelerometer Auto-Rotation Script

When using a lightweight X11 window manager, turning the physical screen will not automatically rotate the orientation. You need to use a background daemon to capture hardware sensor logs from the Surface Pro 5 accelerometer and align xrandr (display) and xinput (touch mapping matrices) synchronously.

### A. Install Hardware Sensor Daemons

```bash
sudo apt update && sudo apt install --no-install-recommends iio-sensor-proxy inotify-tools
```

### B. Create the Target Tracking Script

Build an automated sensor monitoring script inside your user profile directory:
```bash
mkdir -p ~/.config/openbox/scripts
nano ~/.config/openbox/scripts/surface_rotate.sh
```

Paste the following script. It continuously reads the hardware state from monitor-sensor and updates the coordinate transformation matrices so your finger touch matches the screen orientation perfectly: 

```bash
#!/bin/bash
# Target variables for Surface Pro 5 hardware
DISPLAY_OUTPUT="eDP-1"
TOUCH_DEVICE="Intel Precise Touch Screen"

# Empty old logs to prevent disk clutter
> /tmp/sensor.log
monitor-sensor >> /tmp/sensor.log 2>&1 &

# Infinite monitor loop utilizing inotify watches
while inotifywait -e modify /tmp/sensor.log >/dev/null 2>&1; do
    ORIENTATION=$(tail -n 5 /tmp/sensor.log | grep 'orientation' | tail -n 1 | grep -oE '[^ ]+$')
    
    case "$ORIENTATION" in
        normal)
            xrandr --output "$DISPLAY_OUTPUT" --rotate normal
            xinput set-prop "$TOUCH_DEVICE" "Coordinate Transformation Matrix" 1 0 0 0 1 0 0 0 1
            ;;
        bottom-up)
            xrandr --output "$DISPLAY_OUTPUT" --rotate inverted
            xinput set-prop "$TOUCH_DEVICE" "Coordinate Transformation Matrix" -1 0 1 0 -1 1 0 0 1
            ;;
        right-up)
            xrandr --output "$DISPLAY_OUTPUT" --rotate right
            xinput set-prop "$TOUCH_DEVICE" "Coordinate Transformation Matrix" 0 1 0 -1 0 1 0 0 1
            ;;
        left-up)
            xrandr --output "$DISPLAY_OUTPUT" --rotate left
            xinput set-prop "$TOUCH_DEVICE" "Coordinate Transformation Matrix" 0 -1 1 1 0 0 0 0 1
            ;;
    esac
done
```
Make the script executable:
```bash
chmod +x ~/.config/openbox/scripts/surface_rotate.sh
```
### C. Hook the Engine to Start Up Automatically

Open your autostart profile file:
```bash
nano ~/.config/openbox/autostart
```
Append the background process initialization call to the bottom of the script:
```bash
# Start background accelerometer rotation monitoring pipeline
~/.config/openbox/scripts/surface_rotate.sh &
```
Now, when you detach the Surface Pro Type Cover and rotate the physical screen into portrait mode, Openbox will instantly shift your applications and redraw touch parameters seamlessly.
Would you like help setting up a lock-screen daemon (like i3lock or slock) that triggers automatically when you close the Surface Type Cover lid, or do you need help optimizing your GDK scaling variables specifically for portrait mode layouts?



**Before we get started there are a couple things that need installed:**

```
sudo apt install -y git build-essential meson dkms libssl-dev libsdl2-dev libcurl4-openssl-dev libCLI11-dev libeigen3-dev libfmt-dev libinih-dev glslc libgsl-dev libspdlog cmake make autoconf automake libcairomm-1.16-dev gcc g++
```
## Single Touch ITHC (required for multi-touch also)

The github is here: https://github.com/quo/ithc-linux

Open the terminal and cd in to Downloads.

Then clone the code with:

```
git clone https://github.com/quo/ithc-linux.git && cd into ithc-linux
```

As the instructions say run:

```
sudo make dkms-install \
sudo modprobe ithc
```

Now verify single touch works. If so you can go ahead and install.

Before touch will work after booting you need to edit grub:

```
sudo nano /etc/default/grub
```
You will see a line:
```
GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash”
```

Change to:
```
GRUB_CMDLINE_LINUX_DEFAULT=”quiet intremap=nosid splash”
```

Save the file and issue this command to set it:

```
sudo update-grub
```
Reboot and you should have single touch enabled.

## Multi-touch and Stylus IPTSD

The github is here: https://github.com/linux-surface/iptsd/

Alright, cd back into Downloads – get out of ithc if still in that directory. Grab the source with:

```
git clone https://github.com/linux-surface/iptsd/ && cd into iptsd
```

### Issue the following commands to build:

```
meson setup build \
ninja -C build \
sudo ninja -C build install
```

### Test it with:

```
sudo build/src/daemon/iptsd
```

This is from the documentation on the iptsd git page.

```
sudo systemctl daemon-reload \
sudo systemctl disable iptsd \
sudo systemctl start iptsd
```
Edit Service

```
sudo nano /lib/systemd/system/iptsd.service
```

Then saved the file with the following contents:
```
[Unit]
Description=Intel Precise Touch & Stylus Daemon
Documentation=https://github.com/linux-surface/iptsd

[Service]
Type=simple
ExecStart=/usr/local/bin/iptsd

[Install]
WantedBy=default.target
```
Once that is saved you can…

```
sudo systemctl enable iptsd && sudo reboot now
```


