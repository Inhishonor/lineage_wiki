{% include alerts/warning.html content="These instructions are NOT for carrier branded variants." %}

{%- if device.before_install.instructions == "needs_specific_android_fw" and device.before_install.version -%}
1. Download the latest **Android {{ device.before_install.version }}** full stock ROM update from [Oxygen Updater](https://play.google.com/store/apps/details?id=com.arjanvlek.oxygenupdater)
{%- else -%}
1. Download the latest full stock ROM update from [Oxygen Updater](https://play.google.com/store/apps/details?id=com.arjanvlek.oxygenupdater)
{%- endif %}
2. Extract stock ROM using [payload-dumper-go](https://github.com/ssut/payload-dumper-go/releases/latest)
3. Boot up LineageOS recovery
4. Go to "Advanced" -> "Enter fastboot"
   {% include alerts/warning.html content="The \"Enter fastboot\" option may not be present on older LineageOS recovery builds and it cannot be substituted with \"Reboot to bootloader\"." %}
5. Execute following commands:
   ```
   {{- include.content -}}
   ```

{% include snippets/fw_update_success.md %}
