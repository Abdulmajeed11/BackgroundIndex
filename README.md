# BackgroundIndex (Background Server)
### Table of contents
- [DynamicIndexUpdated (Command 1200)](#1200a)
- [DynamicDeviceUpdated (Command 1200)](#1200b)
- [DynamicDeviceList (Command 1200)](#1200c)
- [DynamicDeviceAdded (Command 1200)](#1200d)
- [DynamicRuleAdded (Command 1400)](#1400a)
- [DynamicRuleUpdated (Command 1400)](#1400b)
- [DynamicRuleRemoved (Command 1400)](#1400c)
- [DynamicAllRulesRemoved (Command 1400)](#1400d)
- [DynamicSceneAdded (Command 1300)](#1300a)
- [DynamicSceneActivated (Command 1300)](#1300b)
- [DynamicSceneUpdated (Command 1300)](#1300c)
- [DynamicSceneRemoved (Command 1300)](#1300d)
- [DynamicAllSceneRemoved (Command 1300)](#1300e)
- [DynamicClientAdded (Command 1500)](#1500a)

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
    3.hmset on MAC:<payload.AlmondMAC>,deviceArray     //Where deviceAray =Device keys in Payload

    SQl
    4.Select on AlmondplusDB.NotificationPreferences
      params: AlmondMAC,DeviceID,UserID
    5.Select on NotificationID
      params:UserID
    6.Select on DeviceData
      params: AlmondMAC,DeviceID
    7.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
      params: CA.CMSCode,AU.AlmondMAC

    Functional
    1.Command 1200

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dynamicIndexUpdated)->redisDeviceValue(getIndexValue)->device(updateIndex)->redisDeviceValue(update)->receive(mainFunction)->generator(deviceIndexUpdate)->scsi(sendFinal)

<a name="1200b"></a>
## 2)DynamicDeviceUpdated (Command 1200)
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
    3.hmset on MAC:<payload.AlmondMAC>,deviceArray     //Where deviceAray =Device keys in Payload

    SQl
    4.Insert on AlmondplusDB.DEVICE_DATA
      params:AlmondMAC
    5.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
      params: CA.CMSCode,AU.AlmondMAC

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

    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    2.hmget on DV_<payload AlmondMAC>
    3.hmset on DV_<payload AlmondMAC>
    4.hgetall on MAC:<payload AlmondMAC>

    multi
    8.del on MAC:<payload AlmondMAC>:<removeIds>

    Cassandra
    5.Insert on notification_store.almondhistory
      params: mac,type,data,time

    SQl
    6.Delete on DEVICE_DATA
      params: AlmondMAC,id
    7.Insert on AlmondplusDB
      params: AlmondMAC
    9.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
      params: CA.CMSCode,AU.AlmondMAC

    Functional
    1.Command 1200

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dymamicAddAllDevice)->device(addAll)->redisDeviceValue(getDeviceList)->genericModel(insertBackUpAndUpdate)-> genericModel(removeAndInsert)->redisDeviceValue(addAll)->receive(mainFunction)->scsi(sendFinal)

<a name="1200d"></a>
## 4)DynamicDeviceAdded (Command 1200)
    Command no 
    1200- JSON format

    Required 
    Command,CommandType,Payload,almondMAC

    SQl
    3.Insert on AlmondplusDB.DEVICE_DATA
      params: AlmondMAC
    4.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
      params: CA.CMSCode,AU.AlmondMAC

    Redis
    2.hmset on MAC:<payload.AlmondMAC>,deviceArray     //Where deviceAray =Device keys in Payload

    Functional
    1.Command 1200
    
    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dymamicAddAllDevice)->device(execute)->redisDeviceValue[add]->genericModel[add]->receive(mainFunction)->scsi(sendFinal)

<a name="1400a"></a>
## 5)DynamicRuleAdded (Command 1400)
    Command no 
    1400- JSON format

    Required 
    Command,CommandType,Payload,almondMAC

    SQl
    2.Insert on AlmondplusDB.RULE
      params: AlmondMAC
    
    Functional
    1.Command 1400
    
    Flow
    consumer(processMessage)->controller(processor)->preprocessor(doNothing)->genericModel(execute),genericModel(add)

<a name="1400b"></a>
## 6)DynamicRuleUpdated (Command 1400)
    Command no 
    1400- JSON format

    Required 
    Command,CommandType,Payload,almondMAC

    SQl
    2.Insert on AlmondplusDB.RULE
      params: AlmondMAC

    Functional
    1.Command 1400

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(doNothing)->genericModel(execute),genericModel(Update)

<a name="1400c"></a>
## 7)DynamicRuleRemoved (Command 1400)
    Command no 
    1400- JSON format

    Required 
    Command,CommandType,Payload,almondMAC

    SQl
    2.Delete on RULE
      params: AlmondMAC

    Functional
    1.Command 1400

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(doNothing)->genericModel(execute),genericModel(remove)

<a name="1400d"></a>
## 8)DynamicAllRulesRemoved (Command 1400)
    Command no 
    1400- JSON format

    Required 
    Command,CommandType,Payload,almondMAC

    Cassandra
    2.Insert on notification_store.almondhistory
      params: mac,type,data,time

    SQl
    3.Delete on RULE
      params: AlmondMAC

    Functional
    1.Command 1400

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(doNothing)->genericModel(execute),genericModel(removeAll)

<a name="1300a"></a>
## 9)DynamicSceneAdded (Command 1300)
    Command no
    1300- JSON format

    Required
    Command,CommandType,Payload,almondMAC

    SQl
    2.Insert on AlmondplusDB.SCENE
      params:AlmondMAC

    Functional
    1.Command 1300

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(add) 

<a name="1300b"></a>
## 10)DynamicSceneActivated (Command 1300)
    Command no
    1300- JSON format

    Required
    Command,CommandType,Payload,almondMAC

    SQl
    2.Insert on AlmondplusDB.SCENE
      params:AlmondMAC

    Functional
    1.Command 1300

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(update)

<a name="1300c"></a>
## 11)DynamicSceneUpdated (Command 1300)
    Command no
    1300- JSON format

    Required
    Command,CommandType,Payload,almondMAC

    SQl
    2.Insert on AlmondplusDB.SCENE
      params:AlmondMAC

    Functional
    1.Command 1300

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(update)

<a name="1300d"></a>
## 12)DynamicSceneRemoved (Command 1300)
    Command no
    1300- JSON format

    Required
    Command,CommandType,Payload,almondMAC

    SQl
    2.Delete on SCENE
      params:AlmondMAC

    Functional
    1.Command 1300

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(remove)

<a name="1300e"></a>
## 13)DynamicAllSceneRemoved (Command 1300)
    Command no
    1300- JSON format

    Required
    Command,CommandType,Payload,almondMAC

    SQl
    2.Delete on SCENE
      params:AlmondMAC

    Functional
    1.Command 1300

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(remove)

<a name="1500a"></a>
## 14)DynamicClientAdded (Command 1500)
    Command no 
    1500- JSON format

    Required 
    Command,CommandType,Payload,almondMAC

    SQl
    2.Insert on AlmondplusDB.WIFICLIENTS
      params: AlmondMAC
    3.Select on NotificationID
      params: UserID
    13.Update on AlmondplusDB.NotificationID
       params:RegID

    /*if (oldRegid && oldRegid.length > 0) */
    14.Delete on AlmondplusDB.NotificationID
       params: RegID

    Redis
    4.hmget on AL_<input.AlmondMAC>
    5.LPUSH on AlmondMAC_Client

    /* if (res > count + 1) */
    6.LTRIM on AlmondMAC_Client                //here count = 9, res = Result from step 5
               
               (or)

    /* if (res == 1) */
    6.expire on AlmondMAC_Client             //here, res = Result from step 5

    /* if (res > count + 1) */
    7.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 5
               
               (or)

    /* if (res == 1) */
    7.expire on AlmondMAC_All             //here, res = Result from above step 5

    Postgre
    8.Insert on recentactivity
      params: mac, id, time, index_id, client_id, name, type, value

    Cassandra
    11.Insert on notification_store.notification_records
       params: usr_id, noti_time, i_time, msg 
    12.Select on notification_store.badger
       params: usr_id 
    
    Functional
    1.Command 1500
    9.delete ans.AlmondMAC;
      delete ans.CommandType;
      delete ans.Action;
      delete ans.HashNow;
      delete ans.Devices;
      delete ans.epoch;

    10.delete input.users;

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(add)->receive(mainFunction)->notify(sendAlwaysClient)->generator(wifiNotificationGenerator)->cassandra(qtoCassHistory)->cassandra(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)
