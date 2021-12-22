# tfmodel_parser


- .pb file is the format of tensorflow model

- .tflite file is the format of tflite model, which usually used in mobile devices

## before started

make sure you have installed these packages or tools

- python 3.*
- tensorflow 2.*
- [flatbuffer](https://github.com/google/flatbuffers)
- [rich](https://github.com/willmcgugan/rich)
- [bidict](https://github.com/jab/bidict)

The best way to test is typing this code on your terminal

- test flatbuffer
```bash
flatc --version 
# if you have installed flatbuffer correctly,you will see 
flatc version 2.0.0
```
- test python packages
```python

python # or whatever version you use

import tensorflow
import rich
import bidict
```

## start

#### read .pb file

```python
    with tf.io.gfile.GFile(file, 'rb') as f:
        graph_def = tf.compat.v1.GraphDef()
        graph_def.ParseFromString(f.read())
        tf.compat.v1.import_graph_def(graph_def)

        for node in graph_def.node:
            res.add(node.op) # get node attr from subgraph
```

#### read .tflite file

##### construct base file

- find .fbs file
    you can get the default fbs file [here](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/schema/schema.fbs)

- transfer fbs to python 

    ```bash
    flatc --python your.fbs
    ```

then you get a folder full of .py, move it to your project ; in this project , you may need the 'tflite' folder

##### using flatbuffer on your code

> example : read op from tflite model

open fbs file , find 'root_type', here is model .

- get model 

```
 with open(file, 'rb') as f:
    buf = f.read()
    model_inner = tflite.Model.Model.GetRootAs(buf, 0)
```

- get subgraph

In the table Model , Subgraphs is a vector containing a few subgraphs . So when we get subgraph, must set the index of Subgraph

```python
subgraph = model_inner.Subgraphs(0) # 0 is the index
```

- get op

Here we need all ops to save, we must traverse the vector of op

```python
op_length = subgraph.OperatorsLength()
for index in range(op_length):
    temp_code = subgraph.Operators(index).OpcodeIndex()
    res.add(dic.inverse[temp_code])

```


