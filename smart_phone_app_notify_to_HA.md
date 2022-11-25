# How to add smart phone's application notification to HA?
1. Install HA app
2. Enable HA app'authority of other app notification 
3. Check update rate of HA app
4. Go to HA to chechk if app notifications are read by HA
5. Set a HA automation flow as folowed (only triggered during 0700~2400)
```yaml
alias: Package Notify by Kinet app
description: >-
  Get app notification then send TTS to smart soundbox.
trigger:
  - platform: template
    value_template: >
      {{ '郵件' in state_attr('sensor.bla_l29_active_notification_count',
      'android.bigText_chk.kingnet.app_0') }}
condition:
  - condition: or
    conditions:
      - condition: time
        before: "00:00:00"
        after: "07:30:00"
action:
  - service: xiaomi_miot.intelligent_speaker
    data:
      entity_id: media_player.xiao_ai_yin_xiang_pro_2_play_control
      text: >-
        主人，{{ state_attr('sensor.bla_l29_active_notification_count',
        'android.bigText_chk.kingnet.app_0').split('，')[1] }}
  - service: notify.linebot_1
    data:
      message: >-
        {{ state_attr('sensor.bla_l29_active_notification_count',
        'android.bigText_chk.kingnet.app_0').split('，')[1] }}
mode: single
```
