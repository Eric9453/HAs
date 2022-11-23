# 瓦斯表讀數接入HA (OCR API)
![image](https://user-images.githubusercontent.com/21095518/203475044-4935ffe9-54a1-447b-9e92-bdf86bfa8e41.png)

1. 申請[OCR API](http://ocr.space/OCRAPI)，免費版即可，一天有500次的使用額度，十分足夠 (credited by @maxrdnew)
2. 架設攝影機，構圖務必方正，截圖時光線充足、無反光
3. 將以下內容寫入configuration.yaml中(或是package資料夾中另外新建gas_ocr.yaml也可以，依照個人管理方式進行)，輪巡單位(scan_interval)為秒，依照個人喜好更新頻率調整  
如遇到回傳數值不如預期，請調整鏡頭到最佳辨識畫面後，再調整value_template內的參數
```yaml
sensor:
  - platform: command_line  
    command: 'curl -H "apikey:O*C*R*的*A*P*I" --form "file=@/media/gas.jpg(*截*圖*位*置*)" --form "OCREngine=2" https://api.ocr.space/Parse/Image'
    name: gas_ocr
    unit_of_measurement: 'm³'
    json_attributes:
      - ParsedResults
    value_template: "{{ (value_json['ParsedResults'][0]['ParsedText'].split('\n')[1:2])|replace('O','0')|replace(']','')|replace('[','')|replace(\"'\",'') }}"
    scan_interval: 86400
    command_timeout: 30
```
4. 寫入yaml後重新啟動HA，等待數值讀入
5. 如有疑問可以先參考[HA官方說明](https://www.home-assistant.io/integrations/sensor.command_line/)
6. 設置自動化截圖，看頻率要多久，我是一天一次，檔名都用gas.jpg，存檔路徑需與第3點的yaml一致
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
      filename: /media/gas.jpg
    target:
      entity_id: camera.camera103
mode: single
```
7. value_template參數簡述：  
- value_json['ParsedResults'][0]['ParsedText']→解析json至文字辨識結果的層級  
- .split('\n')[1:2]→利用換行符號分割文段，並取出瓦斯度數的部分  
- |replace→開始瘋狂取代你不想要的內容  

---
題外話：HA中內建的7段數字辨識效果不是很好...還是走OCR PAI吧
