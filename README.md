# JapaneseEmbeddingEval

* JSTS/JSICK: Spearman's rank correlation coefficient
   * Cosine similarity was used to calculate the similarity of sentence pairs.
* MIRACL: top30 recall

| Model                                           |   JSTS valid-v1.1 |   JSICK test |   MIRACL dev |   Average |
|:------------------------------------------------|------------------:|-------------:|-------------:|----------:|
| BAAI/bge-m3(dense_vecs)                         |             0.802 |        0.798 |    0.910[^2] |     0.837 |
| MU-Kindai/SBERT-JSNLI-base                      |             0.766 |        0.652 |        0.326 |     0.581 |
| MU-Kindai/SBERT-JSNLI-large                     |             0.774 |        0.677 |        0.278 |     0.576 |
| cl-nagoya/sup-simcse-ja-base                    |             0.809 |        0.827 |        0.527 |     0.721 |
| cl-nagoya/sup-simcse-ja-large                   |             0.831 |        0.831 |        0.507 |     0.723 |
| cl-nagoya/unsup-simcse-ja-base                  |             0.789 |        0.790 |        0.487 |     0.689 |
| cl-nagoya/unsup-simcse-ja-large                 |             0.814 |        0.796 |        0.485 |     0.699 |
| colorfulscoop/sbert-base-ja                     |             0.742 |        0.657 |        0.254 |     0.551 |
| intfloat/multilingual-e5-base                   |             0.796 |        0.806 |    0.845[^2] |     0.816 |
| intfloat/multilingual-e5-large                  |             0.819 |        0.794 |    0.883[^2] |     0.832 |
| intfloat/multilingual-e5-small                  |             0.789 |        0.814 |    0.847[^2] |     0.817 |
| pkshatech/GLuCoSE-base-ja                       |             0.818 |        0.757 |        0.692 |     0.755 |
| pkshatech/simcse-ja-bert-base-clcmlp            |             0.801 |        0.735 |        0.544 |     0.693 |
| oshizo/sbert-jsnli-luke-japanese-base-lite      |             0.811 |        0.726 |        0.497 |     0.678 |
| sonoisa/sentence-bert-base-ja-mean-tokens-v2    |             0.809 |        0.768 |              |           |
| universal-sentence-encoder-multilingual-3       |             0.790 |        0.800 |              |           |
| universal-sentence-encoder-multilingual-large-3 |             0.801 |        0.823 |              |           |
| bclavie/fio-base-japanese-v0.1 [^3]             |             0.863 |        0.894 |        0.718 |     0.825 |
| bclavie/JaColBERT [^4]                          |             N/A   |        N/A   |    0.872[^1] |     N/A   |
| API|
| text-embedding-3-large                          |             0.838 |        0.812 |        0.841[^1] |     0.830 |
| text-embedding-3-small                          |             0.781 |        0.804 |        0.795[^1] |     0.793 |
| text-embedding-ada-002                          |             0.790 |        0.790 |        0.728[^1] |     0.769 |
| textembedding-gecko-multilingual@001            |             0.801 |        0.804 |        0.800[^1] |     0.801 |
| LLM|
| intfloat/e5-mistral-7b-instruct                 |             0.836 |        0.836 |    0.885[^2] |     0.852 |
| oshizo/japanese-e5-mistral-7b_slerp             |             0.846 |        0.842 |    0.886[^2] |     0.858 |

[^1]: Evaluate only the first 100 queries out of 860 queries
[^2]: According to the [model card of multilingual-e5](https://huggingface.co/intfloat/multilingual-e5-large#training-details), the training set of MIRACL is used for fine tuning, so MIRACL is not an unseen task for this model
[^3]: According to the [blog post about fio-base-japanese-v0.1](https://ben.clavie.eu/fio), the tasks aren't unseen by the model, which makes it hard to directly compare with the other models.
[^4]: JaColBERT is a retrieval model. It is optimised only for document retrieval tasks, and not for semantic similarity/entailment tasks like JSTS or JSICK.

## Datasets

* JSTS valid-v1.1
    * https://github.com/yahoojapan/JGLUE
    * 1,457 sentence pairs

* JSICK test
    * https://github.com/verypluming/JSICK
    * 4,927 sentence pairs

* MIRACL dev
    * https://huggingface.co/datasets/miracl/miracl
    * 860 japanese queries
    * From the 6,953,614 japanese data in miracl/miracl-corpus, the sentences to be searched were selected as follows to reduce computation time.
        1. positive passage for each query
        2. 300 hard negatives for each query
        * Hard negative mining was performed using intfloat/multilingual-e5-base
        * Scores for models other than intfloat/multilingual-e5-base are calculated higher only in the following case, but we believe that they are almost unaffected.
            * A negative that is ranked lower than the top 300 by intfloat/multilingual-e5-base is ranked within the top 30 by that model, which pushes the positive into the top 30 or lower.
    * Some queries contain more than 30 potential positive documents in the miracl-corpus. In this case, even a very good model may not be able to rank the ground truth positive documents within the top 30. We estimated such queries to be about 7% to 10% of the total 860 queries. This number was estimated by referring to the tydiqa data for the same query as the corresponding miracl dev query and counting whether the tydiqa answer phrase was in at least 30 of the 300 hard negatives documents.
