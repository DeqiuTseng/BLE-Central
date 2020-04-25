# CCNet  BlueTooth蓝牙SDK
# Bluetooth thermometer SDK documentation

此文档用于说明 Central模式(App端) SDK使用
This document is used to indicate that Central (App) use the SDK





## iOS SDK 文档说明
## iOS SDK documentation

使用SDK前需导入libCCNetBluetooth.a 及 BabyBluetooth.h头文件到工程
Before using the SDK to import libCCNetBluetooth.a and BabyBluetooth.h header file to the project



1.引入.h文件
1.Import .h file

```objective-c
#import "BabyBluetooth.h"
```



2.初始化蓝牙库
2. Initialize Bluetooth Library

```objective-c
BabyBluetooth *baby;
baby = [BabyBluetooth shareBabyBluetooth];
```



3.设置扫描设备回调，当扫描成功后保存peripheral对象，以便下一步连接设备。
3.Set the scanning device callback. When the scanning is successful, save the peripheral object, so as to connect the device in the next step.
```objective-c
[baby setBlockOnCentralManagerDidUpdateState:^(CBCentralManager *central) {
    if (central.state == CBCentralManagerStatePoweredOn) {
				//设备打开成功，开始扫描设备    
				// Device opened successfully, start scanning device
    }
 }];


[baby setBlockOnDiscoverToPeripherals:^(CBCentralManager *central, CBPeripheral *peripheral, NSDictionary *advertisementData, NSNumber *RSSI) {
        NSLog(@"Scan device :%@",peripheral.name);
}];
```



4.查找到设备后开始进行设备连接

4. Start device connection after finding the device

```objective-c
//使用不同的channel切换委托回调,channelOnPeropheralString 为任意唯一字符串标识
//Use different channels to switch the delegate callback, and channelOnPeropheralString is any unique string identification
[baby setBlockOnConnectedAtChannel:channelOnPeropheralString block:^(CBCentralManager *central, CBPeripheral *peripheral) {
     //设备连接成功
	 //Device connected successfully
}];

[baby setBlockOnFailToConnectAtChannel:channelOnPeropheralView block:^(CBCentralManager *central, CBPeripheral *peripheral, NSError *error) {
		//设备连接失败
		// Device connection failed
}];


[baby setBlockOnDisconnectAtChannel:channelOnPeropheralView block:^(CBCentralManager *central, CBPeripheral *peripheral, NSError *error) {
     //设备断开连接
	 //Device disconnected
}];

baby.having(self.currPeripheral).and.channel(channelOnPeropheralView).then.connectToPeripherals()
```



5.设备连接成功后，开始读取Characteristics

5.After the device is connected successfully, start reading Characteristics

```objective-c
[baby setBlockOnDiscoverCharacteristics:^(CBPeripheral *peripheral, CBService *service, NSError *error) {
     NSLog(@"service name:%@",service.UUID);
     for (CBCharacteristic *c in service.characteristics) {
        NSLog(@"charateristic name is :%@",c.UUID);
     }
}];

baby.having(self.currPeripheral).and.channel(channelOnPeropheralString).then.connectToPeripherals().discoverServices().discoverCharacteristics().begin()
```



6.获取到charateristics之后我们可以获取charateristics的值或者订阅它的通知

6.After we get charateristics, we can get the value of charateristics or subscribe to its notifications

(1)读值

(1)Read Value


```objective-c
[baby setBlockOnReadValueForCharacteristicAtChannel:channelOnCharacteristicView block:^(CBPeripheral *peripheral, CBCharacteristic *characteristics, NSError *error) {
		NSLog(@"characteristic name:%@ value is:%@",characteristics.UUID,characteristics.value);
}];
//读取服务
//Read service
baby.channel(channelOnPeropheralString).characteristicDetails(self.currPeripheral,self.characteristic);
```

(2)订阅通知
(2) Subscribe to notifications

```objective-c
if (self.characteristic.properties & CBCharacteristicPropertyNotify || self.characteristic.properties & CBCharacteristicPropertyIndicate) {
        
		if(self.characteristic.isNotifying) {//取消通知   Cancellation notice
    		[baby cancelNotify:self.currPeripheral characteristic:self.characteristic];
      
  	}else{//订阅通知   Subscription notification
      [weakSelf.currPeripheral setNotifyValue:YES forCharacteristic:self.characteristic];
      [baby notify:self.currPeripheral  characteristic:self.characteristic
     block:^(CBPeripheral *peripheral, CBCharacteristic *characteristics, NSError *error) {
						NSLog(@"new value %@",characteristics.value);
			}];
   }
}
```



（3）写入数据

```objective-c
//设置写数据成功的回调
    [baby setBlockOnDidWriteValueForCharacteristicAtChannel:channelOnCharacteristicView block:^(CBCharacteristic *characteristic, NSError *error) {
         NSLog(@"setBlockOnDidWriteValueForCharacteristicAtChannel characteristic:%@ and new value:%@",characteristic.UUID, characteristic.value);
    }];

//写入数据
Byte b = 0X01;
NSData *data = [NSData dataWithBytes:&b length:sizeof(b)];
[self.currPeripheral writeValue:data forCharacteristic:self.characteristic type:CBCharacteristicWriteWithResponse];
```

 

