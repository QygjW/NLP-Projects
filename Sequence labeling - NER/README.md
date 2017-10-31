# Introduction
**Named entity recognition (NER)**, **Part-of-Speeching tagging (POS)** and **Chinese word segmentation** are tasks of **sequence labeling**.  
Without deep learning, statistical **linear-chain conditional random fields (linear-chain CRF)** is the favorable solution. 
But it requires hand-crafted features. In deep learning fields, bi-directional LSTM + CRF gives the state-of-the-art performance, which is 
discussed in detail here.

# Model
### Model architecture
![](https://github.com/gaoisbest/NLP-Projects/blob/master/Sequence%20labeling%20-%20NER/Model_architecture.png)

The above edited image is from [1] shows the model architecture.  
**Embedding layer**: Word embedding (Word emb.) together with character representation (Char rep.) are as inputs to bi-directional LSTM layer.  
**Dropout layer**: it is added before embeddings are input to next layer.  
**Bidirectional LSTM layer**: Then the outputs of both forward LSTM and backward LSTM (which encodes the contextual word representation [2]) are concatenated as inputs to the CRF layer.  
**CRF layer**: And the CRF layer gives the final prediction.  

For embedding layer, **CNN** (to extract morphological information, such as prefix or suffix [3]) or **bi-directional LSTM** [4] can be used to obtain character representation. As to the performance, they have **no significant difference** [1]. See the image from [1] below:  
![](https://github.com/gaoisbest/NLP-Projects/blob/master/Sequence%20labeling%20-%20NER/Character_representation.png)

### Model formula
For an input sequence `X`, which has `n` characters (i.e., ranges from `0` to `n-1`), and it has been padded with the `start` and `end` symbols.  
$$X = (start, x_{0}, x_{1}, x_{2}, ..., x_{n-1}, end)$$  

For the sequence predictions `y`, which also been padded with the `start_tag` and `end_tag`.  
$$y = (start_tag, y_{0}, y_{1}, y_{2}, ..., y_{n-1}, end_tag)$$  

Paper [4] define its score to be  
$$s(X, y) = \sum\limits_{i = -1}^{n} A_{yi, yi+1} + \sum\limits_{i=-1}^{n} P_{i, yi}$$  

`P` (shape of [n+2, k+2], 2 means the padded sequence and tags) is the score matrix output by the bi-directional LSTM. And $P_{i,j}$ is the score of the $j^{th}$ tag of $i^{th}$ character.  
`A` is the tag transition score matrix. And $A_{i,j}$ means the transition score from the tag $i$ to tag $j$.  

The softmax over all possible tag sequences gives the probability for the sequence $y$:  
$$p(y|X) = \frac{e^{s(X, y)}}{\sum\limits_{y_{tmp} \in Y_X} e^{s(X, y_{tmp})}}$$  

The goal is maximizing the log-probability of the correct tag sequence:  
$$\log p(y|X) = s(X,y) - \log\large{(}\sum\limits_{y_{tmp} \in Y_X} e^{s(X, y_{tmp})} \large{)}$$  

This can be solved by Forward-Backward (for computing probability) and Viterbi (for decoding best tags) algorithms.

### Model parameters
- Word embeddings
- Tags transition matrix `A`
- Matrix `P` related parameters: recurrent units of bi-directional LSTM, linear projection weights

However, part of model parameters or hyper-parameters are more important, paper [1] gives a detailed assessment, and the results are:  
- Major impact
  - Pre-trained word embedding
  - Gradient clipping
- Minor impact
  - Number of bi-directional LSTM layer
  - Number of recurrent units, default 100 is preferred
  - Mini-batch size: 8 (for small datasets), 32 (for large datasets)
  - Tagging schema: IOBES

# Implementation
Here, I focus on **brands** NER.  
For example, if the input sentence is 'I bought a Li Ning hat yesterday', then the model will recognize Li Ning as the brand.

# References
[1] [Optimal Hyperparameters for Deep LSTM-Networks for Sequence Labeling Tasks](https://arxiv.org/pdf/1707.06799.pdf) and [implementation](https://github.com/UKPLab/emnlp2017-bilstm-cnn-crf)  
[2] https://guillaumegenthial.github.io/sequence-tagging-with-tensorflow.html  
[3] [End-to-end Sequence Labeling via Bi-directional LSTM-CNNs-CRF](https://arxiv.org/pdf/1603.01354.pdf)  
[4] [Neural Architectures for Named Entity Recognition](https://arxiv.org/pdf/1603.01360.pdf)