# LowLatencyCloudStreaming

For streaming, RaspberryPi camera with its camera module is used. The streaming test was performed on the laptop with its webcam. Though we can achieve very low latency stream across same router connected 
devices, getting low latency for global streaming is a bit tough.
For this project, gstreamer was used to make the pipelines. GStreamer is a pipeline-based multimedia framework that links together a wide variety of media processing systems to complete complex workflows. 
For instance, GStreamer can be used to build a system that reads files in one format, processes them, and exports them in another. Initially, we had about 3 to 4 seconds delay in the real time and the 
streamed video. With some modifications in the original commands, this lag was cut down to less than 1 second.


# The lag in the stream depends on several factors->

  1) Resolution - Higher the resolution of the streamed video, more will be the processing, thus, more will be the lag.
  2) Framerates - Higher frame rate means more frames are to be encoded, so lag is directly proporational to framerate.
  3) Threading - Processing on multiple threads increases the processing speed, thus lowering the lag.
 
With the use of some advanced pipeline elements like 'speed-preset', 'tune' etc., the lag time was  drastically reduced.


# The terminal command used is ->

./test-launch '( v4l2src device=/dev/video0 ! video/x-raw,width=640,height=480,framerate=15/1,clock-rate=90000 ! videoconvert ! queue ! x264enc bitrate=999999 speed-preset=ultrafast tune=zerolatency 
sliced-threads=true byte-stream=true threads=10 key-int-max=30 intra-refresh=true ! h264parse ! rtph264pay name=pay0 pt=96 )'


# Functions of different elements ->

queue : queue element adds a thread boundary to the pipeline, and support for buffering. The queue creates a new thread on the source pad to decouple the processing on sink and source pad. In the single 
threaded GStreamer pipeline, data starvation may occur. Use the queue element to improve the performance of a single threaded pipeline. Data is queued until one of the limits specified by the 
“max-size-buffers”, “max-size-bytes” and/or “max-size-time” properties has been reached.

threads : for parallelizing processing on multiple cores, for decoupling processing of different pipeline parts, for timers and for blocking operations.

h264parse : it is specifically useful to parse data present in H. 264 encoded videos. H. 264 is a video compression standard and format that uses advanced compression techniques to deliver high-quality 
video at low bit rates

x264enc : This element encodes raw video into H264 compressed data, also otherwise known as MPEG-4 AVC (Advanced Video Codec).
In case of Constant Bitrate Encoding (actually ABR), the "bitrate" will determine the quality of the encoding. This will similarly be the case if this target bitrate is to obtained in multiple (2 or 3)
pass encoding. Alternatively, one may choose to perform Constant Quantizer or Quality encoding, in which case the "quantizer" property controls much of the outcome.

rtph264pay : The rtph264pay element takes in H264 data as input and turns it into RTP packets. RTP is a standard format used to send many types of data over a network, including video.
