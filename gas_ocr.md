# 瓦斯表讀數接入HA (OCR API)
![image](https://user-images.githubusercontent.com/21095518/203713482-531260b9-25c3-4dc5-a241-a46118c257e6.png)

1. 申請[OCR API](http://ocr.space/OCRAPI)，免費版即可，一天有500次的使用額度，十分足夠 (credited by @maxrdnew)
2. 架設攝影機，構圖務必方正，截圖時光線充足、無反光
3. 設置自動化截圖，看頻率要多久，我是一天一次，檔名都用gas.jpg，存檔路徑需與第4點的yaml一致
```yaml
alias: Gas Ocr Snapshot@0400
description: 凌晨4點瓦斯表截圖做OCR
trigger:
  - platform: time
    at: "04:00:00"
condition: []
action:
  - service: camera.snapshot
    data:
      filename: /media/gas.jpg(改成你的存檔位置)
    target:
      entity_id: camera.camera103(改成你的攝影機)
mode: single
```
4. 將以下內容寫入configuration.yaml中(或是package資料夾中另外新建gas_ocr.yaml也可以，依照個人管理方式進行)，存檔路徑需與第3點的yaml一致，輪巡單位(scan_interval)為秒，依照個人喜好更新頻率調整  
如遇到回傳數值不如預期，請調整鏡頭到最佳辨識畫面後，再調整value_template內的參數
```yaml
sensor:
  - platform: command_line  
    command: 'curl -H "apikey:O*C*R*的*A*P*I" --form "file=@/media/gas.jpg(*截*圖*位*置*)" --form "OCREngine=2" https://api.ocr.space/Parse/Image'
    name: gas_ocr
    unit_of_measurement: 'm³'
    json_attributes:
      - ParsedResults
    value_template: "{{ (value_json['ParsedResults'][0]['ParsedText'].split('\n1000')[0].split('\n')[-1])|replace('O','0')|replace(' ','') }}"
    scan_interval: 86400
    command_timeout: 30
```
5. 寫入yaml後重新啟動HA，等待數值讀入
6. 如有疑問可以先參考[HA官方說明](https://www.home-assistant.io/integrations/sensor.command_line/)
7. value_template參數簡述：  
- value_json['ParsedResults'][0]['ParsedText']→解析json至文字辨識結果的層級  
- .split('\n')[1:2]→利用換行符號分割文段，並取出瓦斯度數的部分  
- |replace→開始瘋狂取代你不想要的內容  

8. 如果要一氣呵成的完成定時自動截圖+OCR+符合數據後動作，可使用下列自動化~~；要用在車牌辨識後開門也是可以的，嘻嘻~~
```yaml
alias: Gas Ocr Snapshot@1200
description: 中午12點瓦斯表截圖做OCR後符合數值就切換燈具
trigger:
  - platform: time
    at: "12:00:00"
condition: []
action:
  - service: camera.snapshot
    data:
      filename: /media/gas.jpg
    target:
      entity_id: camera.camera103
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
    enabled: true
  - service: command_line.reload
    data: {}
  - if:
      - condition: template
        value_template: "{{ '0003' in states.sensor.gas_ocr.state }}"
    then:
      - type: toggle
        device_id: b108ec39697d001f0300b02e75583503
        entity_id: switch.sonoff_10011b726c
        domain: switch
mode: single
```

![image](https://user-images.githubusercontent.com/21095518/203475044-4935ffe9-54a1-447b-9e92-bdf86bfa8e41.png)
---
題外話：HA中內建的7段數字辨識效果不是很好...還是走OCR PAI吧
---
## 2023-02-16 更新
給吳大的code
```yaml
# 吳大的瓦斯讀表-屬性
sensor:
  - platform: command_line  #https://www.home-assistant.io/integrations/sensor.command_line/ #輪巡間隔單位為秒
    command: 'curl -H "apikey:###API  API  API###" --form "file=@/config/www/gas.jpg" --form "OCREngine=5" https://api.ocr.space/Parse/Image'
    name: gas_ocr
    unit_of_measurement: 'm³'
    json_attributes:
      - ParsedResults
    value_template: "{{ (value_json['ParsedResults'][0]['ParsedText'].split('\n1000'))[0]|replace('\n','')|replace(' ','') }}"
    scan_interval: 86400
    command_timeout: 30
```
利用HA內的功耗表達成每期度數計算(不用寫資料庫計算)
![SC_2023-02-15_17-24-09_001](https://user-images.githubusercontent.com/21095518/219263602-0091fab2-8b47-4e59-a0a6-a71031b8b5af.png)

搭配Line notify後可以這麼做
![gas](https://user-images.githubusercontent.com/21095518/219264490-7ce8dc37-df86-4c13-9a7f-003da874c59c.jpg)


在小蟻內回call HA內的腳本，因為無法用HA走telnet控制小蟻開關紅外線照明，改為在小蟻內定時控制紅外後回call HA腳本
```yaml
alias: Gas meter snapt and OCR
sequence:
  - service: camera.snapshot
    data:
      filename: /config/www/gas.jpg
    target:
      entity_id: camera.camera3
  - delay:
      hours: 0
      minutes: 0
      seconds: 3
      milliseconds: 0
  - service: command_line.reload
    data: {}
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
  - service: notify.linebot_1
    data:
      message: >-
        瓦斯表讀數現為{{ states('sensor.gas_ocr')|replace('00','') }}m³(度)，本期已使用{{
        states('sensor.gas_consumption')}}度
      data:
        file: /config/www/gas.jpg
  - service: tts.azure_cognitive_speech_say
    data:
      entity_id: media_player.hei_la_ba
      message: >-
        瓦斯表讀數現為{{ states('sensor.gas_ocr')|replace('00','') }}度，本期已使用{{
        states('sensor.gas_consumption')}}度
mode: single
icon: mdi:gas-burner
```
