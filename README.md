# DonkeyID---php扩展-64位自增ID生成器
##原理
	参考Twitter-Snowflake 算法,扩展了其中的细节。具体组成如下图：
	
![bits.jpg](https://github.com/osgochina/donkeyid/blob/master/doc/bits.jpg?raw=true)

> 如图所示，64bits 咱们分成了4个部分。
> 1. 毫秒级的时间戳,有42个bit.能够使用139年，从1970年开始计算，能使用到2109年，当然这些是可以扩展的,可以通知指定起始时间来延长这个日期长度。
> 2. 自定义节点id,防止多进程运行产生重复id,能够256个节点。部署的时候可以配置好服务器id;
> 4. 进程workerid，占位5bit，能够生成32个进程id。根据pid运算获得。
> 4. 进程内毫秒时间自增序号。一毫秒能产生512个id。也就是说并发1秒能产生512000个id。
##使用
###安装
> 下载代码到本地，进入项目文件夹，执行

```Bash
/path/to/phpize &&  ./configure && make && make install
```

```Bssh
echo "extension=donkeyid.so" >> /path/to/php.ini
```

###运行
####api接口
*
* new DonkeyId($type=0,$epoch=0);//$type 类型 值有0,1 epoch 纪元开始时间戳 可以设置从此开始计算秒数
* boolean setNodeId($node_id);
* string getNextId();
* string parseTime($id);
* int parseNodeId($id);
* int parseWorkerId($id);
* int parseSequence($id);

####测试代码
```php

    $donkey = new DonkeyId();
    $donkey->setNodeId(11); //0-511 不要超过这个值
    $id = $donkey->getNextId();
    $time = $donkey->parseTime($id);  //返回的是1970-1-1 00:00:00 到生成事件的毫秒数
    $node = $donkey->parseNodeId($id); //返回生成这个id的节点号
    $worker_id = $donkey->parseWorkerId($id); //返回生成id的进程号 不过是被处理过的，最多0-31
    $sequence = $donkey->parseSequence($id);  //返回同一时间内生成的序号
    
    echo "donkeyid=".$id."\n";
    echo "time=".date("Y-m-d H:i:s",$time/1000)."\n";
    echo "node=".$node."\n";
    echo "workerid=".$worker_id."\n";
    echo "sequence=".$sequence."\n";
   
```