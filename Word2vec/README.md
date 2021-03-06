# Example of word representations

```
Supposed corpus:

Document 1: I like drinking water. They also like drinking water.

Document 2: They like drinking coffee.
```
## One-hot vector
Represent every word as |V|*1 vector, with one denotes the word index, and others are all 0.

#### Pros and cons
* Dot product of any two word vectors is zero, therefore, one-hot vector can not tell the similarity.

## Count vector

|          | also |   coffee      |   drinking |       I        |     like      |     They     |     water    |
|   :---:  | :---:|     :---:     |    :---:   |        :---:   |    :---:      |     :---:    |     :---:    |
|Document 1| 1    |      0        |      2     |        1       |      2        |     1        |       2      |
|Document 2| 0    |      1        |      1     |        0       |      1        |      1       |     0        |

#### Pros and cons
* High dimension (#Vocabulary size * #Corpus size) and sparsity
* Ignore the word order 
* Filter low frequency words

## TF-IDF

`TF` = word count /  total word counts in the document. E.g., TF(water, Document 1) = 2/9.

`IDF = log(N/n)`, `N` the number of documents in the corpus, `n` is the number of documents that word appears. E.g., IDF(They) = log(2/2) = 0.


|          | also |   coffee      |   drinking |    I        |     like      |     They     |     water    |
|   :---:  | :---:|     :---:     |    :---:   |     :---:   |    :---:      |     :---:    |     :---:    |
|Document 1| 1/9*log(2/1)| 0      |2/9*log(2/2)=0|1/2*log(2/1)|2/9*log(2/2)=0 |1/9*log(2/2)=0|2/9*log(2/1)  |
|Document 2| 0    |1/4*log(2/1)   |1/4*log(2/1)|     0       | 1/4*log(2/1)  |  1/4*log(2/1)|     0        |



#### Pros and cons
* Penalising common words that appeared in the corpus.

## Co-occurence matrix


`Co-occurrence`: the number of times that two words co-occurence in a context window.

`Context Window`: composed of window size and direction. 

The results of window size **2** and **both** direction:

|          | also |   coffee      |   drinking |   I        |     like      |     They     |     water    |
|   :---:  | :---:|     :---:     |    :---:   |    :---:   |    :---:      |     :---:    |     :---:    |
|   also   |  0   |      0        |      1     |      0     |      1        |     1        |       1      |
|   coffee |  0   |      0        |      1     |      0     |      1        |      0       |     0        |
| drinking |  1   |      1        |      0     |     1      |      3        |      2       |     2        |
|   I      |  0   |      0        |      1     |      0     |      1        |      0       |     0        |
|   like   |  1   |      1        |      3     |     1      |      0        |      2       |     2        |
|   They   |  1   |      0        |      2     |     0      |      2        |      0       |     1        |
|   water  |  1   |      0        |      2     |      0     |      2        |      1       |     0        |

#### Pros and cons
* high dimension
* applying SVD
```
# input texts
# 1. I enjoy flying.
# 2. I like NLP.
# 3. I like deep learning.

%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt

words = ['I', 'like', 'enjoy', 'deep', 'learning', 'NLP', 'flying', '.']

word_cooccur_matrix = np.array(
    [[0, 2, 1, 0, 0, 0, 0, 0],
    [2, 0, 0, 1, 0, 1, 0, 0],
    [1, 0, 0, 0, 0, 0, 1, 0],
    [0, 1, 0, 0, 1, 0, 0, 0],
    [0, 0, 0, 1, 0, 0, 0, 1],
    [0, 1, 0, 0, 0, 0, 0, 1],
    [0, 0, 1, 0, 0, 0, 0, 1],
    [0, 0, 0, 0, 1, 1, 1, 0]])

U, s, V = np.linalg.svd(word_cooccur_matrix)

for i in range(len(words)):
    plt.xlim((-1, 1))
    plt.ylim((-1,1))
    plt.text(U[i, 0], U[i, 1], words[i])
```

![applying SVD](https://github.com/gaoisbest/NLP-Projects/blob/master/Word2vec/svd.png)


## Word2vec
**The word orders are ignored in each window.**

### Distributional hypothesis
**Similar words have similar context.**

### Subsampling
Frequent words (such as the, and) are [subsampling](http://mccormickml.com/2017/01/11/word2vec-tutorial-part-2-negative-sampling/) .
For each word in one sentence, it can be deleted or not according its frequency. And the hyper-parameter sampling rate (i.e., `sample` in [gensim](https://radimrehurek.com/gensim/models/word2vec.html) `Word2Vec`, default value is `1e-3`)


### Negative Sampling
Negative samples are selected proportional to its frequency (`f(w)^3/4`).


### [Comparisons of CBOW and SG or HS and NS](https://code.google.com/archive/p/word2vec/) :
* skip-gram (slower, better for infrequent words) vs CBOW (fast)
* CBOW smoothes over a lot of the distributional information (by treating an entire context as one observation), useful for **smaller datasets**. Skip-gram treats each context-target pair as a new observation, and tends to do better when **larger datasets**.
* hierarchical softmax (better for infrequent words) vs negative sampling (better for frequent words, better with low dimensional vectors), [why](https://stats.stackexchange.com/questions/180076/why-is-hierarchical-softmax-better-for-infrequent-words-while-negative-sampling)?
* sub-sampling of frequent words: can improve both accuracy and speed for large data sets (useful values are in range 1e-3 to 1e-5), [why](https://www.quora.com/How-does-sub-sampling-of-frequent-words-work-in-the-context-of-Word2Vec)?
* dimensionality of the word vectors: usually more is better, but not always
* context (window) size: for skip-gram usually around 10, for CBOW around 5


### Implementation

#### gensim and fastText
I did a very simple performance comparsion of [gensim](https://radimrehurek.com/gensim/models/word2vec.html) and [fastText](https://github.com/facebookresearch/fastText) based on the same hyper-parameters, the results of gensim seems little better than that of fastText (without n-gram).

`gensim_Word2vec_cbow_hs_model_test.py`: gensim based on cbow model and hierarchical softmax trick.

`gensim_Word2vec_sg_ns_model_test.py`: gensim based on skipgram model and negative sampling trick.

`fastText_Word2vec_cbow_hs_model_test.bash`: fastText based on cbow model and hierarchical softmax trick.

`fastText_Word2vec_sg_ns_model_test.bash`: fastText based on skipgram and negative sampling trick.

The input of the about sripts is [word_vector_108000.cs](https://pan.baidu.com/s/1i5Jy4lv), which contains 108,000 documents (15,000 training data and 3,000 testing data for each category, and 6 category in total).

#### tensorflow
I also implemented the [Word2vec in tensorflow](https://github.com/gaoisbest/NLP-Projects/blob/master/Word2vec/tf_2_word2vec.ipynb) based on CS20SI. The following image show the results, it looks good enough.

![ ](https://github.com/gaoisbest/NLP-Projects/blob/master/Word2vec/tf_word2vec.png)

### Visualization tool: https://ronxin.github.io/wevi/


Reference:

http://web.stanford.edu/class/cs224n/lecture_notes/cs224n-2017-notes1.pdf
