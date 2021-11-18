# TREC-CAsT-ASCFDA

This repo is to reproduce the [ASCFDA-CAsT-2020] (https://trec.nist.gov/pubs/trec29/papers/ASCFDA.C.pdf) pipelines.

For the original version of code are most deployed on TPU & GCP with cumbersome T5 source code, so we reimplemented in a 2021-style with the wonderful tools from [castorini lab](https://github.com/castorini), which is one of the best Information Retrieval Labs in the world.

The method propose in [ASCFDA-CAsT-2020] (https://trec.nist.gov/pubs/trec29/papers/ASCFDA.C.pdf) can be separated into 3 parts:

1. T5-CQR: Coreference Query Rewriting by T5
2. SER: Semantic-based Ellipsis Reduction
3. T5-Rerank: passage Rerank by T5

### Installation:
You should install Java 11 first, and follow the instructions of installing:
1. [Anserini](https://github.com/castorini/anserini)
2. [Chatty Goose](https://github.com/castorini/chatty-goose)
3. [PyGaggle](https://github.com/castorini/pygaggle)

Python >= 3.7

### Data Preprocessing:
Download/Construct the corpus first (Ex: [MSMARCO-Passage-Ranking](https://github.com/microsoft/MSMARCO-Passage-Ranking)), preprocess it into the MS MARCO tsv collctions type:
    <pid><\t><text>
or jsonl type that fits the [pyserini](https://github.com/castorini/pyserini):
    {
    "id": "doc1",
    "contents": "this is the contents."
    }

We use the MS MARCO tsv collctions type in test data here, convert it to the jsonl format first:
    python ./anserini/tools/scripts/msmarco/convert_collection_to_jsonl.py \
            --collection-path ./test_data/small_collection.tsv \
            --output-folder ./test_data_jsonl

Then index it with the pyserini:
    python -m pyserini.index -collection JsonCollection \
                         -generator DefaultLuceneDocumentGenerator \
                         -threads 1 \
                         -input ./test_data_jsonl \
                         -index indexes/sample_collection_jsonl \
                         -storePositions -storeDocvectors -storeRaw
The index step will be done multiple times

We finished the indexing here, can starting from each step then.

### T5-CQR

In T5-CQR, we fine-tuned the sequence-to-sequence model T5 with the [CANARD](https://sites.google.com/view/qanta/projects/canard) dataset to rewrite the queries and replace the pronoun with the original subject. This method are reimplemented by [Chatty Goose](https://github.com/castorini/chatty-goose) in their NTR module, we will use their pre-trained version here.
(To fine-tune T5 on your own dataset, please use (T5-huggingface)[https://huggingface.co/transformers/model_doc/t5.html]) to build a new one and reload to the Chatty-Goose settings.

After that retrieve it with the BM25.

# SER:
In this section, we separate the retrieved documents of last part into sentences and use [doc2query](https://github.com/nyu-dl/dl4ir-doc2query) to get the potential query for each sentence.
Here, [docTTTTTquery](https://github.com/castorini/docTTTTTquery) is a better version than doc2query and even easier to use. Therefore, we apply docTTTTTquery here rather than doc2query here.

The potential queries of each sentence are used to measure the closeness between the 
sentence and the CQR-query, while the sentence near CQR-query will be appended to it to add supplement information.

# Rerank:
We rerank the SER-retrieved passages with **CQR-query** and the T5-reranker fine-tuned with [MSMARCO-Passage-Ranking](https://github.com/microsoft/MSMARCO-Passage-Ranking)

# TODO:
evaluation with TREC CAsT 2019






















