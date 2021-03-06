# Hashing for Image Retrieval

Web implement of hashing methods for image retrieval in Python and [DPLM method](http://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=7574359) was applied in this version:
```
    Shen F, Zhou X, Yang Y, et al. A fast optimization method for general binary code learning  
    IEEE Transactions on Image Processing, 2016, 25(12): 5610-5621.
```
In further versions, I will add more methods, especially deep end-to-end framework methods. If you have any question, you can ask me through Issues. 


## Getting started
1. Ensure you have installed Caffe on you platform and you can you can refer to this [document](http://caffe.berkeleyvision.org/installation.html) if not.
2. There are two query methods: memory and database. If you choose memory method, you can ingnore the database configuration and just load data from .pkl file. Configure enviroment variant such as caffe root path, database, query source via configuration.conf.
3. Import the database named irs_sun397.sql in folder DB to your own database or directly use hash binary code and corresponding path saved in .pkl file in folder hashing_model, The details are in [Database](#database) and [PKL file](#pkl-file).  
  **Remark:** database is necessary to large-scale image dataset but .pkl file is convenient to experiment.
4. Download the hashing model from [here](https://pan.baidu.com/s/1jId1Qse)(code is zqye) and place in folder caffe_model.(ONLY DPLM MODEL)
5. Download the SUN397 dataset from [here](http://vision.princeton.edu/projects/2010/SUN/SUN397.tar.gz) and place in static/media/dataset folder.
6. After correcttly placing all files , run app.py by using command below
```shell
    python app.py
```
then browse http://127.0.0.1:5000.



## Dataset and deep feature
The SUN397 dataset VGG16's 'fc7' feature extracted by using deep learning framework Caffe. There are a [page](http://groups.csail.mit.edu/vision/SUN/) where offer details about SUN397 dataset and download [link](http://vision.princeton.edu/projects/2010/SUN/SUN397.tar.gz). The VGG16 pre-trained model can be download from [here](https://gist.github.com/ksimonyan/211839e770f7b538e2d8). The raw features can be download from [here](https://pan.baidu.com/s/1dFMrqq1).(code is b98q)


## Hashing binary code
After some pre-processing(center and normlize), 4096-d feature was transformed into 128-bit hashing binary code by using hashing method. 


## PKL file
There are two pkl files named DPLM128Path.pkl, DPLM128Code.pkl and stored image path and corresponding hashing binary codes. There are all stored in list, but path's element is str and hashing binary code is tuple. You can load the data as follows:
```python
>>> import pickle
>>> code = pickle.load(open('/your/own/path/hashing_model/DPLM128Code.pkl','rb'))
>>> path = pickle.load(open('/your/own/path/hashing_model/DPLM128Path.pkl','rb'))
>>> code[0]
(1249808051, 2609047194, 471897213, 2675925156)
>>> path[0]
'/SUN397/p/picnic_area/sun_bpcvysbqhstfifcb.jpg'
```


## Database
The hashing binary code are stored in the database with mysql. The database is named irs_sun397.sql and the table 'img' construction is bellow: 

| Field  |      Type    | Null  | Key | Default |      Extra     |
| ------ |:-------------| -----:| --- |:-------:|:--------------:|
| id     | int(11)      | NO    | PRI | NULL    | auto_increment |
| path   | varchar(150) | YES   |     | NULL    |                |
| code_1 | bigint(20)   | YES   |     | NULL    |                |
| code_2 | bigint(20)   | YES   |     | NULL    |                |
| code_3 | bigint(20)   | YES   |     | NULL    |                |
| code_4 | bigint(20)   | YES   |     | NULL    |                |

A detailed data is as follows

|id    | path      | code_1     | code_2    | code_3     | code_4  |
|------|-----------|------------|-----------|------------|---------|
| 10000 | /SUN397/m/manufactured_home/sun_bwnvkpvdgnxhkjst.jpg | 4076301309 |917904382 | 1211050903 | 2286664558 |

The database can be found in folder DB and imported to your own database by using command:
```sql
    source irs_sun397.sql
```

A hamming distance function has to be created in order to retrieval simililar images with hashing binary code. 

```sql
CREATE FUNCTION HAMMINGDISTANCE(
  A0 BIGINT, A1 BIGINT, A2 BIGINT, A3 BIGINT, 
  B0 BIGINT, B1 BIGINT, B2 BIGINT, B3 BIGINT
)
RETURNS INT DETERMINISTIC
RETURN 
  BIT_COUNT(A0 ^ B0) +
  BIT_COUNT(A1 ^ B1) +
  BIT_COUNT(A2 ^ B2) +
  BIT_COUNT(A3 ^ B3);
```

