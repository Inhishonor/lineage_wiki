{% assign device = site.data.devices[page.device] %}
{% assign current_branch = device.current_branch %}


<script src="{{ site.baseurl }}/assets/js/pagination.js{{ cacheBust }}"></script>

<div id="paginated-content" markdown="1">

{% if device.migrated_to and device.migrated_to != "" %}
{% include templates/device_migrated_to.md %}
{% endif %}

{% capture install_content -%}
The provided instructions are for LineageOS {{ current_branch }}. These will only work if you follow every section and step precisely. <br/>
Do **not** continue after something fails!
{%- endcapture %}

{% include alerts/warning.html content=install_content %}

## Basic requirements

1. Read through the instructions at least once before actually following them, so as to avoid any problems due to any missed steps!
2. Make sure your computer has `adb`{% unless device.install_method == 'heimdall' or device.install_method == 'dd' or device.install_method == 'nintendo' %} and `fastboot`{% endunless %}. Setup instructions can be found [here]({{ "adb_fastboot_guide.html#installing-adb-and-fastboot" | relative_url }}).
{%- unless device.stock_is_not_android %}
3. Enable [USB debugging]({{ "adb_fastboot_guide.html#setting-up-adb" | relative_url }}) on your device.
{%- endunless %}
{%- if device.models %}
4. Make sure that your model number is one of the following (exact match required!):
{%- for model in device.models %}
* {{ model }}
{%- endfor %}
{%- endif %}
5. Boot your device with the stock OS at least once and check every functionality.
{%- if device.type == "phone" %}
{%- unless device.quirks contains "ims" %}
    {% include alerts/warning.html content="Make sure that you can send and receive SMS and place and receive calls (also via WiFi and LTE, if available), otherwise it won't work on LineageOS either! Additionally, some devices require that VoLTE/VoWiFi be utilized once on stock to provision IMS." %}
{%- endunless %}
{%- endif %}
{%- unless device.stock_is_not_android %}
6. Remove all Google accounts from your device to avoid "Factory reset protection"
{%- endunless %}
7. LineageOS is provided as-is with no warranty. While we attempt to verify [everything works](https://github.com/LineageOS/charter/blob/main/device-support-requirements.md) you are installing this at your own risk!

{%- if device.before_install %}
{% capture path %}templates/device_specific/before_install_{{ device.before_install.instructions }}.md{% endcapture %}
{% include {{ path }} %}
{%- endif %}

{% if device.required_bootloader %}
## Special requirements

{%- capture bootloader %}
Your device must be on bootloader version {% for el in device.required_bootloader %} {% if forloop.last %} `{{ el }}` {% else %} `{{ el }}` / {% endif %} {% endfor %}, otherwise the instructions found in this page will not work.
The current bootloader version can be checked by running the command `getprop ro.bootloader` in a terminal app or by running `adb -d shell getprop ro.bootloader` from a command prompt (on Windows) or terminal (on Linux or macOS) window.
{% endcapture %}
{% include alerts/warning.html content=bootloader %}
{%- endif %}

{%- if device.before_install.instructions == "needs_specific_android_fw" %}
## Checking the correct firmware

Installation on your device requires a specific [firmware]({{ "glossary/#firmware" | absolute_url }}){: .glossary} version to be installed before you continue.

{%- if device.before_install.version %}
- Firmware refers to a device-specific set of images that are included in, and updated by the stock OS
- LineageOS builds for this device require an **Android {{ device.before_install.version }}** version of the [stock OS]({{ "glossary/#stock-rom" | absolute_url }}){: .glossary} to be installed prior to following the installation guide
- Please ensure that you are checking the **Android** version, and not the vendor OS version
{%- endif %}
- Being on another custom ROM, including unofficial builds of the same version of LineageOS, does not ensure that this requirement has been fulfilled
- Please re-read this section as many times as necessary to fully understand the requirements

{%- capture content %}
{%- if device.before_install.version %}
If you are unsure what firmware version you are currently on, we strongly recommend returning to the corresponding stock OS before following the installation guide!
{%- elsif device.before_install_device_variants %}
If you are unsure what firmware version you are currently on, we strongly recommend following [this guide]({{ device | device_link: "/fw_update" | relative_url }}), just in case!
{%- endif %}
{%- endcapture %}
{% include alerts/note.html content=content %}

Failing to install the correct firmware version prior to installation may result in failure to install LineageOS, unexpected crashes post-installation,
or permanent damage to your device!
{%- endif %}

{%- if device.install_method %}
{% capture recovery_install_method %}templates/recovery_install_{{ device.install_method }}.md{% endcapture %}
{% include {{ recovery_install_method }} %}
{%- else %}
## Unlocking the bootloader / Installing a custom recovery

There are no recovery installation instructions for this discontinued device.
{%- endif %}

{%- if device.before_lineage_install %}
{% capture path %}templates/device_specific/before_lineage_install_{{ device.before_lineage_install }}.md{% endcapture %}
{% include {{ path }} %}
{%- endif %}

## Installing LineageOS from recovery

{%- if device.architecture.userspace -%}
{% assign userspace_architecture = device.architecture.userspace %}
{%- else -%}
{% assign userspace_architecture = device.architecture %}
{%- endif -%}

{%- if device.maintainers != empty %}
1. Download the [LineageOS zip file](https://download.lineageos.org/devices/{{ device.codename }}) that you would like to install or [build]({{ device | device_link: "/build" | relative_url }}) the package yourself.
   {% include alerts/note.html content="This page may contain additional files that are not mentioned in this guide. Please ignore these as they are unnecessary to the installation process." %}
{%- else %}
1. [Build]({{ device | device_link: "/build" | relative_url }}) a LineageOS installation package.
{%- endif %}
2. If you are not in recovery, reboot into recovery:
    * {{ device.recovery_boot }}
    {% if device.vendor == "LG" %}
        {% include templates/recovery_boot_lge.md %}
    {% endif %}
{%- if device.uses_twrp %}
3. Now tap **Wipe**.
4. Now tap **Format Data** and continue with the formatting process. This will remove encryption and delete all files stored in the internal storage.
{%- if device.is_ab_device %}
{%- else %}
5. Return to the previous menu and tap **Advanced Wipe**, then select the *Cache* and *System* partitions and then **Swipe to Wipe**.
{%- endif %}
6. Sideload the LineageOS `.zip` package:
    * On the device, select "Advanced", "ADB Sideload", then swipe to begin sideload.
    * On the host machine, sideload the package using:
    ```
    adb -d sideload filename.zip
    ```
        {% include alerts/specific/tip_adb_flash_success.html %}
{%- else %}
3. Now tap **Factory Reset**, then **Format data / factory reset** and continue with the formatting process. This will remove encryption and delete all files stored in the internal storage, as well as format your cache partition (if you have one).
4. Return to the main menu.
{%- if device.has_no_usb %}
5. Copy the the LineageOS `.zip` package and any addons you wish to install to your device's removable SD card, then insert it in your device and install it, but **do not reboot to system** before you read/followed the rest of the instructions!:
    * On the device, select "Apply Update", then "Apply from EXT_SDCARD", and select the LineageOS zip file to begin installation.
    {% include alerts/warning.html content="EXT_SDCARD will reflect the name of your SD card's filesystem name." %}
{%- else %}
5. Sideload the LineageOS `.zip` package but **do not reboot to system** before you read/followed the rest of the instructions!
    * On the device, select "Apply Update", then "Apply from ADB" to begin sideload.
    * On the host machine, sideload the package using:
    ```
    adb -d sideload filename.zip
    ```
{%- endif %}
{%- if current_branch >= 20 and device.is_ab_device %}
        {% include alerts/specific/tip_ab_reboot_recovery.html %}
{%- endif %}
{%- if device.has_no_usb %}
{%- else %}
        {% include alerts/specific/tip_adb_flash_success.html %}
{%- endif %}
{%- endif %}

## Installing Add-Ons

{% include alerts/note.html content="If you don't want to install any add-on (such as Google Apps), you can skip this whole section!" %}

{%- capture gapps_note %}
If you want to install Google Apps add-on package (use the `{{ userspace_architecture }}` architecture), you can download it from [here]({{ "gapps" | relative_url }}).
This add-on needs to be installed **before** booting into LineageOS for the first time!
{%- endcapture %}
{% include alerts/warning.html content=gapps_note %}

{%- if device.is_ab_device and device.uses_twrp %}
1. Even though you are already in recovery, run:
    ```
    adb -d reboot sideload
    ```
    Then run for each of those packages in sequence:
    ```
    adb -d sideload /path/to/zip
    ```
{%- elsif device.is_ab_device and current_branch < 20 %}
1. Even though you are already in recovery, click `Advanced`, then `Reboot to Recovery`
2. When your device reboots, click `Apply Update`, then `Apply from ADB`, then run for each of those packages in sequence:
    ```
    adb -d sideload /path/to/zip
    ```
{%- elsif device.uses_twrp != true %}
{%- if device.has_no_usb %}
1. Click `Apply Update`, then `Apply Update from EXT_SDCARD`, then select the add-on zip file to begin installation, and repeat for all desired packages in sequence.
    {% include alerts/warning.html content="EXT_SDCARD will reflect the name of your SD card's filesystem name." %}
{%- else %}
1. Click `Apply Update`, then `Apply from ADB`, then run for each of those packages in sequence:
    ```
    adb -d sideload /path/to/zip
    ```
{%- endif %}
{%- else %}
1. Repeat the sideload steps above for all desired packages in sequence.
{%- endif %}
{%- if device.is_ab_device or device.uses_twrp != true %}
<br/>When presented with a screen that says `Signature verification failed`, click `Yes`. It is expected as add-ons aren't signed with LineageOS's official key!
{%- endif %}

## All set!

Once you have installed everything successfully, you can now reboot your device into the OS for the first time!

{%- if device.uses_twrp and device.is_ab_device != true %}
* Run:
    ```
    adb -d reboot
    ```
{%- else %}
* Click the back arrow in the top left of the screen, then "Reboot system now".
{%- endif %}

{%- capture first_boot %}
The first boot usually takes no longer than 15 minutes, depending on the device.
If it takes longer, you may have missed a step, otherwise feel free to [get assistance](#get-assistance).
{%- endcapture %}
{% include alerts/note.html content=first_boot %}

{% if device.custom_recovery_link or device.uses_twrp %}
{% include alerts/specific/warning_recovery_app.html %}
{% endif %}

If you are wondering about the next steps, continue, otherwise, enjoy your new LineageOS installation.

## What next?

- If you are considering relocking your bootloader, see our [FAQ]({{ "faq#canshould-i-relock-my-bootloader" | absolute_url }}) about it.
- If your device is displaying a warning on every boot - there is nothing you can do about it but ignoring.
- Please do not use any methods to deinstall system apps. We are already shipping only a minimal set of apps and doing so can break your system. You are warned!
- We don't support any methods for rooting and thus won't provide support with it on any of our platforms
- LineageOS is Android. It supports apps the same way as any other Android device, but apps might choose to exclude you from using them. Please go back to your [device's quirks section]({{ device | device_link | append: "#known-quirks" | relative_url }}) and read about "Device integrity".

</div>

<button class="btn btn-primary" style="display: none" onClick="previousPage()" id="previous-page">Previous step</button>
<button class="btn btn-primary" style="display: none" onClick="nextPage()" id="next-page">Next step</button>

## Get assistance

After you've double checked that you followed the steps precisely, didn't skip any and still have questions or got stuck, feel free to ask on [our subreddit](https://reddit.com/r/LineageOS), on [our Discord server](https://discord.gg/gD6DMtf), or in
[#LineageOS on Libera.Chat](https://web.libera.chat/gamja/?channel=#lineageos).

