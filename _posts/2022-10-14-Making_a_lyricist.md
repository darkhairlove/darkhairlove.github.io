---
layout: single
title:  "[AIFFEL]_작사가만들기"
categories: AI
tag: [python, aiffel, ai]
toc: true
author_profile: false
---

<head>
  <style>
    table.dataframe {
      white-space: normal;
      width: 100%;
      height: 240px;
      display: block;
      overflow: auto;
      font-family: Arial, sans-serif;
      font-size: 0.9rem;
      line-height: 20px;
      text-align: center;
      border: 0px !important;
    }

    table.dataframe th {
      text-align: center;
      font-weight: bold;
      padding: 8px;
    }

    table.dataframe td {
      text-align: center;
      padding: 8px;
    }

    table.dataframe tr:hover {
      background: #b8d1f3; 
    }

    .output_prompt {
      overflow: auto;
      font-size: 0.9rem;
      line-height: 1.45;
      border-radius: 0.3rem;
      -webkit-overflow-scrolling: touch;
      padding: 0.8rem;
      margin-top: 0;
      margin-bottom: 15px;
      font: 1rem Consolas, "Liberation Mono", Menlo, Courier, monospace;
      color: $code-text-color;
      border: solid 1px $border-color;
      border-radius: 0.3rem;
      word-break: normal;
      white-space: pre;
    }

  .dataframe tbody tr th:only-of-type {
      vertical-align: middle;
  }

  .dataframe tbody tr th {
      vertical-align: top;
  }

  .dataframe thead th {
      text-align: center !important;
      padding: 8px;
  }

  .page__content p {
      margin: 0 0 0px !important;
  }

  .page__content p > strong {
    font-size: 0.8rem !important;
  }
  .language-result div pre {
    background : #ffffff00;
    color : red;
    }
  </style>
</head>



```python
from google.colab import drive
drive.mount('/content/drive')
```

<pre>
Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
</pre>

```python
import re
import glob
import os
import tensorflow as tf

txt_file_path = '/content/drive/MyDrive/Colab/AIFFEL/day/lyrics/*' 

txt_list = glob.glob(txt_file_path) 

raw_corpus = [] 

# 여러개의 txt 파일을 모두 읽어서 raw_corpus 에 담음.
for txt_file in txt_list:
    with open(txt_file, "r") as f:
        raw = f.read().splitlines() 
        raw_corpus.extend(raw)

print("데이터 크기:", len(raw_corpus))
print("Examples:\n", raw_corpus[:3])
```

<pre>
데이터 크기: 187088
Examples:
 ['Looking for some education', 'Made my way into the night', 'All that bullshit conversation']
</pre>

```python
# 입력된 문장을
#     1. 소문자로 바꾸고, 양쪽 공백을 지웁니다
#     2. 특수문자 양쪽에 공백을 넣고
#     3. 여러개의 공백은 하나의 공백으로 바꿉니다
#     4. a-zA-Z?.!,¿가 아닌 모든 문자를 하나의 공백으로 바꿉니다
#     5. 다시 양쪽 공백을 지웁니다
#     6. 문장 시작에는 <start>, 끝에는 <end>를 추가합니다
def preprocess_sentence(sentence):
    sentence = sentence.lower().strip() # 1
    sentence = re.sub(r"([?.!,¿])", r" \1 ", sentence) # 2
    sentence = re.sub(r'[" "]+', " ", sentence) # 3
    sentence = re.sub(r"[^a-zA-Z?.!,¿]+", " ", sentence) # 4
    sentence = sentence.strip() # 5
    sentence = '<start> ' + sentence + ' <end>' # 6
    return sentence
```

## 정제된 문자



```python
corpus = []

for sentence in raw_corpus:
    # 원하지 않는 문장은 건너뜀.
    if len(sentence) == 0 : continue
    if sentence[-1] == ":": continue
    preprocessed_sentence = preprocess_sentence(sentence)
    # 토큰의 개수가 15개를 넘어가는 문장 제외
    if len(preprocessed_sentence.split()) > 15 : continue
    corpus.append(preprocessed_sentence)
corpus[:10]
```
```result
['<start> looking for some education <end>',
 '<start> made my way into the night <end>',
 '<start> all that bullshit conversation <end>',
 '<start> i don t even wanna waste your time <end>',
 '<start> let s just say that maybe <end>',
 '<start> you could help me ease my mind <end>',
 '<start> if that s love in your eyes <end>',
 '<start> it s more than enough <end>',
 '<start> had some bad love <end>',
 '<start> ooh , ooh looking for some affirmation <end>']
```

