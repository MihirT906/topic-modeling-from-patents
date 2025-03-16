# TopicGPT
This repository contains the code for our project for the course CS685-NeuralNetworks. The code has reused the implementation from paper: ["TopicGPT: Topic Modeling by Prompting Large Language Models"](https://arxiv.org/abs/2311.01449). 

![TopicGPT Pipeline Overview](pipeline.png)



## Updates
We have modified pipelines for our project detailed below:

### Background
The USPTO patent classification system assigns patents to one or multiple categories. This classification is typically done by humans before a patent is approved. Over time, the classification hierarchy (CPC) has evolved to accommodate new technologies and applications. For example, with the increasing interest in climate change-related technology, the category "Y02", which relates to patents focusing on "technologies for mitigating climate change" is a new addition.

### Problem Statement
In this project, we leverage Y02 patent data to align TopicGPT for the creation of topic hierarchies. This system serves two key purposes: (1) assisting patent officials in streamlining the labor-intensive process of manually annotating patents, and (2) empowering innovators and industry professionals to uncharted patents accessible from different points of view.

### Contributions
- Collected and compiled a new dataset by scraping recent data from USPTO's website. Our dataset was published on [Kaggle](https://www.kaggle.com/datasets/mattisbodynek/uspto-climate-positive-patent-abstracts-cpc-y02)

- Applied prompt engineering and few-shot learning to align TopicGPT with our generation objectives. 

- Developed a robust evaluation technique utilizing inter-annotator agreement with LLMs like Mistral, Llama 3, Qwen, and DBRX

- Demonstrated superior interpretability and flexibility to user-defined goals compared to BERTTopic

## Setup
- Install the requirements: `pip install -r requirements.txt`
- Set your OpenAI key according to [this article](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety).
- Refer to https://openai.com/pricing/ for OpenAI API pricing or to https://blog.perplexity.ai/blog/introducing-pplx-api for Perplexity API pricing.

## Data
- Prepare your `.jsonl` data file in the following format:
    ```
    {
        "id": "Optional IDs",
        "text": "Documents",
        "label": "Optional ground-truth labels"
    }
    ```
- Put the data file in `data/input`. There is also a sample data file `data/input/sample.jsonl` to debug the code.
- If you want to sample a subset of the data for topic generation, run `python script/data.py --data <data_file> --num_samples 1000 --output <output_file>`. This will sample 1000 documents from the data file and save it to `<output_file>`. You can also specify `--num_samples` to sample a different number of documents, see the paper for more detail.
- Raw dataset: [[link]](https://drive.google.com/drive/folders/1rCTR5ZQQ7bZQoewFA8eqV6glP6zhY31e?usp=sharing). 

## Pipeline
- You can either run `script/run.sh` to run the entire pipeline or run each step individually. See the notebook in `script/example.ipynb` for a step-by-step guide.
- Topic generation: Modify the prompts according to the templates in `templates/generation_1.txt` and `templates/seed_1.md`. Then, to run topic generation, do: 
    ```
    python3 script/generation_1.py --deployment_name gpt-4 \
                            --max_tokens 300 --temperature 0.0 --top_p 0.0 \
                            --data data/input/sample.jsonl \
                            --prompt_file prompt/generation_1.txt \
                            --seed_file prompt/seed_1.md \
                            --out_file data/output/generation_1.jsonl \
                            --topic_file data/output/generation_1.md \
                            --verbose True
    ```

- Topic refinement: If you want to refine the topics, modify the prompts according to the templates in `templates/refinement.txt`. Then, to run topic refinement, do: 
    ```
    python3 refinement.py --deployment_name gpt-4 \
                    --max_tokens 500 --temperature 0.0 --top_p 0.0 \
                    --prompt_file prompt/refinement.txt \
                    --generation_file data/output/generation_1.jsonl \
                    --topic_file data/output/generation_1.md \
                    --out_file data/output/refinement.md \
                    --verbose True \
                    --updated_file data/output/refinement.jsonl \
                    --mapping_file data/output/refinement_mapping.txt \
                    --refined_again False \
                    --remove False
    ```

- Topic assignment: Modify the prompts according to the templates in `templates/assignment.txt`. Then, to run topic assignment, do: 
    ```
    python3 script/assignment.py --deployment_name gpt-3.5-turbo \
                            --max_tokens 300 --temperature 0.0 --top_p 0.0 \
                            --data data/input/sample.jsonl \
                            --prompt_file prompt/assignment.txt \
                            --topic_file data/output/generation_1.md \
                            --out_file data/output/assignment.jsonl \
                            --verbose True
    ```
- Topic correction: If the assignment contains errors or hallucinated topics, modify the prompts according to the templates in `templates/correction.txt` (note that this prompt is very similar to the assignment prompt, only adding a `{Message}` field towards the end of the prompt). Then, to run topic correction, do: 
    ```
    python3 script/correction.py --deployment_name gpt-3.5-turbo \
                            --max_tokens 300 --temperature 0.0 --top_p 0.0 \
                            --data data/output/assignment.jsonl \
                            --prompt_file prompt/correction.txt \
                            --topic_file data/output/generation_1.md \
                            --out_file data/output/assignment_corrected.jsonl \
                            --verbose True
    ```

- Second-level topic generation: If you want to generate second-level topics, modify the prompts according to the templates in `templates/generation_2.txt`. Then, to run second-level topic generation, do: 
    ```
    python3 script/generation_2.py --deployment_name gpt-4 \
                    --max_tokens 300 --temperature 0.0 --top_p 0.0 \
                    --data data/output/generation_1.jsonl \
                    --seed_file data/output/generation_1.md \
                    --prompt_file prompt/generation_2.txt \
                    --out_file data/output/generation_2.jsonl \
                    --topic_file data/output/generation_2.md \
                    --verbose True
    ```

## Cite
```
@misc{pham2023topicgpt,
      title={TopicGPT: A Prompt-based Topic Modeling Framework}, 
      author={Chau Minh Pham and Alexander Hoyle and Simeng Sun and Mohit Iyyer},
      year={2023},
      eprint={2311.01449},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```
