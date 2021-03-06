有一个脚本，用法比如： `kserverctl.sh restart zk/kfk`

在另一个repo的文档里用到过。

它可以用来简化在systemctl的注册。



觉得突发奇想想到的这个处理二维度选择的办法还挺有趣的。至少比嵌套分支的代码少且明晰，算是利用了软件本身的启动脚本的命名了。

反正软件肯定会给启动脚本一个有特征的命名。

其实这个工具还能以拼接命令参数的思路进一步简化一下。不过其实也不太好，毕竟启动和停止脚本的传参也不一样。只是再用`start`/`stop`去拼接它bin目录下提供的服务启停脚本的话，停止就也得传参，这就不严谨了。

故有总结：当分支中共性以外的部分个分支条件本身可有一一对应，即可把这部分从复杂分支里提取出来变成简单分支；但如果不光有一一对应的部分，就不太必要这样搞了。当然，高级语言里也不是不能在确保严谨的情况下进一步抽象。shell的话……回头试试吧……但作为辅助注册systemctl的工具目前它这样已经很足够了我觉得。

[`kserverctl.sh`](./kserverctl.sh)

```bash
#! /bin/bash

#[ for: kafka-2.6.1 ]#

homepath_var_checker ()
{
    [[ "$1" != "" && -d "$1" ]] ;
} &&

homepath_var_checker "$KAFKA_HOME" ||
{
    rtc=$? ;
    echo '['"$(date +%FT%T.%3N%:::z)"'] ERROR : HOME Dir '"$KAFKA_HOME"' not found !!' >&2 ;
    exit $rtc ;
} ;

#[ make sure value of XXX_HOME is a legal dir ]#


source /etc/profile ||
{
    rtc=$? ;
    echo '['"$(date +%FT%T.%3N%:::z)"'] ERROR : run : source /etc/profile failed !!' >&2 ;
    exit $rtc ;
} ;

#[ make sure /etc/profile can be source ]#


service_type="${2:-kafka}" ;

case "$service_type" in
kafka|kfk) 
    service_name=kafka properties_file_name=server ;;
zookeeper|zk) 
    service_name=zookeeper properties_file_name=$service_name ;;
*) 
    {
        echo '['"$(date +%FT%T.%3N%:::z)"'] ERROR : para2 only can be : kafka/kfk/zookeeper/zk ' >&2 ;
        exit 6 ;
    } ;;
esac ;

#[ make sure para2 is legal , and init the needed vals ]#

#[ then , use values to run shell script ]#

{
    case "$1" in
    start) 
        ( $KAFKA_HOME/bin/${service_name}-server-start.sh $KAFKA_HOME/config/${properties_file_name}.properties & ) ;;
    stop) 
        $KAFKA_HOME/bin/${service_name}-server-stop.sh >&2 ;;
    restart) 
        { bash "$0" stop $service_name ; bash "$0" start $service_name ; } ;;
    *) 
        { echo '['"$(date +%FT%T.%3N%:::z)"'] ERROR : para1 only can be : start/stop/restart ' >&2 ; exit 3 ; } ;;
    esac
} ||
{
    rtc=$? ;
    echo '['"$(date +%FT%T.%3N%:::z)"'] ERROR : something might wrong ... '"($rtc)" >&2 ;
    exit $rtc ;
} ;

# end-of-file
```
