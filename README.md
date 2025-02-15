### <div align="center">⚡FlashRAG: A Python Toolkit for Efficient RAG Research<div>


<div align="center">
<a href="https://arxiv.org/abs/2405.13576"target="_blank"><img src=https://img.shields.io/badge/arXiv-b5212f.svg?logo=arxiv></a>
<a href="https://huggingface.co/datasets/ignore/FlashRAG_datasets" target="_blank"><img src=https://img.shields.io/badge/%F0%9F%A4%97%20HuggingFace%20Datasets-27b3b4.svg></a>
<a href="https://github.com/RUC-NLPIR/FlashRAG/blob/main/LICENSE">
<img alt="License" src="https://img.shields.io/badge/LICENSE-MIT-green">
</a>
</div>

<h4 align="center">

<p>
<a href="#wrench-installation">Installation</a> |
<a href="#sparkles-features">Features</a> |
<a href="#running-quick-start">Quick-Start</a> |
<a href="#gear-components"> Components</a> |
<a href="#robot-supporting-methods"> Supporting Methods</a> |
<a href="#notebook-supporting-datasets"> Supporting Datasets</a> |
<a href="#raised_hands-additional-faqs"> FAQs</a>
</p>


</h4>

FlashRAG is a Python toolkit for the reproduction and development of Retrieval Augmented Generation (RAG) research. Our toolkit includes 32 pre-processed benchmark RAG datasets and 12 state-of-the-art RAG algorithms. 

<p align="center">
<img src="asset/framework.png">
</p>

With FlashRAG and provided resources, you can effortless reproduce existing SOTA works in the RAG domain or implement your custom RAG processes and components.



## :sparkles: Features

- **🛠 Extensive and Customizable Framework**: Includes essential components for RAG scenarios such as retrievers, rerankers, generators, and compressors, allowing for flexible assembly of complex pipelines.

- **🗂 Comprehensive Benchmark Datasets**: A collection of 32 pre-processed RAG benchmark datasets to test and validate RAG models' performances.

- **🎯 Pre-implemented Advanced RAG Algorithms**: Features 12 advancing RAG algorithms with reported results, based on our framework. Easily reproducing results under different settings.

- **🧩 Efficient Preprocessing Stage**: Simplifies the RAG workflow preparation by providing various scripts like corpus processing for retrieval, retrieval index building, and pre-retrieval of documents.

- **🚀 Optimized Execution**: The library's efficiency is enhanced with tools like vLLM, FastChat for LLM inference acceleration, and Faiss for vector index management.

## :mag_right: Roadmap

FlashRAG is still under development and there are many issues and room for improvement. We will continue to update. And we also sincerely welcome contributions on this open-source toolkit.

- [ ] Support multiple API based generators (e.g. ChatGPT, Claude, Gemini)
- [ ] Integrating sentence-transformers
- [ ] Add more evaluation metrics (e.g. Unieval, name-entity F1) and benchmarks (e.g. RGB benchmark)
- [ ] Enhance code adaptability and readability


## :wrench: Installation 

To get started with FlashRAG, simply clone it from Github and install (requires Python 3.9+): 

```bash
git clone https://github.com/RUC-NLPIR/FlashRAG.git
cd FlashRAG
pip install -e . 
```

## :running: Quick Start


#### Toy Example

Run the following code to implement a naive RAG pipeline using provided toy datasets. 
The default retriever is ```e5``` and default generator is ```llama2-7B-chat```. You need to fill in the corresponding model path in the following command. If you wish to use other models, please refer to the detailed instructions below.

```bash
cd examples/quick_start
python simple_pipeline.py \
    --model_path=<LLAMA2-7B-Chat-PATH> \
    --retriever_path=<E5-PATH>
```

After the code is completed, you can view the intermediate results of the run and the final evaluation score in the output folder under the corresponding path.

**Note:** This toy example is just to help test whether the entire process can run normally. Our toy retrieval document only contains 1000 pieces of data, so it may not yield good results.

#### Using the ready-made pipeline

