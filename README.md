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
- [DynamicClientUpdated (Command 1500)](#1500b)
- [DynamicClientList (Command 1500)](#1500c)
- [DynamicClientRemoved (Command 1500)](#1500d)
- [DynamicAllClientsRemoved (Command 1500)](#1500e)
- [DynamicAllDevicesRemoved (Command 1200)](#1200e)
- [DynamicClientJoined (Command 1500)](#1500f)
- [DynamicClientLeft (Command 1500)](#1500g)
- [DynamicAlmondNameChange (Command 49)](#49)
- [DynamicAlmondModeChange (Command 153)](#153)
- [DynamicAlmondProperties (Command 1050)](#1050)

<a name="1200a"></a>
## 1)DynamicIndexUpdated (Command 1200)
    Command no 
    1200- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC
   
    REDIS
    
    /*if (data.index==0 && packet.cmsCode)*/
    2. hgetall on MAC:<AlmondMAC>,data.DeviceID   
                    
             (or)

    2. Return

    /* if(Object.keys(variables).length==0) */
      multi   
    3.hmset on MAC:<AlmondMAC>:Key, variables    
    //Above, key = Device keys, variables = Device Values
     
                       (or)

     /* if (deviceArray.length>0) */
      multi
    3.hmset on MAC:<AlmondMAC>,deviceArray        //Where deviceAray =Device keys in Payload

     7.LPUSH on AlmondMAC_Device                // params: redisData

     /* if (res > count + 1) */
     8.LTRIM on AlmondMAC_Device                //here count = 9, res = Result from step 10
               
                (or)

     /* if (res == 1) */
     8.expire on AlmondMAC_Device             //here, res = Result from step 10

     9.LPUSH on AlmondMAC_All               // params: redisData

     /* if (res > count + 1) */
     10.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 12
               
               (or)

     /* if (res == 1) */
     10.expire on AlmondMAC_All             //here, res = Result from above step 12

     Postgres
     11.Insert on recentactivity
       params: mac, id, time, index_id, index_name, name, type, value

     Cassandra
     14.Insert on notification_store.notification_records
        params: usr_id, noti_time, i_time, msg
     15.Update on notification_store.badger  
        params: user_id  
     16.Select on notification_store.badger
        params: usr_id

    SQl
    4.Select on AlmondplusDB.NotificationPreferences
      params: AlmondMAC,DeviceID,UserID
    5.Select on NotificationID
      params:UserID
    6.Select on DeviceData
      params: AlmondMAC,DeviceID
    17.Update on AlmondplusDB.NotificationID
       params:RegID

     /*if (oldRegid && oldRegid.length > 0) */
     18.Delete on AlmondplusDB.NotificationID
        params: RegID
     19.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
        params: CA.CMSCode,AU.AlmondMAC 

    Functional
    1.Command 1200
    12.delete ans.AlmondMAC;
       delete ans.CommandType;
       delete ans.Action;
       delete ans.HashNow;
       delete ans.Devices;
       delete ans.epoch
    13.delete input.users

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dynamicIndexUpdated)->redisDeviceValue(getIndexValue)->device(updateIndex)->redisDeviceValue(update)->receive(mainFunction)->receive(dynamicIndexUpdate)->generator(deviceIndexUpdate)->cassQueries(qtoCassHistory)->cassQueries(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)->scsi(sendFinal)

<a name="1200b"></a>
## 2)DynamicDeviceUpdated (Command 1200)
    Command no 
    1200- JSON format

    Required 
    Command,CommandType,Payload,almondMAC
 
    REDIS
    
    /* if(Object.keys(variables).length==0) */
    multi   
    2.hmset on MAC:<AlmondMAC>:Key, variables    
    //Above, key = Device keys, variables = Device Values
     
                       (or)

     /* if (deviceArray.length>0) */
     multi
    2.hmset on MAC:<AlmondMAC>,deviceArray        //Where deviceAray =Device keys in Payload

    SQl
    3.Insert on AlmondplusDB.DEVICE_DATA
      params:AlmondMAC
    4.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
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

    REDIS
    2.hgetall on MAC:<AlmondMAC>

    multi
    4.hmset on AL_<AlmondMAC>       // values = [deviceRestore,restore]

    multi
    5.hgetall on MAC:<AlmondMAC>:deviceIDs

    multi
    9.del on MAC:<almondMAC>

    multi
    10.del on MAC:<payload AlmondMAC>:<removeIds>

    /* if(Object.keys(variables).length==0) */
    multi   
    11.hmset on MAC:<AlmondMAC>:Key, variables    
    //Above key = Device keys, variables = Device Values
     
                       (or)

     /* if (deviceArray.length>0) */
     multi
    11.hmset on MAC:<AlmondMAC>,deviceArray        //Where deviceAray =Device keys in Payload


    Cassandra
    6.Insert on notification_store.almondhistory
      params: mac,type,data,time

    SQl
    3.Select on DEVICE_DATA
      params:AlmondMAC
    7.Delete on DEVICE_DATA
      params: AlmondMAC
    8.Insert on AlmondplusDB.DEVICE_DATA
      params: AlmondMAC
    12.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
      params: CA.CMSCode,AU.AlmondMAC

    Functional
    1.Command 1200

    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dymamicAddAllDevice)->device(addAll)->redisDeviceValue(getDeviceList)->genericModel(insertBackUpAndUpdate)->device(getDevices)->genericModel(get)->redisDeviceValue(getFormatted)->genericModel(removeAndInsert)->redisDeviceValue(addAll)->receive(mainFunction)->scsi(sendFinal)

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

    REDIS
    
    /* if(Object.keys(variables).length==0) */
    multi   
    2.hmset on MAC:<AlmondMAC>:Key, variables    
    //Above, key = Device keys, variables = Device Values
     
                       (or)

     /* if (deviceArray.length>0) */
     multi
    2.hmset on MAC:<AlmondMAC>,deviceArray        //Where deviceAray =Device keys in Payload

    Functional
    1.Command 1200
    
    Flow
    consumer(processMessage)->controller(processor)->preprocessor(dymamicAddAllDevice)->device(execute)->redisDeviceValue(add)->genericModel(add)->receive(mainFunction)->scsi(sendFinal)

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
    15.Update on AlmondplusDB.NotificationID
       params:RegID

    /*if (oldRegid && oldRegid.length > 0) */
    16.Delete on AlmondplusDB.NotificationID
       params: RegID

    REDIS
    4.hmget on AL_<AlmondMAC>                   // params: ["name"]
    5.LPUSH on AlmondMAC_Client                 // params: redisData

    /* if (res > count + 1) */
    6.LTRIM on AlmondMAC_Client                //here count = 9, res = Result from step 5
               
               (or)

    /* if (res == 1) */
    6.expire on AlmondMAC_Client             //here, res = Result from step 5
    
    7.LPUSH on AlmondMAC_All                // params: redisData
    /* if (res > count + 1) */
    8.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 7
               
               (or)

    /* if (res == 1) */
    8.expire on AlmondMAC_All             //here, res = Result from above step 7

    Postgres
    9.Insert on recentactivity
      params: mac, id, time, index_id, client_id, name, type, value

    Cassandra
    12.Insert on notification_store.notification_records
       params: usr_id, noti_time, i_time, msg 
    13.Update on notification_store.badger
       params: usr_id
    14.Select on notification_store.badger
       params: usr_id 
    
    Functional
    1.Command 1500
    10.delete ans.AlmondMAC;
      delete ans.CommandType;
      delete ans.Action;
      delete ans.HashNow;
      delete ans.Devices;
      delete ans.epoch;

    11.delete input.users;

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(add)->receive(mainFunction)->notify(sendAlwaysClient)->generator(wifiNotificationGenerator)->cassandra(qtoCassHistory)->cassandra(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)

<a name="1500b"></a>
## 15)DynamicClientUpdated (Command 1500)
    Command no 
    1500- JSON format

    Required 
    Command,CommandType,Payload,almondMAC

    SQl
    2.Insert on AlmondplusDB.WIFICLIENTS
      params: AlmondMAC

    Functional
    1.Command 1500

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(update)->receive(mainFunction)

<a name="1500c"></a>
## 16)DynamicClientList (Command 1500)
     Command no 
     1500- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     SQl
     2.Insert on  AlmondplusDB.WIFICLIENTS
       params: AlmondMAC
     3.Delete on WIFICLIENTS
       params: AlmondMAC
     4.Insert on  AlmondplusDB.WIFICLIENTS
      params: AlmondMAC

     Functional
     1.Command 1500
    
     Flow
     socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(addAll),genericModel(insertBackUpAndUpdate),genericModel(get),cassandra(execute)->genericModel(removeAndInsert)->receive(mainFunction)

<a name="1500d"></a>
## 17)DynamicClientRemoved (Command 1500)
     Command no 
     1500- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     SQl
     2.Select on WifiClients
       params: AlmondMAC,ClientID
     3.Delete on WIFICLIENTS
       params: AlmondMAC
     4.Select on NotificationID
       params: UserID
     16.Update on AlmondplusDB.NotificationID
        params:RegID

     /*if (oldRegid && oldRegid.length > 0) */
     17.Delete on AlmondplusDB.NotificationID
        params: RegID

     REDIS
     5.hmget on AL_<AlmondMAC>                 //// params: ["name"]
     6.LPUSH on AlmondMAC_Client               // params: redisData

     /* if (res > count + 1) */
     7.LTRIM on AlmondMAC_Client                //here count = 9, res = Result from step 6
               
                (or)

     /* if (res == 1) */
     7.expire on AlmondMAC_Client             //here, res = Result from step 6

     8.LPUSH on AlmondMAC_All               // params: redisData
     /* if (res > count + 1) */
     9.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 8
               
               (or)

     /* if (res == 1) */
     9.expire on AlmondMAC_All             //here, res = Result from above step 8

     Postgres
     10.Insert on recentactivity
       params: mac, id, time, index_id, client_id, name, type, value

     Cassandra
     13.Insert on notification_store.notification_records
        params: usr_id, noti_time, i_time, msg
     14.Update on notification_store.badger  
        params: user_id  
     15.Select on notification_store.badger
        params: usr_id 
    
     Functional
     1.Command 1500
     11.delete ans.AlmondMAC;
        delete ans.CommandType;
        delete ans.Action;
        delete ans.HashNow;
        delete ans.Devices;
        delete ans.epoch;

     12.delete input.users;

     Flow
     socket(packet)->controller(processor)->preprocessor(dynamicClientRemoved)->genericModel(execute)->genericModel(remove)->receive(mainFunction)->receive(sendAlwaysClient)->generator(wifiNotificationGenerator)->cassQueries(qtoCassHistory)->cassQueries(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)

<a name="1500e"></a>
## 18)DynamicAllClientsRemoved (Command 1500)
     Command no 
     1500- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     SQl
     3.Delete on WIFICLIENTS
       params: AlmondMAC
     4.Select on NotificationID
       params: UserID
     11.Update on AlmondplusDB.NotificationID
        params:RegID

      /*if (oldRegid && oldRegid.length > 0) */
      12.Delete on AlmondplusDB.NotificationID
        params: RegID

     Redis
     5.hmget on AL_<AlmondMAC>

     Cassandra
     2.Insert on notification_store.almondhistory
       params: mac,type,data,time
     8.Insert on notification_store.notification_records
        params: usr_id, noti_time, i_time, msg
     9.Update on notification_store.badger  
        params: user_id  
     10.Select on notification_store.badger
        params: usr_id 
     
     Functional
     1.Command 1500
     6.delete ans.AlmondMAC;
        delete ans.CommandType;
        delete ans.Action;
        delete ans.HashNow;
        delete ans.Devices;
        delete ans.epoch;

     7.delete input.users;

     Flow
     socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(removeAll)->receive(mainFunction)->receive(AlwaysTrue)->generator(wifiNotificationGenerator)->cassQueries(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)

<a name="1200e"></a>
## 19)DynamicAllDevicesRemoved (Command 1200)
     Command no 
     1200- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     SQl
     7.Delete on DEVICE_DATA
       params: AlmondMAC
     8.Select on NotificationID
       params: UserID
     20.Update on AlmondplusDB.NotificationID
        params:RegID

     /*if (oldRegid && oldRegid.length > 0) */
     21.Delete on AlmondplusDB.NotificationID
        params: RegID

     REDIS
     2.hgetall on MAC:<AlmondMAC>

     multi
     3.del on MAC:<AlmondMAC>

     multi 
     4.del on  MAC:<AlmondMAC>:deviceIds

     multi
     5.hdel on AL_<AlmondMAC>           // value = deviceRestore

     9.hmget on AL_<AlmondMAC>
     10.LPUSH on AlmondMAC_Device                // params: redisData

     /* if (res > count + 1) */
     11.LTRIM on AlmondMAC_Device                //here count = 9, res = Result from step 10
               
                (or)

     /* if (res == 1) */
     11.expire on AlmondMAC_Device             //here, res = Result from step 10

     12.LPUSH on AlmondMAC_All               // params: redisData
     /* if (res > count + 1) */
     13.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 12
               
               (or)

     /* if (res == 1) */
     13.expire on AlmondMAC_All             //here, res = Result from above step 12

     Postgres
     14.Insert on recentactivity
       params: mac, id, time, index_id, index_name, name, type, value

     Cassandra
     6. Insert on  notification_store.almondhistory
        params: mac,type,data,time
     17.Insert on notification_store.notification_records
        params: usr_id, noti_time, i_time, msg
     18.Update on notification_store.badger  
        params: user_id  
     19.Select on notification_store.badger
        params: usr_id 

     Functional
     1.Command 1200
    15.delete ans.AlmondMAC;
       delete ans.CommandType;
       delete ans.Action;
       delete ans.HashNow;
       delete ans.Devices;
       delete ans.epoch
    16.delete input.users

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->device(execute)->redisDeviceValue(removeAll)->genericModel(removeAll)->receive(mainFunction)->receive(AlwaysTrue)->generator(wifiNotificationGenerator)->cassQueries(qtoCassHistory)->cassQueries(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)   

<a name="1500f"></a>
## 20)DynamicClientJoined (Command 1500)
     Command no 
     1500- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     SQl
     2.Insert on AlmondplusDB.WIFICLIENTS
       params: AlmondMAC
     3.Select on AlmondplusDB.WifiClientsNotificationPreferences
       params: AlmondMAC,ClientID,UserID
     4.Select on NotificationID
       params: UserID
     16.Update on AlmondplusDB.NotificationID
        params:RegID

     /*if (oldRegid && oldRegid.length > 0) */
     17.Delete on AlmondplusDB.NotificationID
        params: RegID

     REDIS 
     5.hmget on AL_<almondMAC>           // params: ["name"]
     6.LPUSH on AlmondMAC_Client         // params: redisData

     /* if (res > count + 1) */
     7.LTRIM on AlmondMAC_Client                //here count = 9, res = Result from step 6
               
                (or)

     /* if (res == 1) */
     7.expire on AlmondMAC_Client             //here, res = Result from step 6

     8.LPUSH on AlmondMAC_All               //params: redisData
     /* if (res > count + 1) */
     9.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 8
               
               (or)

     /* if (res == 1) */
     9.expire on AlmondMAC_All             //here, res = Result from above step 8

     Postgres
     10.Insert on recentactivity
       params: mac, id, time, index_id, client_id, name, type, value

     Cassandra
     13.Insert on notification_store.notification_records
        params: usr_id, noti_time, i_time, msg
     14.Update on notification_store.badger  
        params: user_id  
     15.Select on notification_store.badger
        params: usr_id

     Functional
     1.Command 1500
     11.delete ans.AlmondMAC;
        delete ans.CommandType;
        delete ans.Action;
        delete ans.HashNow;
        delete ans.Devices;
        delete ans.epoch;

     12.delete input.users;

     Flow
     socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(update)->receive(mainFunction)->receive(checkClientPreference)->generator(wifiNotificationGenerator)->cassQueries(qtoCassHistory)->cassQueries(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)

<a name="1500g"></a>
## 21)DynamicClientLeft (Command 1500)
     Command no 
     1500- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     SQl
     2.Insert on AlmondplusDB.WIFICLIENTS
       params: AlmondMAC
     3.Select on AlmondplusDB.WifiClientsNotificationPreferences
       params: AlmondMAC,ClientID,UserID
     4.Select on NotificationID
     16.Update on AlmondplusDB.NotificationID
        params:RegID

     /*if (oldRegid && oldRegid.length > 0) */
     17.Delete on AlmondplusDB.NotificationID
        params: RegI

     REDIS 
     5.hmget on AL_<almondMAC>           // params: ["name"]
     6.LPUSH on AlmondMAC_Client         // params: redisData

     /* if (res > count + 1) */
     7.LTRIM on AlmondMAC_Client                //here count = 9, res = Result from step 6
               
                (or)

     /* if (res == 1) */
     7.expire on AlmondMAC_Client             //here, res = Result from step 6

     8.LPUSH on AlmondMAC_All               //params: redisData
     /* if (res > count + 1) */
     9.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 8
               
               (or)

     /* if (res == 1) */
     9.expire on AlmondMAC_All             //here, res = Result from above step 8

     Postgres
     10.Insert on recentactivity
       params: mac, id, time, index_id, client_id, name, type, value

     Cassandra
     13.Insert on notification_store.notification_records
        params: usr_id, noti_time, i_time, msg
     14.Update on notification_store.badger  
        params: user_id  
     15.Select on notification_store.badger
        params: usr_id

     Functional
     1.Command 1500
     11.delete ans.AlmondMAC;
       delete ans.CommandType;
       delete ans.Action;
       delete ans.HashNow;
       delete ans.Devices;
       delete ans.epoch;

     12.delete input.users;

     Flow
     socket(packet)->controller(processor)->preprocessor(doNothing)->genericModel(execute)->genericModel(update)->receive(mainFunction)->receive(checkClientPreference)->generator(wifiNotificationGenerator)->cassQueries(qtoCassHistory)->cassQueries(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)

<a name="49"></a>
## 22)DynamicAlmondNameChange (Command 49)
     Command no 
     49- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     REDIS
     2.hmset on AL_<AlmondMAC>          // params: [redisKey[key], data[key]     
 
     Functional
     1.Command 49

     Flow
     socket(packet)->controller(processor)->preprocessor(doNothing)->almondCommands(almond_name_change)

<a name="153"></a>
## 23)DynamicAlmondModeChange (Command 153)
     Command no 
     153- JSON format

     Required 
     Command,CommandType,Payload,almondMAC

     REDIS
     2.hmset on AL_<AlmondMAC>          // params: [redisKey[key], data[key]     
 
     Functional
     1.Command 153

     Flow
     socket(packet)->controller(processor)->preprocessor(doNothing)->almondCommands(changeMode)

<a name="1050"></a>
## 24)DynamicAlmondProperties (Command 1050)
     Command no
     1050- JSON format

     Required
     Command,CommandType,Payload,almondMAC

     SQl
     3.Select on ALMONDPROPERTIES
       params:AlmondMAC
     4.Update on AlmondProperties2
       params:AlmondMAC
     5.select from NotificationID 
       params:UserID 
     17.Update on AlmondplusDB.NotificationID
        params:RegID

     /*if (oldRegid && oldRegid.length > 0) */
     18.Delete on AlmondplusDB.NotificationID
        params: RegI
     19.Select on SCSIDB.CMSAffiliations,AlmondplusDB.AlmondUsers,SCSIDB.CMS
        params: CA.CMSCode,AU.AlmondMAC 


     Redis
     2.hmset on AL_<AlmondMAC>                   // params: [redisKey[key], data[key]]
     6.hmget on AL_<almondMAC>                     // params: ["name"]

     7.LPUSH on AlmondMAC_Device         // params: redisData

     /* if (res > count + 1) */
     8.LTRIM on AlmondMAC_Device                //here count = 9, res = Result from step 7
               
                (or)

     /* if (res == 1) */
     8.expire on AlmondMAC_Device             //here, res = Result from step 7

     9.LPUSH on AlmondMAC_All               //params: redisData
     /* if (res > count + 1) */
     10.LTRIM on AlmondMAC_All                //here count = 19, res = Result from step 9
               
               (or)

     /* if (res == 1) */
     10.expire on AlmondMAC_All             //here, res = Result from above step 9

     Postgres
     11.Insert on recentactivity
        params: mac, id, time, index_id, index_name, name, type, value

     Cassandra
     14.Insert on notification_store.notification_records
        params: usr_id, noti_time, i_time, msg
     15.Update on notification_store.badger  
        params: user_id  
     16.Select on notification_store.badger
        params: usr_id

    Functional
    1.Command 1050
    12.delete ans.AlmondMAC;
       delete ans.CommandType;
       delete ans.Action;
       delete ans.HashNow;
       delete ans.Devices;
       delete ans.epoch;
    13.delete input.users;

    Flow
    socket(packet)->controller(processor)->preprocessor(doNothing)->almondCommands(DynamicAlmondProperties)->genericModel(get)->receive(mainFunction)->receive(almondProperties)->generator(propertiesNotification)->cassQueries(qtoCassHistory)->cassQueries(qtoCassConverter)->msgService(notificationHandler)->msgService(handleResponse)->scsi(sendFinal)
