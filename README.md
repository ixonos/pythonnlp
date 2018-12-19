# pythonnlp
Python tools for Finnish NLP

## Tools:
 - Turku neural parser (mainly for Finnish) (https://turkunlp.github.io/Turku-neural-parser-pipeline/ and https://github.com/TurkuNLP/Turku-neural-parser-pipeline)
 - FinnPos parser (with omorfi and CRF) (https://github.com/mpsilfve/FinnPos)
 - Wit interface (https://wit.ai)
 - Wordmesh (https://github.com/mukund109/word-mesh)
 - Gensim

## Building:

```
cd build
docker build -t pythonnlp .
```

## Word embeddings from Facebook / fasttext

See https://github.com/facebookresearch/fastText/blob/master/pretrained-vectors.md

It is recommended that you download the fasttext vectors offline and then mount them in the docker commandline, see example below:

```
mkdir fasttext
cd fasttext
curl -O https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.fi.vec
curl -O https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.sv.vec
curl -O https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.en.vec

docker run -t -i -v `pwd`/fasttext:/fasttext -v pythonnlp
```

To extract the vectors from the text file, use gensim in python:

```
python3
from gensim.models import KeyedVectors
fi_model = KeyedVectors.load_word2vec_format('/fasttext/wiki.fi.vec')
fi_vectors = fi_model.wv
fi_vectors.save("/fasttext/gensim_fi")
```

Once you have built it:

```
from gensim.models import KeyedVectors
fi_vectors = KeyedVectors.load("/fasttext/gensim_fi", mmap='r')

fi_vectors.most_similar("aurinko")
fi_vectors.most_similar("kesä")
fi_vectors.doesnt_match("hyvä paha auto hieno".split())
fi_vectors.distance("auto", "juna")
```

These embeddings have been built with sub-word information and there is no pre-processing (cleaning up, lemmatization, etc), so the "most similar" method mostly returns inflections of the same word. The vectors should still work fine if you use lemmatized forms (i.e. base word).

## Turku NLP

The neural dependency parser works best with GPUs, which isn't possible when using Docker with a Mac. Using the parser as shown below is really slow. The server mode would be better, but it doesn't seem to work without GPU mode.

```
cd Turku-neural-parser-pipeline
cat mytext.txt | python3 full_pipeline_stream.py --gpu -1 --conf models_fi_tdt/pipelines.yaml --pipeline parse_plaintext > /tmp/myfile.conllu
```

## FinnPos

Example

```
cat mytext.txt | tr ' ' '\n' | perl -pe 's/\.$/\n.\n/' | ftb-label
```

