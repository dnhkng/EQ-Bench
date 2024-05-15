![EQ-Bench Logo](./images/eqbench_logo_sml.png)

# EQ-Bench

EQ-Bench is a benchmark for language models designed to assess emotional intelligence. You can read more about it in our [paper](https://arxiv.org/abs/2312.06281).

The latest leaderboard can be viewed at [EQ-Bench Leaderboard](https://eqbench.com).

## News

### 2024-04-19 Minor updates

- Changed behaviour when using Transformers and no chat template is specified. In this scenario, the benchmark will now apply the tokenizer's chat template if there is one.
- Models are now loaded in 16 bit precision if "none" quantisation is selected.
- Preliminary support for Llama3 models (adding <|eot_id|> to the tokenizer).

### Version 2.3

This version includes two new benchmarks: `creative-writing` and `judgemark`.

#### Creative Writing Benchmark

This is a LLM-as-a-judge benchmark using detailed criteria to assess the model's output to a set of creative writing prompts.

#### Judgemark

This is the latest benchmark task in the EQ-Bench pipeline. It tests a model's ability to judge creative writing from a set of pre-generated outputs from 20 test models. The writing prompts & judging process are the same as those used in the creative writing benchmark. Several metrics are aggregated on the judge's performance (correlation with other benchmarks + measures of spread), resulting in a "Judgemark" score.

#### Launching the New Benchmarks

To launch individual benchmark tasks: 

`python eq-bench.py --benchmarks eq-bench`

`python eq-bench.py --benchmarks creative-writing`

`python eq-bench.py --benchmarks judgemark`

The creative-writing and judgemark tasks require the following parameters to be configures in your config.cfg:

`judge_model_api = `
`judge_model = `
`judge_model_api_key = `

#### Creative Writing Benchmark Details

Official scores for the creative-writing benchmark use claude-3-opus as judge. However you can use any openai, mistralai or anthropic model as judge. Be aware that the results won't be directly comparable between judge models.

The creative writing benchmark involves 19 writing prompts. The model's output is then judged according to 36 criteria for good & bad writing, with each criteria being scored 0-10.

Given the small number of questions in the test set, you may wish to run several iterations of the benchmark to reduce variance. You can set n_iterations in the config per benchmark run. We recommend 3+ iterations. Benchmarking a model over 3 iterations using Claude Opus will cost approx. $3.00.

Temperature is set at 0.7 for the test model inference, so output will vary between iterations.

[For more details, click here.](https://eqbench.com/about.html)

<details>
<summary>### Version 2.2 Released</summary>

Changes:

- Added [llama.cpp](https://github.com/ggerganov/llama.cpp) support
- Fixed bug with ooba regularly crashing
- Misc compatibility & bug fixes

* If using llama.cpp as the inferencing engine, you will need to launch the llama.cpp server first and then run the benchmark. The benchmark will look for the api at the default address of `http://127.0.0.1:8080`. Multiple benchmark runs are not supported when using llama.cpp.

Other news: EQ-Bench has been added to eleuther-eval-harness! Go [check it out](https://github.com/EleutherAI/lm-evaluation-harness).

</details>

<details>
<summary>### Version 2.1 Released</summary>

Changes:

- Add support for additional languages (just en and de in this release)
- Add support for custom OpenAI-compatible api endpoints, such as ollama
- By default the revision component of the questions is not included

<details>
<summary>v2.1 details</summary>

### DE Support

German language support was kindly added by [CrispStrobe](https://github.com/CrispStrobe). The prompts were translated by GPT-4. You can expect the scores using the German version to be slightly lower than the English version, assuming the model's language competency for each is equal.

### Revision Component

After collecting a lot of data from v2, it's clear that the revision component has a mostly negative effect. Only 8% of the time does it improve the score, and on average the revised score is 2.95% lower than the first pass score. Since we choose the highest of the first pass vs revised aggregate scores, the revision component is rarely affecting the overall score.

Since revising requires significantly more inference, we opt to set it off by default. You can still enable it with the `-revise` argument. The upshot of disabling revision is that the benchmark is now much cheaper/faster to run, and the prompts are a little less complex. This change should have a negligible effect on scores.

</details>

</details>

<details>
<summary>### Version 2 Released</summary>

V2 of EQ-Bench contains 171 questions (compared to 60 in v1) and a new system for scoring. It is better able to discriminate performance differences between models. V2 is less subject to variance caused by perturbations (e.g. temp, sampler, quantisation, prompt format, system message). Also added is the ability to upload results to firebase.

We encourage you to move to v2, and to note which version you are using (EQ-Bench v1 or EQ-Bench v2) when publishing results to avoid confusion.

NOTE: V1 scores are not directly comparable to v2 scores.

<details>
<summary>More v2 details</summary>

Version 2 of the benchmark brings three major changes:

1. Increased the number of test questions from 60 to 171.
2. Changed the scoring system from _normalised_ to _full scale_.
3. Uploading results to firebase.

### Known issues:

- When using oobabooga as the inferencing engine, the api plugin stops responding after approx. 30 queries. This is handled by the benchmark pipeline by the query timing out (according to the value set in config.cfg), and then reloading ooba. The cause is unknown at this stage; the benchmark should however still complete.

### Score sensitivity to perturbations

Originally 200 dialogues were generated for the test set, of which 60 of the best (most coherent & challenging) were selected for v1 of the benchmark. We had initially established very low variance between runs of the v1 benchmark, when holding all parameters the same. However it has become apparent that minor perturbations to the model or inferencing parameters can cause score variance beyond what is explained by the actual change in performance.

Traditional multiple choice tests are less prone to this kind of variance because these perturbations are unlikely to change an answer from "A" to "B". In contrast, EQ-Bench questions require a subjective prediction of emotional intensity on a range of 0-10. Small perturbations to the model or inferencing params can produce significantly different numerical predictions. This is a source of noise that can be mitigated by increasing the number of questions. So for v2 we opted to expand the test set to 171 out of the originally generated 200.

We tested v1 against v2 for a number of models, while controlling a range of parameters (temp, sampler, quantisation, prompt format, system message). We find v2 scores to be significantly more stable to perturbations to these variables, and so we expect the scores to be more closely representative of the true performance of the model.

### Scoring system changes

In v1 of EQ-Bench we elected to normalise the four emotional intensity ratings in each question to sum to 10. The reasoning for this was that different subjects might have different ideas about, for example, what constitutes a _10_ rating. Given the subjectivity here, multiple perspectives can be valid.

A systematic bias in how the subject rates emotional intensity might correlate with a similar systematic bias in the creators of the reference answers, resulting in an artificially inflated score. So to eliminate this issue we normalised both the reference answer and the subject answer so that we are only comparing the _relative_ intensity of each emotion.

This seemed like a good idea at the time, however normalising in this way is far from a perfect solution. It handles certain edge cases poorly, and several models benchmarked with numbers that were significant outliers compared to other major benchmarks (E.g. Mixtral 8x7 produced unusually low scores). In addition, normalising the answers means we are losing the ability to assess the model's ability to make reasonable predictions about the absolute intensity of emotions.

In v2 we opted for a different approach: We still calculate the score by computing the difference from the reference answer, however, we no longer normalise the values. To mitigate the subjective nature of rating emotional intensity, we scale down smaller differences (differences between 1-4 from reference) on a curve. Differences from 5 to 10 are counted 1:1.

The result of these changes is better discriminative ability of the benchmark, and generally slightly higher scores compared to v1. As with v1, the score baseline is calibrated so that a score of 0 corresponds to answering randomly, and a score of 100 matches the reference answers exactly.

</details>

</details>

<details>
<summary>### Version 1.1 Released</summary>

This version adds support for Oobabooga. The benchmark pipeline can automatically download each model, launch the model with ooba using the specified parameters, and close the ooba server after the run completes, optionally deleting the model files.

</details>

## Requirements

- Linux
- Python 3x
- Working install of Oobabooga (optional)
- Sufficient GPU / System RAM to load the models
- Python libraries listed in `install_reqs.sh`

<details>
<summary>Show python libraries</summary>

### EQ-bench requirements
- `tqdm`
- `sentencepiece`
- `hf_transfer`
- `openai`
- `scipy`
- `torch`
- `peft`
- `bitsandbytes`
- `transformers` (preferably the latest version installed directly from GitHub: `huggingface/transformers`)
- `trl`
- `accelerate`
- `tensorboardX`
- `huggingface_hub`

### Requirements for QWEN models
- `einops`
- `transformers_stream_generator` (version 0.0.4)
- `deepspeed`
- `tiktoken`
- `flash-attention` (the latest version installed directly from GitHub: `Dao-AILab/flash-attention`)
- `auto-gptq`
- `optimum`

### Requirements for uploading results
- `gspread`
- `oauth2client`
- `firebase_admin`

</details>

## Installation

### Quick start:

If you are installing EQ-Bench into a fresh linux install (like a runpod or similar), you can run `ooba_quick_install.sh`. This will install oobabooga into the current user's home directory, install all EQ-Bench dependencies, then run the benchmark pipeline.

### Install (not using quick start)

Note: Ooobabooga is optional. If you prefer to use transformers as the inference engine, or if you are only benchmarking through the OpenAI API, you can skip installing it.

- Install the required Python dependencies by running `install_reqs.sh`.
- Optional: install the [Oobabooga library](https://github.com/oobabooga/text-generation-webui/tree/main) and make sure it launches.
- Optional: Set up firebase / firestore for results upload (see instructions below).
- Optional: Set up Google Sheets for results upload (see instructions below).

### Configure

- Set up `config.cfg` with your API keys and runtime settings.
- Add benchmark runs to `config.cfg`, in the format:
   - `run_id, instruction_template, model_path, lora_path, quantization, n_iterations, inference_engine, ooba_params, downloader_filters`

      - `run_id`: A name to identify the benchmark run
      - `instruction_template`: The filename of the instruction template defining the prompt format, minus the .yaml (e.g. Alpaca)
      - `model_path`: Huggingface model ID, local path, or OpenAI model name
      - `lora_path` (optional): Path to local lora adapter
      - `quantization`: Using bitsandbytes package (8bit, 4bit, None)
      - `n_iterations`: Number of benchmark iterations (final score will be an average)
      - `inference_engine`: Set this to transformers, openai, ooba or llama.cpp.
      - `ooba_params` (optional): Any additional ooba params for loading this model (overrides the global setting above)
      - `downloader_filters` (optional): Specify --include or --exclude patterns (using same syntax as huggingface-cli download)

## Benchmark run examples

`# run_id, instruction_template, model_path, lora_path, quantization, n_iterations, inference_engine, ooba_params, downloader_args`

`myrun1, openai_api, gpt-4-0613, , , 1, openai, ,`

`myrun2, Llama-v2, meta-llama/Llama-2-7b-chat-hf, /path/to/local/lora/adapter, 8bit, 3, transformers, , ,`

`myrun3, Alpaca, ~/my_local_model, , None, 1, ooba, --loader transformers --n_ctx 1024 --n-gpu-layers -1, `

`myrun4, Mistral, TheBloke/Mistral-7B-Instruct-v0.2-GGUF, , None, 1, ooba, --loader llama.cpp --n-gpu-layers -1 --tensor_split 1,3,5,7, --include ["*Q3_K_M.gguf", "*.json"]`

`myrun5, Mistral, mistralai/Mistral-7B-Instruct-v0.2, , None, 1, ooba, --loader transformers --gpu-memory 12, --exclude "*.bin"`

`myrun6, ChatML, model_name, , None, 1, llama.cpp, None,`

## Running the benchmark

- Run the benchmark:
   - `python3 eq-bench.py`
- Results are saved to `benchmark_results.csv`

## Script Options

- `-h`: Displays help.
- `-w`: Overwrites existing results (i.e., disables the default behaviour of resuming a partially completed run).
- `-d`: Downloaded models will be deleted after each benchmark successfully completes. Does not affect previously downloaded models specified with a local path.
- `-f`: Use hftransfer for multithreaded downloading of models (faster but can be unreliable).
- `-v`: Display more verbose output.
- `-r`: Set the number of retries to attempt if a benchmark run fails. Default is 5.
- `-l`: Sets the language: `en` and `de` currently supported. Defaults to English if not specified.
- `-v1`: Runs v1 of the benchmark (legacy). If not set, the benchmark defaults to v2.
- `-revise`: Enables the revision component of the test questions (this is off by default since v2.1).

## Prompt Formats / Instruction Templates

EQ-Bench uses the same instruction template format as the Oobabooga library. You can modify the existing ones or add your own. When you specify a prompt format in config.cfg, use the filename minus the .yaml, e.g. Alpaca.

- If using `transformers` as the inference engine, the benchmark pipeline uses templates located in `[EQ-Bench dir]/instruction-templates`.
- If using `ooba` as the inference engine, the pipeline uses templates located in `[ooba dir]/instruction-templates`

When using transformers, if you leave the prompt format blank in config.cfg, transformers will apply the chat template in the tokenizer if there is one.

When using ooba, if you leave the prompt format blank in config.cfg, ooba will make its best guess as to what the prompt format should be.

## Setting up Firebase / Firestore for Results Uploading (Optional)

<details>
  <summary>Show instructions</summary>

1. Create a new firebase project.
2. Create a service account within this project.
3. Generate a new private key, save to `firebase_creds.json` in EQ-Bench root directory.
4. Create a default firestore database in the project.

When EQ-Bench sees `firebase_creds.json` in the EQ-Bench directory, it will upload results to this firestore db when a benchmark run completes.

</details>

## Setting up Google Sheets for Results Uploading (Optional)

<details>
  <summary>Show instructions</summary>
  
1. Create a new Google Sheet.
2. Set the share settings so that anyone with the link can edit.
3. Set google_spreadsheet_url in `config.cfg` to the URL of the sheet you just created.
4. Go to [Google Cloud Console](https://console.cloud.google.com/).
5. Create a new project and ensure it is selected as active in the dropdown at the top of the page.
6. Enable the Google Sheets API for the project:
   - In the search bar, type "sheets"
   - Click Google Sheets API
   - Click `Enable`
7. Create a service account:
   - In the search bar, type "Service accounts" and click the appropriate result
   - Click `+ Create Service Account`
   - Give it a name & id, then click `Create and continue`
   - Grant this service account access: Basic -> Editor
   - Click `Done`
8. Click on the service account, then navigate to Keys -> Add key -> Create new key -> JSON.
9. Save the file to `google_creds.json` in the eq-bench directory.

</details>

## Cite

```
@misc{paech2023eqbench,
      title={EQ-Bench: An Emotional Intelligence Benchmark for Large Language Models}, 
      author={Samuel J. Paech},
      year={2023},
      eprint={2312.06281},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```