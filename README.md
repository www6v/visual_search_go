![Computing-Thinking](http://people.cs.vt.edu/~kafura/CS6604/Images/CT-Logo.jpg)

A visual search engine(for go) based on Elasticsearch, and A Tensorflow implementation of faster RCNN detection.

![Visual search enging](https://raw.githubusercontent.com/yangboz/visual_search_go/master/Screenshot%20from%202017-11-28%2009-55-37.png)
## Requirements
 There are serveral python2.7 libraries you must to install before building the search engine.
 
 * `elasticsearch==5.2.0`
 * `Tensorflow==0.12.1`
 * `Flask`
 * `opencv-python`
 * `easydict`
 * `cython`
 * `numpy`
 
 * Or
 
 ```
 pip install -r requirements.txt
 ```

* Then

```
cd visual_search/lib & make
```

## Setup
 * Setup Elasticsearch5

 The easiest way to setup is using [Docker](https://www.docker.com/) with [Docker Compose](https://docs.docker.com/compose/). With `docker-compose` everything you have to do is so simple:

 ```bash
 cd visual_search/elasticsearch
 docker-compose up -d
 ```

 * Building elasticsearch plugin

 We need to build Elasticsearch plugin to compute distance between feature vectors.
 Make sure that you have [Maven](https://maven.apache.org/) installed.

 ```bash
 cd visual_search/es-plugin
 mvn install

 cd target/release
 // create simple server to serve plugin
 python -m 'SimpleHTTPServer' &

 //install plugin
 cd ../elasticsearch
 docker exec -it elasticsearch_elasticsearch_1 elasticsearch-plugin install http://localhost:8000/esplugin-0.0.1.zip
 docker-compose restart
 ```

 * Index preparation

 ```bash
 curl -XPUT http://localhost:9200/im_data -d @schema_es.json
 ```
 * Setup faster r-cnn

 I used earlier  `faster r-cnn` version implemented by [@Endernewton](https://github.com/endernewton) for object detection. You can fetch pre-trained model [here](https://drive.google.com/drive/folders/0BzY0S4QyX701OE1BLW5MTldkRVk?usp=sharing). 
## Indexing images to elasticsearch

 ```bash
 export WEIGHT_PATH=...
 export MODEL_PATH=...
 export INPUT=..
 cd visual_search
 python index_es.py --weight $WEIGHT_PATH --model_path $MODEL_PATH --input $INPUT
 ```
### Example

```
sudo python index_es.py --weight ./models/imagenet_weights/vgg16.weights --model_path ./models/faster_rcnn_models/coco_2014_train+coco_2014_valminusminival/default/vgg16_faster_rcnn_iter_490000.ckpt --input ../KGSoutput/2002-01-01-9.png
```

## Start server

 Before starting the server, you must to update `IMGS_PATH` variable in `visual_search/server.py` to the location of folder where images are stored.

 ```bash
 cd visual_search
 python server.py
 ```

 and access the link `http://localhost:5000/static/index.html` to test the search engine.

 Have fun =))

## Issue

```
export CFLAGS="-I /usr/local/lib/python3.5/dist-packages/numpy/core/include $CFLAGS"
```

https://github.com/tensorflow/tensorflow/issues/251

## References

https://blog.algorithmia.com/deep-dive-into-object-detection-with-open-images-using-tensorflow/