```python
def tokenize(corpus):
  
    tokenizer = tf.keras.preprocessing.text.Tokenizer(
        num_words=12000, 
        filters=' ',
        oov_token="<unk>"
    )
    tokenizer.fit_on_texts(corpus)
    tensor = tokenizer.texts_to_sequences(corpus)   
    tensor = tf.keras.preprocessing.sequence.pad_sequences(tensor, padding='post')  
    
    print(tensor,tokenizer)
    return tensor, tokenizer

tensor, tokenizer = tokenize(corpus)
print(tensor.shape)
```

<pre>
[[  2 290  28 ...   0   0   0]
 [  2 219  13 ...   0   0   0]
 [  2  25  15 ...   0   0   0]
 ...
 [  2  44  89 ...   0   0   0]
 [  2   4  24 ...   3   0   0]
 [  2  23   9 ...   3   0   0]] <keras_preprocessing.text.Tokenizer object at 0x7f19fb392410>
(156013, 15)
</pre>

```python

src_input = tensor[:, :-1]  
tgt_input = tensor[:, 1:]    

print(src_input[0])
print(tgt_input[0])
```

<pre>
[   2  290   28   94 4486    3    0    0    0    0    0    0    0    0]
[ 290   28   94 4486    3    0    0    0    0    0    0    0    0    0]
</pre>

```python
BUFFER_SIZE = len(src_input)
BATCH_SIZE = 256
steps_per_epoch = len(src_input) // BATCH_SIZE

VOCAB_SIZE = tokenizer.num_words + 1   
```


```python
from sklearn.model_selection import train_test_split

enc_train, enc_val, dec_train, dec_val = train_test_split(src_input, tgt_input, test_size=0.2, random_state=34)

dataset_train = tf.data.Dataset.from_tensor_slices((enc_train, dec_train))
dataset_train = dataset_train.shuffle(BUFFER_SIZE)
dataset_train = dataset_train.batch(BATCH_SIZE, drop_remainder=True)

dataset_val = tf.data.Dataset.from_tensor_slices((enc_val, dec_val))
dataset_val = dataset_val.shuffle(BUFFER_SIZE)
dataset_val = dataset_val.batch(BATCH_SIZE, drop_remainder=True)
```


```python
class TextGenerator(tf.keras.Model):
    def __init__(self, vocab_size, embedding_size, hidden_size):
        super().__init__() 
        self.embedding = tf.keras.layers.Embedding(vocab_size, embedding_size) 
        self.rnn_1 = tf.keras.layers.LSTM(hidden_size, return_sequences=True)  
        self.rnn_2 = tf.keras.layers.LSTM(hidden_size, return_sequences=True)
        self.linear = tf.keras.layers.Dense(vocab_size)
        
    def call(self, x):
        out = self.embedding(x)
        out = self.rnn_1(out)
        out = self.rnn_2(out)
        out = self.linear(out)
        
        return out
  
embedding_size = 512
hidden_size = 2048
model = TextGenerator(tokenizer.num_words + 1, embedding_size , hidden_size) # tokenizer.num_words에 +1인 이유는 문장에 없는 pad가 사용되었기 때문이다.
```


```python
optimizer = tf.keras.optimizers.Adam() 
loss = tf.keras.losses.SparseCategoricalCrossentropy(
    from_logits=True,
    reduction='none' 
)
tf.config.list_physical_devices('GPU')
model.compile(loss=loss, optimizer=optimizer)
model.fit(dataset_train, epochs=10, validation_data=dataset_val, batch_size=256) 
```

