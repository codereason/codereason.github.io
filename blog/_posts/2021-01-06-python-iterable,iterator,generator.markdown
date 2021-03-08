---
layout: post
title:  "python中的迭代器与生成器"
date:   2021-01-06
categories: [Jekyll Paper]
tags: [Getting Start]
---

### iterable 
iterable是可迭代对象，可以被用于循环遍历的都是iterable，例如string或者文件,可以放在循环里的 
如 for x in iterable:。实际上这是由iterator做到的，因为iterable 都可以套上iter() 该方法可以返回一个 iterator 用于next迭代。
ITERABLE可以定义一个__getitem__函数用于索引查询

### iterator 
 iterator 由一个iterable的__iter__方法返回，也可以被自己__iter__方法返回(return self)，同时还必须有一个__next__方法用于循环迭代或StopIteration异常，因此迭代器必须要实现 __iter__() 与 __next__() 
 每个iterator 一定是iterable的

### generator

generator 是迭代器的一种。

Python使用生成器对延迟操作提供了支持。所谓延迟操作，是指在需要的时候才产生结果，而不是立即产生结果。这也是生成器的主要好处。
语法上和函数类似：生成器函数和常规函数几乎是一样的。它们都是使用def语句进行定义，差别在于，生成器使用yield语句返回一个值，而常规函数使用return语句返回一个值。

自动实现迭代器协议：对于生成器，Python会自动实现迭代器协议，以便应用到迭代背景中，（如for循环，sum函数）。由于生成器自动实现了迭代器协议，所以，我们可以调用它的next方法，并且，在没有值可以返回的时候，生成器自动产生StopIteration异常

状态挂起：生成器使用yield语句返回一个值。yield语句挂起该生成器函数的状态，保留足够的信息，以便之后从它离开的地方继续执行

优点一：生成器的好处是延迟计算，一次返回一个结果。也就是说，它不会一次生成所有的结果。这对于大数据量处理，将非常有用。

```python
#列表解析
print(sum([i for i in range(100000000)]))#占用内存大，机器容易卡死，甚至可能报MemoryError
#生成器表达式
print(sum(i for i in range(100000000)))#几乎不占内存
```
```python
>>> print(sum([i for i in range(100000000)]))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 1, in <listcomp>
MemoryError

```

```python
s='cat'  #s是一个可迭代对象
t = iter(s)  #t是一个迭代器，由一个next函数与__iter__函数
next(t)  #输出 'c'
next(t)  #输出 'a'
next(t)  #输出 't'
next(t)  #抛出StopIteration
iter(t) is t # iterator必是iterable的

```

关系图

```
 sequence
  +
  |
  v
   def __getitem__(self, index: int):
  +    ...
  |    raise IndexError
  |
  |
  |              def __iter__(self):
  |             +     ...
  |             |     return <iterator>
  |             |
  |             |
  +--> or <-----+        def __next__(self):
       +        |       +    ...
       |        |       |    raise StopIteration
       v        |       |
    iterable    |       |
           +    |       |
           |    |       v
           |    +----> and +-------> iterator
           |                               ^
           v                               |
   iter(<iterable>) +----------------------+
                                           |
   def generator():                        |
  +    yield 1                             |
  |                 generator_expression +-+
  |                                        |
  +-> generator() +-> generator_iterator +-+
```

```python
class MyIterable:

    def __init__(self):
        self.n = 0

    def __getitem__(self, index: int):
        return (1, 2, 3)[index]

    def __next__(self):
        n = self.n = self.n + 1
        if n > 3:
            raise StopIteration
        return n

# if you can iter it without raising a TypeError, then it's an iterable.
iter(MyIterable())

# but obviously `MyIterable()` is not an iterator since it does not have
# an `__iter__` method.
from collections.abc import Iterator
assert isinstance(MyIterable(), Iterator)  # AssertionError
```
生成器中有yield字段 可以用于更高效的迭代
```python
class Iterable1:
    def __iter__(self):
        # a method (which is a function defined inside a class body)
        # calling iter() converts iterable (tuple) to iterator
        return iter((1,2,3))

class Iterable2:
    def __iter__(self):
        # a generator
        for i in (1, 2, 3):
            yield i

class Iterable3:
    def __iter__(self):
        # with PEP 380 syntax
        yield from (1, 2, 3)

# passes
assert list(Iterable1()) == list(Iterable2()) == list(Iterable3()) == [1, 2, 3]
```

### 生成器与机器学习训练

一般在处理数据的时候，比如训练模型，我们经常会用list把全量数据append进去训练，数据量非常大的时候肯定会爆内存，因此我们最好可以生成器进行训练。

例如model.fit(X,y)这种全量训练还是为了书写简便，方便实验用上的，但是keras里也内置了fit_generator这种类似生成器的方法训练。
如：

```python
def generate_arrays_from_file(path):
     while True:
        with open(path) as f:
            for line in f:
     # create numpy arrays of input data
     # and labels, from each line in the file
                x1, x2, y = process_line(line)
                yield ({'input_1': x1, 'input_2': x2},{'output': y})

model.fit_generator(generate_arrays_from_file('./my_folder'),steps_per_epoch=10000, epochs=10)
#接受的 generate_arrays_from_file 就是一个生成器
```

不同于fit()一次性加载所有的train数据集，遍历一遍就可以作为一轮epoch的结束，generator是可以从给定的数据集中“无限”生成数据的，并且因为一次只加载数据集的一部分（generator就是为了解决数据集过大无法一次性加载到内存的问题），所以他并不知道什么时候才是一轮epoch的结束。同样的，batch_size也没有作为参数传递给fit_generator()，所以必须有机制来判断：(1)什么时候结束一轮epoch (2)batch_size是多少。

