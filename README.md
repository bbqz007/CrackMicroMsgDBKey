# CrackMicroMsgDBKey
Crack MicroMsg (or WeChat) DB Key, by using WinDbg
# overview
Well known as that, a MicroMsg (or WeChat) App uses sqlcipher to protect the DBs in the client site. the android App used md5(imei+wxid) for the db key since the year more earlier than 2017, i am not sure they work like that still? but i am sure the the PC App productions use a 32 bytes db hexkey, and the key is calculated by the server, then storaged by the client site. thus we can scan the running memory of the client site to crack the db key.

# where
the key is kept by the **AccountService** class. you can have the acknowledgement of the position of the key by the method `AccountService::setDBKey`. you need to reverse the this method and do some analysis when you want to get more details about it. otherwise, Fortunately, i have [**how to**](#howto) help you use windbg only to crack the db key simply.

**AccountService** has two important members. they keep a PubKey and PriKey for some RSA stuffs. and the hexkey locate in fixed offset relating with these member.( hexkey is one member of AccountService as well). we can find out these two members by `-----BEGIN RSA PRIVATE KEY-----` or `-----BEGIN PUBLIC KEY-----`. it seems no changes for such many years and versions, eventhough now on 2024/9/21.

Hi, my github is https://github.com/bbqz007.

# warnig
* there is no skilled requirement to crack your db key. any process can scan your MicroMsg (or WeChat) process by only a api `ReadProcessMemory`.
* a security soft or Input Method Software, scans your PC more natively. they inject their dlls in every process.
* this is a white hat to help you know about the risks. a user should have rights to have the acknowledgement.
* when you use search engine with key like `wechat db github`, you can get a lot repos.
* i don't give any software. you can use windbg following my steps. there is only a choice.

# howto
## x86
* `s-a 0 L10000000 "-----BEGIN RSA PRIVATE KEY-----"`
  * theRes, use `alias theRes ` to assign the result address.
  * or replace the theRes below with the result address by hand.
* `s-d wechatwin L2000000 theRes 0 0 0 377 37f`
  * theRes2, use `alias theRes2 ` to assign the result address.
  * or replace the theRes2 below with the result address by hand.
* `da poi(theRes2 - 18)`
  * check if "-----BEGIN PUBLIC KEY-----"
* `db poi(theRes2 - a8) Lpoi(theRes2 - a4)`
  * you got it if it shows 32 bytes.
  * collect the hex bytes and make lowercase letters without spaces.

<img src="imgs/WeChat.x86.crackkey.png" width="61.8%">

## x64 
* `s-a 0 L10000000 "-----BEGIN RSA PRIVATE KEY-----"`
  * theRes, use `alias theRes ` to assign the result address.
  * or replace the theRes below with the result address by hand.
* `s-q wechatwin L2000000 theRes 0 377 37f`
  * theRes2, use `alias theRes2 ` to assign the result address.
  * or replace the theRes2 below with the result address by hand.
* `da poi(theRes2 - 20)`
  * check if "-----BEGIN PUBLIC KEY-----"
* `db poi(theRes2 - f8) Lpoi(theRes2 - f0)`
  * you got it if it shows 32 bytes.
  * collect the hex bytes and make lowercase letters without spaces.

<img src="imgs/WeChat.x64.crackkey.png" width="61.8%">
 
# preview
i have the personal kit (https://github.com/bbqz007/KTL) to preview the MicroMsg db. when you get your hex key, and want to view your DBs, it can help you. my kit does not link any ws2_32.dll or any dll link to ws2_32.dll。my kit has a jitC can run cpp code as plugin. i write a patch AlgoDataTool.cpp (https://github.com/bbqz007/KTL/tree/master/patch/src/wechat-db) for setting hex key and other cipher stuffs. you can modify it for any and decide wether to run or not. my kit is mainly for offline analysis to stock market, and the data source including sqlite3-like db.

<img src="imgs/GIF_KTL_WeChatDb.gif" width="61.8%">

# downloads
* [MicroMsg (or WeChat) x64](https://github.com/tom-snow/wechat-windows-versions) : https://github.com/tom-snow/wechat-windows-versions
* [MicroMsg (or WeChat) x86](https://github.com/tom-snow/wechat-windows-versions-x86) : https://github.com/tom-snow/wechat-windows-versions-86