You can use the pipeline class we have already built (as shown in [pipelines](#pipelines)) to implement the RAG process inside. In this case, you just need to configure the config and load the corresponding pipeline.

Firstly, load the entire process's config, which records various hyperparameters required in the RAG process. You can input yaml files as parameters or directly as variables. The priority of variables as input is higher than that of files.

```python
from flashrag.config import Config

config_dict = {'data_dir': 'dataset/'}
my_config = Config(config_file_path = 'my_config.yaml',
                config_dict = config_dict)
```
You can refer to the [basic yaml file](./flashrag/config/basic_config.yaml) we provide to set your own parameters. For specific parameter names and meanings, please refer to the [config parameter description](./flashrag/config/basic_config.yaml).

Next, load the corresponding dataset and initialize the pipeline. The components in the pipeline will be automatically loaded. 

```python
from flashrag.utils import get_dataset
from flashrag.pipeline import SequentialPipeline
from flashrag.prompt import PromptTemplate
from flashrag.config import Config

config_dict = {'data_dir': 'dataset/'}
my_config = Config(config_file_path = 'my_config.yaml',
                config_dict = config_dict)
all_split = get_dataset(my_config)
test_data = all_split['test']

pipeline = SequentialPipeline(my_config)
```

You can specify your own input prompt using `PromptTemplete`:
```python
prompt_templete = PromptTemplate(
    config, 
    system_prompt = "Answer the question based on the given document. Only give me the answer and do not output any other words.\nThe following are given documents.\n\n{reference}",
    user_prompt = "Question: {question}\nAnswer:"
)
pipeline = SequentialPipeline(my_config, prompt_template=prompt_templete)
```

Finally, execute `pipeline.run` to obtain the final result.

```python
output_dataset = pipeline.run(test_data, do_eval=True)
```
The `output_dataset` contains the intermediate results and metric scores for each item in the input dataset.
Meanwhile, the dataset with intermediate results and the overall evaluation score will also be saved as a file (if `save_intermediate_data` and `save_metric_score` are specified).

#### Build your own pipeline

Sometimes you may need to implement more complex RAG process, and you can build your own pipeline to implement it.
You just need to inherit `BasicPipeline`, initialize the components you need, and complete the `run` function.

```python
from flashrag.pipeline import BasicPipeline
from flashrag.utils import get_retriever, get_generator

class ToyPipeline(BasicPipeline):
  def __init__(self, config, prompt_templete=None):
    # Load your own components
    pass

  def run(self, dataset, do_eval=True):
    # Complete your own process logic

    # get attribute in dataset using `.`
    input_query = dataset.question
    ...
    # use `update_output` to save intermeidate data
    dataset.update_output("pred",pred_answer_list)
    dataset = self.evaluate(dataset, do_eval=do_eval)
    return dataset
```

Please first understand the input and output forms of the components you need to use from our [documentation](./docs/basic_usage.md).


#### Just use components

If you already have your own code and only want to use our components to embed the original code, you can refer to the [basic introduction of the components](./docs/basic_usage.md) to obtain the input and output formats of each component.

## :gear: Components

In FlashRAG, we have built a series of common RAG components, including retrievers, generators, refiners, and more. Based on these components, we have assembled several pipelines to implement the RAG workflow, while also providing the flexibility to combine these components in custom arrangements to create your own pipeline.

#### RAG-Components

<table>
  <thead>
    <tr>
      <th>Type</th>
      <th>Module</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="1">Judger</td>
      <td>SKR Judger</td>
      <td>Judging whether to retrieve using <a href="https://aclanthology.org/2023.findings-emnlp.691.pdf">SKR</a> method</td>
    </tr>
    <tr>
      <td rowspan="4">Retriever</td>
      <td>Dense Retriever</td>
      <td>Bi-encoder models such as dpr, bge, e5, using faiss for search</td>
    </tr>
    <tr>
      <td>BM25 Retriever</td>
      <td>Sparse retrieval method based on Lucene</td>
    </tr>
    <tr>
      <td>Bi-Encoder Reranker</td>
      <td>Calculate matching score using bi-Encoder</td>
    </tr>
    <tr>
      <td>Cross-Encoder Reranker</td>
      <td>Calculate matching score using cross-encoder</td>
    </tr>
    <tr>
      <td rowspan="4">Refiner</td>
      <td>Extractive Refiner</td>
      <td>Refine input by extracting important context</td>
    </tr>
    <tr>
      <td>Abstractive Refiner</td>
      <td>Refine input through seq2seq model</td>
    </tr>
    <tr>
      <td>LLMLingua Refiner</td>
      <td><a href="https://aclanthology.org/2023.emnlp-main.825/">LLMLingua-series</a> prompt compressor</td>
    </tr>
    <tr>
      <td>SelectiveContext Refiner</td>
      <td><a href="https://arxiv.org/abs/2310.06201">Selective-Context</a> prompt compressor</td>
    </tr>
    <tr>
      <td rowspan="4">Generator</td>
      <td>Encoder-Decoder Generator</td>
      <td>Encoder-Decoder model, supporting <a href="https://arxiv.org/abs/2007.01282">Fusion-in-Decoder (FiD)</a></td>
    </tr>
    <tr>
      <td>Decoder-only Generator</td>
      <td>Native transformers implementation</td>
    </tr>
    <tr>
      <td>FastChat Generator</td>
      <td>Accelerate with <a href="https://github.com/lm-sys/FastChat">FastChat</a></td>
    </tr>
    <tr>
      <td>vllm Generator</td>
      <td>Accelerate with <a href="https://github.com/vllm-project/vllm">vllm</a></td>
    </tr>
  </tbody>
</table>

#### Pipelines

Referring to a [survey on retrieval-augmented generation](https://arxiv.org/abs/2312.10997), we categorized RAG methods into four types based on their inference paths.

- **Sequential**: Sequential execuation of RAG process, like Query-(pre-retrieval)-retriever-(post-retrieval)-generator
- **Conditional**: Implements different paths for different types of input queries
- **Branching** : Executes multiple paths in parallel, merging the responses from each path
- **Loop**: Iteratively performs retrieval and generation

In each category, we have implemented corresponding common pipelines. Some pipelines have corresponding work papers.

<table>
    <thead>
        <tr>
            <th>Type</th>
            <th>Module</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="1">Sequential</td>
            <td>Sequential Pipeline</td>
            <td>Linear execution of query, supporting refiner, reranker</td>
        </tr>
        <tr>
            <td rowspan="1">Conditional</td>
            <td>Conditional Pipeline</td>
            <td>With a judger module, distinct execution paths for various query types</td>
        </tr>
        <tr>
            <td rowspan="2">Branching</td>
            <td>REPLUG Pipeline</td>
            <td>Generate answer by integrating probabilities in multiple generation paths</td>
        </tr>
          <td>SuRe Pipeline</td>
          <td>Ranking and merging generated results based on each document</td>
        </tr>
        <tr>
            <td rowspan="4">Loop</td>
            <td>Iterative Pipeline</td>
            <td>Alternating retrieval and generation</td>
        </tr>
        <tr>
            <td>Self-Ask Pipeline</td>
            <td>Decompose complex problems into subproblems using <a href="https://arxiv.org/abs/2210.03350">self-ask</a> </td>
        </tr>
        <tr>
            <td>Self-RAG Pipeline</td>
            <td>Adaptive retrieval, critique, and generation</td>
        </tr>
        <tr>
            <td>FLARE Pipeline</td>
            <td>Dynamic retrieval during the generation process</td>
        </tr>
    </tbody>
</table>


## :robot: Supporting Methods

We have implemented 12 works with a consistent setting of:
- **Generator:** LLAMA3-8B-instruct with input length of 4096
- **Retriever:** e5-base-v2 as embedding model, retrieve 5 docs per query
- **Prompt:** A consistent default prompt, templete can be found in the [code](./flashrag/prompt/base_prompt.py).

For open-source methods, we implemented their processes using our framework. For methods where the author did not provide source code, we will try our best to follow the methods in the original paper for implementation.

For necessary settings and hyperparameters specific to some methods, we have documented them in the **specific settings** column. For more details, please consult our [code](./examples/methods/run_exp.py).

It’s important to note that, to ensure consistency, we have utilized a uniform setting. However, this setting may differ from the original setting of the method, leading to variations in results compared to the original outcomes.


| Method               | Type           | NQ (EM) | TriviaQA (EM) | Hotpotqa (F1) | 2Wiki (F1)| PopQA (F1)| WebQA(EM) | Specific setting                                                                  |
|----------------------|----------------|---------|---------------|---------------|---------------|---------------|---------------|------------------------------------------------------------------------------------|
| Naive Generation     | Sequential     | 22.6    | 55.7          | 28.4          |  33.9| 21.7| 18.8| |
| Standard RAG         | Sequential     | 35.1    | 58.9          | 35.3          | 21.0 | 36.7|15.7| |
| [AAR-contriever-kilt](https://aclanthology.org/2023.acl-long.136.pdf)  | Sequential     | 30.1    | 56.8          | 33.4          | 19.8 | 36.1  | 16.1| |
| [LongLLMLingua](https://aclanthology.org/2023.acl-long.136.pdf)        | Sequential     | 32.2    | 59.2          | 37.5          |25.0| 38.7| 17.5| Compress Ratio=0.5 |
| [RECOMP-abstractive](https://aclanthology.org/2023.acl-long.136.pdf)   | Sequential     | 33.1    | 56.4          | 37.5          | 32.4 | 39.9| 20.2| |
| [Selective-Context](https://arxiv.org/abs/2310.06201)    | Sequential     | 30.5    | 55.6          | 34.4          |18.5| 33.5| 17.3| Compress Ratio=0.5|
| [Ret-Robust](https://arxiv.org/abs/2310.01558)           | Sequential     | 42.9    | 68.2          | 35.8          |43.4|57.2|33.7| Use LLAMA2-13B with trained lora|
| [SuRe](https://arxiv.org/abs/2404.13081)                 | Branching      | 37.1    | 53.2          | 33.4          |20.6|48.1|24.2| Use provided prompt|
| [REPLUG](https://arxiv.org/abs/2301.12652)               | Branching      | 28.9    | 57.7          | 31.2          |21.1|27.8|20.2|  |
| [SKR](https://aclanthology.org/2023.findings-emnlp.691.pdf)                  | Conditional    | 25.5    | 55.9          | 29.8          | 28.5|24.5|18.6|Use infernece-time training data|
| [Self-RAG](https://arxiv.org/abs/2310.11511)             | Loop   | 36.4    | 38.2          | 29.6          | 25.1|32.7|21.9| Use trained selfrag-llama2-7B|
| [FLARE](https://arxiv.org/abs/2305.06983)                | Loop   | 22.5    | 55.8          | 28.0          |33.9| 20.7| 20.2| |
| [Iter-Retgen](https://arxiv.org/abs/2305.15294),      [ITRG](https://arxiv.org/abs/2310.05149)   | Loop | 36.8    | 60.1          | 38.3          | 21.6| 37.9| 18.2| |




## :notebook: Supporting Datasets & Document Corpus

### Datasets

We have collected and processed 35 datasets widely used in RAG research, pre-processing them to ensure a consistent format for ease of use. For certain datasets (such as Wiki-asp), we have adapted them to fit the requirements of RAG tasks according to the methods commonly used within the community. All datasets are available at [Huggingface datasets](https://huggingface.co/datasets/ignore/FlashRAG_datasets). 

For each dataset, we save each split as a `jsonl` file, and each line is a dict as follows:
```python
{
  'id': str,
  'question': str,
  'golden_answers': List[str],
  'metadata': dict
}
```


Below is the list of datasets along with the corresponding sample sizes:

| Task                      | Dataset Name    | Knowledge Source | # Train   | # Dev   | # Test |
|---------------------------|-----------------|------------------|-----------|---------|--------|
| QA                        | NQ              | wiki             | 79,168    | 8,757   | 3,610  |
| QA                        | TriviaQA        | wiki & web       | 78,785    | 8,837   | 11,313 |
| QA                        | PopQA           | wiki             | /         | /       | 14,267 |
| QA                        | SQuAD           | wiki             | 87,599    | 10,570  | /      |
| QA                        | MSMARCO-QA      | web              | 808,731   | 101,093 | /      |
| QA                        | NarrativeQA     | books and story  | 32,747    | 3,461   | 10,557 |
| QA                        | WikiQA          | wiki             | 20,360    | 2,733   | 6,165  |
| QA                        | WebQuestions    | Google Freebase  | 3,778     | /       | 2,032  |
| QA                        | AmbigQA         | wiki             | 10,036    | 2,002   | /      |
| QA                        | SIQA            | -                | 33,410    | 1,954   | /      |
| QA                        | CommenseQA      | -                | 9,741     | 1,221   | /      |
| QA                        | BoolQ           | wiki             | 9,427     | 3,270   | /      |
| QA                        | PIQA            | -                | 16,113    | 1,838   | /      |
| QA                        | Fermi           | wiki             | 8,000     | 1,000   | 1,000  |
| multi-hop QA              | HotpotQA        | wiki             | 90,447    | 7,405   | /      |
| multi-hop QA              | 2WikiMultiHopQA | wiki             | 15,000    | 12,576  | /      |
| multi-hop QA              | Musique         | wiki             | 19,938    | 2,417   | /      |
| multi-hop QA              | Bamboogle       | wiki             | /         | /       | 125    |
| Long-form QA              | ASQA            | wiki             | 4,353     | 948     | /      |
| Long-form QA              | ELI5            | Reddit           | 272,634   | 1,507   | /      |
| Open-Domain Summarization | WikiASP         | wiki             | 300,636   | 37,046  | 37,368 |
| multiple-choice           | MMLU            | -                | 99,842    | 1,531   | 14,042 |
| multiple-choice           | TruthfulQA      | wiki             | /         | 817     | /      |
| multiple-choice           | HellaSWAG       | ActivityNet      | 39,905    | 10,042  | /      |
| multiple-choice           | ARC             | -                | 3,370     | 869     | 3,548  |
| multiple-choice           | OpenBookQA      | -                | 4,957     | 500     | 500    |
| Fact Verification         | FEVER           | wiki             | 104,966   | 10,444  | /      |
| Dialog Generation         | WOW             | wiki             | 63,734    | 3,054   | /      |
| Entity Linking            | AIDA CoNll-yago | Freebase & wiki  | 18,395    | 4,784   | /      |
| Entity Linking            | WNED            | Wiki             | /         | 8,995   | /      |
| Slot Filling              | T-REx           | DBPedia          | 2,284,168 | 5,000   | /      |
| Slot Filling              | Zero-shot RE    | wiki             | 147,909   | 3,724   | /      |

### Document Corpus

Our toolkit supports jsonl format for retrieval document collections, with the following structure:

```jsonl
{"id":"0", "contents": "...."}
{"id":"1", "contents": "..."}
```
The `contents` key is essential for building the index. For documents that include both text and title, we recommend setting the value of  `contents` to `{title}\n{text}`. The corpus file can also contain other keys to record additional characteristics of the documents.

In the academic research, Wikipedia and MS MARCO are the most commonly used retrieval document collections. For Wikipedia, we provide a [comprehensive script](./docs/process-wiki.md) to process any Wikipedia dump into a clean corpus. Additionally, various processed versions of the Wikipedia corpus are available in many works, and we have listed some reference links.


For MS MARCO, it is already processed upon release and can be directly downloaded from its [hosting link](https://huggingface.co/datasets/Tevatron/msmarco-passage-corpus) on Hugging Face.


## :raised_hands: Additional FAQs

- [How to build my own corpus, such as a specific segmented Wikipedia?](./docs/process-wiki.md) 
- [How to index my own corpus?](./docs/building-index.md)

## :bookmark: License

FlashRAG is licensed under the [MIT License](./LICENSE).

## :star2: Citation
Please kindly cite our paper if helps your research:
```BibTex
@article{FlashRAG,
    author={Jiajie Jin and
            Yutao Zhu and
            Xinyu Yang and
            Chenghao Zhang and
            Zhicheng Dou},
    title={FlashRAG: A Modular Toolkit for Efficient Retrieval-Augmented Generation Research},
    journal={CoRR},
    volume={abs/2405.13576},
    year={2024},
    url={https://arxiv.org/abs/2405.13576},
    eprinttype={arXiv},
    eprint={2405.13576}
}
```