这时候steps_per_epoch就顺理成章的出现了。这个参数实际上就是指定了每一轮epoch需要执行多少steps，也就是多少steps，才能认为一轮epoch结束。那么衍生问题就是，一个step是怎么度量？其实就是规定每个step加载多少数据，也就是batch_size。他们的关系如下：

steps_per_epoch=len(x_train)/batch_size

一句话概括，就是对于整个训练数据集，generator要在多少步内完成一轮遍历（epoch），从而也就规定了每步要加载多少数据（batch_size）。

再结合kexue.fm上的word2vec训练方法
```python
#! -*- coding:utf-8 -*-
#Keras版的Word2Vec，作者：苏剑林，http://kexue.fm
#Keras 2.0.6 ＋ Tensorflow 测试通过

import numpy as np
from keras.layers import Input,Embedding,Lambda
from keras.models import Model
import keras.backend as K

word_size = 128 #词向量维度
window = 5 #窗口大小
nb_negative = 16 #随机负采样的样本数
min_count = 10 #频数少于min_count的词将会被抛弃
nb_worker = 4 #读取数据的并发数
nb_epoch = 2 #迭代次数，由于使用了adam，迭代次数1～2次效果就相当不错
subsample_t = 1e-5 #词频大于subsample_t的词语，会被降采样，这是提高速度和词向量质量的有效方案
nb_sentence_per_batch = 20
#目前是以句子为单位作为batch，多少个句子作为一个batch（这样才容易估计训练过程中的steps参数，另外注意，样本数是正比于字数的。）

import pymongo
class Sentences: #语料生成器，必须这样写才是可重复使用的
    def __init__(self):
        self.db = pymongo.MongoClient().weixin.text_articles
    def __iter__(self):
        for t in self.db.find(no_cursor_timeout=True).limit(100000):
            yield t['words'] #返回分词后的结果

sentences = Sentences()
words = {} #词频表
nb_sentence = 0 #总句子数
total = 0. #总词频

for d in sentences:
    nb_sentence += 1
    for w in d:
        if w not in words:
            words[w] = 0
        words[w] += 1
        total += 1
    if nb_sentence % 10000 == 0:
        print u'已经找到%s篇文章'%nb_sentence

words = {i:j for i,j in words.items() if j >= min_count} #截断词频
id2word = {i+1:j for i,j in enumerate(words)} #id到词语的映射，0表示UNK
word2id = {j:i for i,j in id2word.items()} #词语到id的映射
nb_word = len(words)+1 #总词数（算上填充符号0）

subsamples = {i:j/total for i,j in words.items() if j/total > subsample_t}
subsamples = {i:subsample_t/j+(subsample_t/j)**0.5 for i,j in subsamples.items()} #这个降采样公式，是按照word2vec的源码来的
subsamples = {word2id[i]:j for i,j in subsamples.items() if j < 1.} #降采样表

def data_generator(): #训练数据生成器
    while True:
        x,y = [],[]
        _ = 0
        for d in sentences:
            d = [0]*window + [word2id[w] for w in d if w in word2id] + [0]*window
            r = np.random.random(len(d))
            for i in range(window, len(d)-window):
                if d[i] in subsamples and r[i] > subsamples[d[i]]: #满足降采样条件的直接跳过
                    continue
                x.append(d[i-window:i]+d[i+1:i+1+window])
                y.append([d[i]])
            _ += 1
            if _ == nb_sentence_per_batch:
                x,y = np.array(x),np.array(y)
                z = np.zeros((len(x), 1))
                yield [x,y],z
                x,y = [],[]
                _ = 0
#CBOW输入
input_words = Input(shape=(window*2,), dtype='int32')
input_vecs = Embedding(nb_word, word_size, name='word2vec')(input_words)
input_vecs_sum = Lambda(lambda x: K.sum(x, axis=1))(input_vecs) #CBOW模型，直接将上下文词向量求和

#构造随机负样本，与目标组成抽样
target_word = Input(shape=(1,), dtype='int32')
negatives = Lambda(lambda x: K.random_uniform((K.shape(x)[0], nb_negative), 0, nb_word, 'int32'))(target_word)
samples = Lambda(lambda x: K.concatenate(x))([target_word,negatives]) #构造抽样，负样本随机抽。负样本也可能抽到正样本，但概率小。

#只在抽样内做Dense和softmax
softmax_weights = Embedding(nb_word, word_size, name='W')(samples)
softmax_biases = Embedding(nb_word, 1, name='b')(samples)
softmax = Lambda(lambda x: 
                    K.softmax((K.batch_dot(x[0], K.expand_dims(x[1],2))+x[2])[:,:,0])
                )([softmax_weights,input_vecs_sum,softmax_biases]) #用Embedding层存参数，用K后端实现矩阵乘法，以此复现Dense层的功能

#留意到，我们构造抽样时，把目标放在了第一位，也就是说，softmax的目标id总是0，这可以从data_generator中的z变量的写法可以看出

model = Model(inputs=[input_words,target_word], outputs=softmax)
model.compile(loss='sparse_categorical_crossentropy', optimizer='adam', metrics=['accuracy'])
#请留意用的是sparse_categorical_crossentropy而不是categorical_crossentropy

model.fit_generator(data_generator(), 
                    steps_per_epoch=nb_sentence/nb_sentence_per_batch, 
                    epochs=nb_epoch,
                    workers=nb_worker,
                    use_multiprocessing=True
                   )

model.save_weights('word2vec.model')

#通过词语相似度，检查我们的词向量是不是靠谱的
embeddings = model.get_weights()[0]
normalized_embeddings = embeddings / (embeddings**2).sum(axis=1).reshape((-1,1))**0.5

```


