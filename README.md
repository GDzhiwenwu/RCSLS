## RCSLS: PyTorch implementation

RCSLS[[1](https://arxiv.org/abs/1804.07745)] is a method for cross-lingual word embedding alignment, ([originally implemented](https://github.com/facebookresearch/fastText/tree/master/alignment)) in NumPy. This is a PyTorch implementation of the algorithm, based on the [MUSE](https://github.com/facebookresearch/MUSE) codebase. 

## Instructions from [MUSE](https://github.com/facebookresearch/MUSE):

## Dependencies
* Python 2/3 with [NumPy](http://www.numpy.org/)/[SciPy](https://www.scipy.org/)
* [PyTorch](http://pytorch.org/)
* [Faiss](https://github.com/facebookresearch/faiss) (recommended) for fast nearest neighbor search (CPU or GPU).

MUSE is available on CPU or GPU, in Python 2 or 3. Faiss is *optional* for GPU users - though Faiss-GPU will greatly speed up nearest neighbor search - and *highly recommended* for CPU users. Faiss can be installed using "conda install faiss-cpu -c pytorch" or "conda install faiss-gpu -c pytorch".

## Get evaluation datasets
Get monolingual and cross-lingual word embeddings evaluation datasets:
* Our 110 [bilingual dictionaries](https://github.com/facebookresearch/MUSE#ground-truth-bilingual-dictionaries)
* 28 monolingual word similarity tasks for 6 languages, and the English word analogy task
* Cross-lingual word similarity tasks from [SemEval2017](http://alt.qcri.org/semeval2017/task2/)
* Sentence translation retrieval with [Europarl](http://www.statmt.org/europarl/) corpora

by simply running (in data/):

```bash
./get_evaluation.sh
```
*Note: Requires bash 4. The download of Europarl is disabled by default (slow), you can enable it [here](https://github.com/facebookresearch/MUSE/blob/master/data/get_evaluation.sh#L99-L100).*

## Additional instructions 

## Get monolingual word embeddings

```
# English fastText Wikipedia embeddings
wget -c "https://dl.fbaipublicfiles.com/fasttext/vectors-wiki/data/wiki.en.vec" -P data/
wget -c "https://dl.fbaipublicfiles.com/fasttext/vectors-wiki/data/wiki.es.vec" -P data/
```

Replace `en` with other language codes for other languages.

## Align monolingual word embeddings
```
python supervised.py --src_lang en --tgt_lang es--src_emb data/wiki.en.vec --tgt_emb data/wiki.es.vec --seed 2
```

By default, *dico_train* will point to our ground-truth dictionaries (downloaded above); when set to "identical_char" it will use identical character strings between source and target languages to form a vocabulary. Logs and embeddings will be saved in the dumped/ directory.

### Evaluate monolingual or cross-lingual embeddings (CPU|GPU)
MUSE also includes a simple script to evaluate the quality of monolingual or cross-lingual word embeddings on several tasks:

**Monolingual**
```bash
python evaluate.py --src_lang en --src_emb data/wiki.en.vec --max_vocab 200000
```

**Cross-lingual**
```bash
python evaluate.py --src_lang en --tgt_lang es --src_emb data/wiki.en-es.en.vec --tgt_emb data/wiki.en-es.es.vec --max_vocab 200000
```

## Word embedding format
By default, the aligned embeddings are exported to a text format at the end of experiments: `--export txt`. Exporting embeddings to a text file can take a while if you have a lot of embeddings. For a very fast export, you can set `--export pth` to export the embeddings in a PyTorch binary file, or simply disable the export (`--export ""`).

When loading embeddings, the model can load:
* PyTorch binary files previously generated by MUSE (.pth files)
* fastText binary files previously generated by fastText (.bin files)
* text files (text file with one word embedding per line)

The two first options are very fast and can load 1 million embeddings in a few seconds, while loading text files can take a while.

## Results

The reported precision scores are for a fixed number of epochs (10) and with a new unsupervised model selection criterion based on the size of the dictionary induced based on every epoch, based on the current cross-lingual alignment: a bigger dictionary appears to be indicative of a better alignment.

| Code                                                  | en-es | es-en | en-fr | fr-en | en-de | de-en | en-ru | ru-en | en-zh | zh-en |  avg  |
| ----------------------------------------------------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| Joulin et al. [1](https://arxiv.org/abs/1804.07745)   | 84.1  | 86.3  | 83.3  | 84.1  | 79.1  | 76.3  | 57.9  | 67.2  | 45.9  | 46.4  | 71.1  |
| this implementation (10 epochs)                       | 84.2  | 86.6  | 83.9  | 84.7  | 78.3  | 76.6  | 57.6  | 66.7  | 47.6  | 47.4  | 71.4  |
| this implementation (unsup. model selection)          | 84.3  | 86.6  | 83.9  | 85.0  | 78.7  | 76.7  | 57.6  | 67.1  | 47.6  | 47.4  | 71.5  |


[1] A. Joulin, P. Bojanowski, T. Mikolov, H. Jegou, E. Grave, [*Loss in Translation: Learning Bilingual Word Mapping with a Retrieval Criterion*](https://arxiv.org/abs/1804.07745)

```
@InProceedings{joulin2018loss,
    title={Loss in Translation: Learning Bilingual Word Mapping with a Retrieval Criterion},
    author={Joulin, Armand and Bojanowski, Piotr and Mikolov, Tomas and J\'egou, Herv\'e and Grave, Edouard},
    year={2018},
    booktitle={Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing},
}
```


