SRC=GSOD 
DST=

# http://blog.csdn.net/mrcharles/article/details/50442367

# Prepare script
## generate list
```
echo "create input list"
echo '
#!/bin/bash  
a=$1  
rm ncdc_files.txt  
hdfs dfs -rm /ncdc_files.txt  
  
while [ $a -le $2 ]  
do  
        filename="/GSOD/${a}/gsod_${a}.tar"   
        echo -e "$filename" >>ncdc_files.txt  
        a=`expr $a + 1`  
done  

hdfs dfs -put ncdc_files.txt /  
'> generate_input_list.sh
chmod +x generate_input_list.sh
```

## file merge
```
echo "create input list"
echo '
#!/bin/bash  
read hdfs_file  
echo "$hdfs_file"  
# Retrieve file from HDFS to local disk  
echo "reporter:status:Retrieving $hdfs_file" >&2   
hdfs dfs -get $hdfs_file .  
# Create local directory  
target=`basename $hdfs_file .tar`  
mkdir $target  
  
echo "reporter:status:Un-tarring $hdfs_file to $target" >&2   
tar xf `basename $hdfs_file` -C $target  
# Unzip each station file and concat into one file  
echo "reporter:status:Un-gzipping $target" >&2  
for file in $target/*  
do  
        gunzip -c $file >> $target.all  
        echo "reporter:status:Processed $file" >&2  
done  
# Put gzipped version into HDFS  
echo "reporter:status:Gzipping $target and putting in HDFS" >&2   
hdfs dfs -rm -r /GSOD_ALL/$target.gz 
gzip -c $target.all | hdfs dfs -put - /GSOD_ALL/$target.gz  
rm `basename $hdfs_file`  
rm -r $target  
rm $target.all  
'> load_ncdc_map.sh
chmod +x load_ncdc_map.sh  
```

# 

## copy to hdfs
```
hdfs dfs -mkdir /GSOD /GSOD_ALL  
hdfs dfs -ls / 
hdfs dfs -put ~/source/data/gsod/19* /GSOD/ 
hdfs fsck /GSOD -files -blocks -racks 
```

## generate list
```
./generate_input_list.sh 1950 1960
more ncdc_files.txt  
```

## merge each year
```
#cat ncdc_files.txt | ./load_ncdc_map.sh   
cat ncdc_files.txt | while read line 
do
    echo $line | ./load_ncdc_map.sh
done
hdfs dfs -ls /GSOD_ALL  
```

# Stream
```
hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-2.7.3.jar \
    -D mapreduce.job.reduces=0 \
    -D mapreduce.map.speculative=false \
    -D mapreduce.task.timeout=12000000 \
    -inputformat org.apache.hadoop.mapred.lib.NLineInputFormat \
    -input /ncdc_files.txt \
    -output /output/1 \
    -mapper load_ncdc_map.sh \
    -file load_ncdc_map.sh  
```

hdfs dfsadmin -safemode leave