## Android SDK文档说明


##Android SDK documentation

使用SDK前导入libCCNetBluetooth.aar至工程
Import libccnetbluetooth.aar to the project before using SDK



1.创建一个BluetoothClient，建议作为一个全局单例，管理所有BLE设备的连接。
1. Create a BluetoothClient. As a global example, it is recommended to manage the connections of all BLE devices.

```java
BluetoothClient mClient = new BluetoothClient(context);
```



2.每次扫描都要创建新的SearchRequest，不能复用。扫描到后通过onDeviceFounded返回扫描结果，保存device方便下一步连接设备
2. A new SearchRequest must be created for each scan and cannot be reused. After scanning, return the scanning result through onDeviceFounded, save the device to connect to the device in the next step

```java
SearchRequest request = new SearchRequest.Builder()
        .searchBluetoothLeDevice(3000, 3)   // 先扫BLE设备3次，每次3s   Scan ble device for 3 times, 3S each time
        .build();

mClient.search(request, new SearchResponse() {
    @Override
    public void onSearchStarted() {

    }

    @Override
    public void onDeviceFounded(SearchResult device) {
        Beacon beacon = new Beacon(device.scanRecord);
        BluetoothLog.v(String.format("beacon for %s\n%s", device.getAddress(), beacon.toString()));



    }

    @Override
    public void onSearchStopped() {

    }

    @Override
    public void onSearchCanceled() {

    }
});
```

如果需要手动停止扫描调用  
If you need to stop the scan call manually

```java
mClient.stopSearch();
```



3.扫描到设备后，对设备进行连接，传入第2步扫描到的设备Mac地址进行连接。
3. After scanning to the device, connect the device, and pass in the MAC address of the device scanned in step 2 to connect.

```java
//如果要监听蓝牙连接状态可以注册回调，只有两个状态：连接和断开。
//If you want to monitor the Bluetooth connection status, you can register a callback. There are only two statuses: connected and disconnected.
mClient.registerConnectStatusListener(mDevice.getAddress(), mBleConnectStatusListener);

private final BleConnectStatusListener mBleConnectStatusListener = new BleConnectStatusListener() {

    @Override
    public void onConnectStatusChanged(String mac, int status) {
        if (status == STATUS_CONNECTED) {

        } else if (status == STATUS_DISCONNECTED) {

        }
    }
};
mClient.unregisterConnectStatusListener(MAC, mBleConnectStatusListener);


//当然你也可以设置更多的连接参数进行连接
//Of course, you can also set more connection parameters to connect
BleConnectOptions options = new BleConnectOptions.Builder()
        .setConnectRetry(3)   // 连接如果失败重试3次  //Retry 3 times if connection fails
        .setConnectTimeout(30000)   // 连接超时30s   //Connection timeout 30s
        .setServiceDiscoverRetry(3)  // 发现服务如果失败重试3次  //Discovery service retries 3 times if it fails
        .setServiceDiscoverTimeout(20000)  // 发现服务超时20s  //Discovery service timeout 20s
        .build();
//开始进行连接  //Start connection
mClient.connect(MAC, options, new BleConnectResponse() {
    @Override
    public void onResponse(int code, BleGattProfile data) {

    }
});

//断开连接  //Disconnect
mClient.disconnect(MAC);
```



4.连接成功后从BleGattProfile data 读取serviceUUID、characterUUID，就可以开始读数据了
4. After the connection is successful, read serviceUUID、characterUUID from BleGattProfile data, and then start reading data



（1）读取数据
(1) Read data

```
mClient.read(MAC, serviceUUID, characterUUID, new BleReadResponse() {
    @Override
    public void onResponse(int code, byte[] data) {
        if (code == REQUEST_SUCCESS) {

        }
    }
});
```

（2）订阅通知
(2) Subscribe to notifications

```java
//订阅通知  //Subscribe to notifications
mClient.notify(MAC, serviceUUID, characterUUID, new BleNotifyResponse() {
    @Override
    public void onNotify(UUID service, UUID character, byte[] value) {
        
    }

    @Override
    public void onResponse(int code) {
        if (code == REQUEST_SUCCESS) {

        }
    }
});


//关闭通知   //Close notification
mClient.unnotify(MAC, serviceUUID, characterUUID, new BleUnnotifyResponse() {
    @Override
    public void onResponse(int code) {
        if (code == REQUEST_SUCCESS) {

        }
    }
});
```



（3）写入数据

要注意这里写的byte[]不能超过20字节，如果超过了需要自己分成几次写。建议的办法是第一个byte放剩余要写的字节的长度。

```java 
mClient.write(MAC, serviceUUID, characterUUID, bytes, new BleWriteResponse() {
    @Override
    public void onResponse(int code) {
        if (code == REQUEST_SUCCESS) {

        }
    }
});
```

