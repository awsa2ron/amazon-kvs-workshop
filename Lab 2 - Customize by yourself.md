# Lab 2 - Customize

## reduce buffer size

Default storage size(mostly video buffer) is 128MB, which is should be able to buffer up to 120s video data. But embeded device usually don't have enough RAM. So let's change it to reduce RAM consumption.

1. Double click on amazon-kinesis-video-streams-producer-c/samples/KvsAacAudioVideoStreamingSample.c 
2. add one line after line 222:
```
pDeviceInfo->storageInfo.storageSize = 3 * 1024 * 1024; 
```
3. Save, make and test.
4. Checking
After setting `storageSize` to 3MB, we can finger out the RAM comsuption like this:

![ram consumption](images/kvs-ram-openSSL.png)

VmRSS is the RAM consumption, it almost 18MB on the Ubuntu18.04 now. 

> PS: You can reduce the `storageSize` down to 1MB, but we don't recommand, because  the application will too sensitive to the network connection quality if you do so. Setting `storageSize` to 3-5MB on resource limited device is recommanded.


## change to mbedTLS

Forthermore, we can reduce the RAM consumption by change the default OpenSSL to MbedTLS.


```
# Please delete previous project to avoid pertencial issue.
git clone --recursive https://github.com/awslabs/amazon-kinesis-video-streams-producer-c.git
cd amazon-kinesis-video-streams-producer-c/
mkdir build
cd build/ 
cmake .. -DUSE_OPENSSL=OFF -DUSE_MBEDTLS=ON
make
export LD_LIBRARY_PATH=~/environment/amazon-kinesis-video-streams-producer-c/open-source/lib:$LD_LIBRARY_PATH
./kvsAacAudioVideoStreamingSample my-kvs-stream 6000
```

*TBD, because source code bug*



### Deep dive into KVS producer

There are some usefull information, AKA Metrics in KVS, like `Currently available storage size in bytes`, need to deep dive into code. We can leverage it to handle network traffic jam.

1. We change to another sample, double click on amazon-kinesis-video-streams-producer-c/samples/KvsVideoOnlyStreamingSample.c 
2. add one line after line 60:
```
    ClientMetrics kinesisVideoClientMetrics;
    kinesisVideoClientMetrics.version = CLIENT_METRICS_CURRENT_VERSION;
```
![metrics1](images/kvs-metrics-1.png)

3. add one line after line 137:
```
    CHK_STATUS(getKinesisVideoMetrics(clientHandle, &kinesisVideoClientMetrics));
    printf("KVS video buffer size:%d KB, Available:%d KB\n", \
            kinesisVideoClientMetrics.contentStoreSize >> 10, \
            kinesisVideoClientMetrics.contentStoreAvailableSize >> 10 \
    );
```
![metrics2](images/kvs-metrics-2.png)


4. Save, make and test.
```
make
./kvsVideoOnlyStreamingSample your-kvs-name 600 ../samples/h264SampleFrames/
```

5. Checking

![metrics-result](images/kvs-metrics-result.png)

The `available storage size` is very important when the SDK encounters a stream latency condition. user application need to check this parameter frequently and reduce video bit rate or frame per second under such condition. 

### IoT certification

*TBD*



## Done


You are done with the Customize and are ready to move to [Lab 3]