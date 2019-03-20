# BackgroundIndex  (Background Server)
### Table of contents
- [DynamicIndexUpdated (Command 1200)](#1200a)

<a name="1200a"></a>
## 1)DynamicIndexUpdated (Command 1200)
    Command no 
    1200- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC
   
    Redis

    multi
    2.hmset on MAC:payload.AlmondMAC,deviceArray       //where deviceArray = paylaod

    SQl
    3.Select on AlmondplusDB.NotificationPreferences
      params: AlmondMAC,DeviceID,UserID
    4.Select on NotificationID
      params:UserID
    5.Select on DeviceData
      params: AlmondMAC,DeviceID

    Functional
    1.Command 1200

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dynamicIndexUpdated)->redisDeviceValue(getIndexValue)->device(updateIndex)->redisDeviceValue(update)->receive(mainFunction)
