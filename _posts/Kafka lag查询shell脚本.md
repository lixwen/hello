#!/bin/bash
filename='processing.topics'
result_filename='result.topics'
server=$1

> $filename

if [[ -z $server ]]
then echo '请输入server地址，例如 sh lag-check.sh 127.0.0.1:9092'
exit 8
fi


if [[ ! -s $result_filename ]] 
then
    echo '首次运行'
    # 查询lag
    bin/kafka-consumer-groups.sh --bootstrap-server $server --all-groups --describe | awk '{if ($1 != "GROUP") print $2, $6}' | awk '{arr[$1]+=$2} END { for (key in arr) {if (arr[key] > 0 && key ~ /topic/) printf("%s\t%s\n", key, arr[key])}}' > $filename

    # 过滤空消息topic
    while read line; do
        # reading each line
        stringarray=($line)
        topic=${stringarray[0]}
        result=`bin/kafka-console-consumer.sh --bootstrap-server $server --topic $topic --from-beginning --max-messages 1 --timeout-ms 1000`
        echo $result
        if [[ $result ]]  
        then echo $line >> $result_filename
        fi
    done < $filename
else
    echo '非首次运行'
    mv $result_filename result.topics_bak
    while read line; do
        # reading each line
        stringarray=($line)
        topic=${stringarray[0]}
        result=`bin/kafka-console-consumer.sh --bootstrap-server $server --topic $topic --from-beginning --max-messages 1 --timeout-ms 1000`
        echo $result
        if [[ $result ]]  
        then echo $line >> $result_filename
        fi
    done < result.topics_bak
fi


echo 'lag检查结束，请查看当前目录下的result.topics文件。其中第一列为topic名称，第二列为未消费消息数量。'