<pre>
Epoch 1/10
487/487 [==============================] - 37s 38ms/step - loss: 3.2869 - val_loss: 2.8996
Epoch 2/10
487/487 [==============================] - 18s 37ms/step - loss: 2.7434 - val_loss: 2.6463
Epoch 3/10
487/487 [==============================] - 18s 37ms/step - loss: 2.4323 - val_loss: 2.4659
Epoch 4/10
487/487 [==============================] - 18s 37ms/step - loss: 2.1281 - val_loss: 2.3302
Epoch 5/10
487/487 [==============================] - 18s 37ms/step - loss: 1.8379 - val_loss: 2.2290
Epoch 6/10
487/487 [==============================] - 18s 37ms/step - loss: 1.5790 - val_loss: 2.1656
Epoch 7/10
487/487 [==============================] - 18s 37ms/step - loss: 1.3644 - val_loss: 2.1287
Epoch 8/10
487/487 [==============================] - 18s 37ms/step - loss: 1.2008 - val_loss: 2.1240
Epoch 9/10
487/487 [==============================] - 18s 37ms/step - loss: 1.0897 - val_loss: 2.1351
Epoch 10/10
487/487 [==============================] - 18s 37ms/step - loss: 1.0249 - val_loss: 2.1567
</pre>
<pre>
<keras.callbacks.History at 0x7f19fd39af10>
</pre>
embedding_size = 512 /

hidden_size = 2048

- val_loss : 2.16

- val_loss : 2.15



embedding_size = 64 /

idden_size  = 1026

- val_loss : 2.6




```python
model.summary()
```

<pre>
Model: "text_generator"
_________________________________________________________________
 Layer (type)                Output Shape              Param #   
=================================================================
 embedding (Embedding)       multiple                  6144512   
                                                                 
 lstm (LSTM)                 multiple                  20979712  
                                                                 
 lstm_1 (LSTM)               multiple                  33562624  
                                                                 
 dense (Dense)               multiple                  24590049  
                                                                 
=================================================================
Total params: 85,276,897
Trainable params: 85,276,897
Non-trainable params: 0
_________________________________________________________________
</pre>

```python

def generate_text(model, tokenizer, init_sentence="<start>", max_len=20): #시작 문자열을 init_sentence 로 받으며 디폴트값은 <start> 를 받는다
    # 테스트를 위해서 입력받은 init_sentence도 텐서로 변환합니다
    test_input = tokenizer.texts_to_sequences([init_sentence]) #텍스트 안의 단어들을 숫자의 시퀀스의 형태로 변환
    test_tensor = tf.convert_to_tensor(test_input, dtype=tf.int64)
    end_token = tokenizer.word_index["<end>"]

    
    while True: #루프를 돌면서 init_sentence에 단어를 하나씩 생성성
        predict = model(test_tensor) 
        predict_word = tf.argmax(tf.nn.softmax(predict, axis=-1), axis=-1)[:, -1] 
        test_tensor = tf.concat([test_tensor, tf.expand_dims(predict_word, axis=0)], axis=-1)
        if predict_word.numpy()[0] == end_token: break
        if test_tensor.shape[1] >= max_len: break

    generated = ""
    # tokenizer를 이용해 word index를 단어로 하나씩 변환합니다 
    for word_index in test_tensor[0].numpy():
        generated += tokenizer.index_word[word_index] + " "

    return generated #최종적으로 모델이 생성한 문장을 반환
```


```python
generate_text(model, tokenizer, init_sentence="<start> l love", max_len=20)
# generate_text 함수에 lyricist 라 정의한 모델을 이용해서 ilove 로 시작되는 문장을 생성
```


'<start> l love amour , yeah <end> '

## 회고

- 어려웠던 점 : embedding_size와 hidden_size를 얼마만큼 늘려야 loss를 줄일지가 어려웠다. model.fit의 인자를 batch_size만 추가했다.

- 알아낸 점 및 모호한 점 : model.fit의 shuffle를 추가한 결과랑 아닌 결과랑 차이점이 있는지 궁금하다. 이미 ataset_train.shuffle()을 통해서 model.fit에서 shuffle을 안해도 되는건지 궁금하다. 출력된 문장에 < start >와 < end >, < unk >는 출력을 안하는 방법이 있는지 궁금하다.

- 노력한 점 :  embedding_size와 hidden_size 외에도 다른 model.fit의 인자를 추가하는 방법을 찾기 위해 노력했다. 토큰화 했을 때, 문장 길이가 15개 이하만 train데이터로 하는 과정을 찾아 넣었다.

- 자기다짐 :  embedding_size와 hidden_size을 여러 가지로 해보는 경험이 중요하다고 느꼈다. train과 test 를 split에서 test data의 비율은 고정이고 radomstate를 바꿨을때 val_loss의 값은 크게 차이가 없다는 것을 보았을 때, embedding과 hidden size가 중요하다는 것을 알 수 있었다. 

