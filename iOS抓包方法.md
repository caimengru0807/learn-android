1. 将iOS连接到mac上，并获取到设备的UDID。（UDID可以通过iTunes或xCode获取）。
2. 在终端上输入rvictl -s UDID 创建RVI接口。 
3. 打开wireshark 找到rvi网卡进行抓包。 
4. 抓包完毕，输入rvictl -x UDID 关闭RVI虚拟接口。
