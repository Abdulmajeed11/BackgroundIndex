# BackgroundIndex (Background Server)
### Table of contents
- [DynamicIndexUpdated (Command 1200)](#1200a)
- [DynamicDeviceUpdated (Command 1200)](#1200b)
- [DynamicDeviceList (Command 1200)](#1200c)

<a name="1200a"></a>
## 1)DynamicIndexUpdated (Command 1200)
    Command no 
    1200- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC
   
    Redis
    
    /*if (data.index==0 && packet.cmsCode)*/
    2.hgetall on MAC:<packet.AlmondMAC>,data.DeviceID   
                    
             (or)

    2. Skip the code


    multi
    3.hmset on MAC:<payload.AlmondMAC>,deviceArray       //where deviceArray = paylaod

    SQl
    4.Select on AlmondplusDB.NotificationPreferences
      params: AlmondMAC,DeviceID,UserID
    5.Select on NotificationID
      params:UserID
    6.Select on DeviceData
      params: AlmondMAC,DeviceID

    Functional
    1.Command 1200

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dynamicIndexUpdated)->redisDeviceValue(getIndexValue)->device(updateIndex)->redisDeviceValue(update)->receive(mainFunction)->generator(deviceIndexUpdate)->scsi(sendFinal)

<a name="1200b"></a>
## 2)DynamicDeviceUpdated (Command 1200)
    Command no 
    1200- JSON format
 
    Redis

     /*if (data.index==0 && packet.cmsCode)*/
    2.hgetall on MAC:<packet.AlmondMAC>,data.DeviceID   
                    
             (or)

    2. Skip the code

    multi
    3.hmset on MAC:<payload.AlmondMAC>,deviceArray       //where deviceArray = paylaod

    SQl
    4.Insert on AlmondplusDB.DEVICE_DATA
      params:AlmondMAC

    Required 
    Command,CommandType,Payload,almondMAC

    Functional
    1.Command 1200
    
    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dynamicDeviceUpdated)->device(execute)->redisDeviceValue(update)->genericModel(update), genericModel(add)->receive(mainFunction)->scsi(sendFinal)

<a name="1200c"></a>
## 3)DynamicDeviceList (Command 1200)
    Command no 
    1200- JSON format

    Redis
    2.hmget on DV_<payload AlmondMAC>
    3.hmset on DV_<payload AlmondMAC>
    4.hgetall on MAC:<payload AlmondMAC>

    multi
    8.del on MAC:<payload AlmondMAC>:<removeIds>

    Cassandra
    5.Insert on notification_store.almondhistory
      params: mac,type,data,time
    9.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
      params: CA.CMSCode,AU.AlmondMAC

    SQl
    6.Delete on DEVICE_DATA
      params: AlmondMAC,id
    7.Insert on AlmondplusDB
      params: AlmondMAC

    Functional
    1.Command 1200

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dymamicAddAllDevice )->device(addAll)->redisDeviceValue(getDeviceList)->genericModel(insertBackUpAndUpdate)-> genericModel(removeAndInsert)->redisDeviceValue(addAll)->receive(mainFunction)->scsi(sendFinal)


