~~有點搞不懂在HA自動化能做到的功能為什麼要另外去NR裡面var才執行，多此一舉啊...~~  
無論是用在腳本或是事件觸發後的自動化都可以如法炮製  
- 不想保留截圖的話可以這樣做
```yaml
alias: Cam1截圖後發送(不保留歷史截圖)
sequence:
  - service: camera.snapshot
    data:
      filename: /config/www/Camera_snaps/Cam1_latest.jpg
    target:
      entity_id: camera.camera1
  - service: notify.linebot_1
    data:
      message: Cam 1 latest photo {{now().strftime("%Y-%m-%d %H:%M:%S")}}
      data:
        file: /config/www/Camera_snaps/Cam1_latest.jpg
mode: single
icon: mdi:cctv
```

- 想保留截圖的做法(**但遇到截圖時間剛好跨分時執行的話比較容易出問題；譬如截圖時間在32分59秒，但LINE傳圖時間在33分02秒，就會找不到圖**)
```yaml
alias: Cam1截圖後發送(保留歷史截圖)
sequence:
  - service: camera.snapshot
    data:
      filename: /config/www/Camera_snaps/Cam1_{{now().strftime("%Y-%m-%d")}}/{{now().strftime("%Y-%m-%d_%H-%M")}}.jpg
    target:
      entity_id: camera.camera1
  - service: notify.linebot_1
    data:
      message: Cam 1 latest photo {{now().strftime("%Y-%m-%d %H:%M:%S")}}
      data:
        file: /config/www/Camera_snaps/Cam1_{{now().strftime("%Y-%m-%d")}}/{{now().strftime("%Y-%m-%d_%H-%M")}}.jpg
mode: single
icon: mdi:cctv
```
