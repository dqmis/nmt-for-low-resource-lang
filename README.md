# Multilingual Neural Machine Translation for Low-Resource Languages

---

## Project Setup and Requirements

- ### Requirements

The project requires `Python ^3.11` version. Other dependencies are listed in the `pyproject.toml` file.

- ### Installation

The project uses Poetry to manage dependencies. To install the dependencies on Snellius, run the following command:

```bash
# snel specific
/sw/arch/RHEL8/EB_production/2023/software/Anaconda3/2023.07-2/bin/conda init bash

# restart shell
conda create python=3.11 -n venv
conda activate venv

pip install poetry

## or use poetry to create a virtual environment
# poetry env use python3.11
# poetry shell

poetry install
poetry run pre-commit install
```
# Steps to run

- ### Connect to Snellius and request resources

You can request resources using the following command:

```bash
srun --partition=gpu --gpus=1 --ntasks=1 --cpus-per-task=18  --time=00:01:00 --pty bash -i

conda activate venv
```

or `run a file with the job specification` that will run jobs in the background on Snellius. This allows you to submit jobs that will keep running even if you disconnect from the server. Example of this file is `train_job.slurm`, You can adjust it to Your needs and run:

```bash
sbatch train_job.slurm
```

To view the status of the job, you can use the following command:

```bash
squeue -u $USER
```

- ### Download the data

Download the dataset of the chosen language. Run the script and choose source and target languages:

```bash
python scripts/download_nllb.py
```

- ### Augment the downloaded data

To augment the dataset by using backtranslation, run the following script and provide all the necessary arguments:

- **dataset_path**
- **output_path**
- **lang_from**
- **lang_to**

Set the language_from to the target language (e.g. Estonian) and language_to which usually is English.

Example:

```bash
python scripts/augment_data_backtranslate.py --dataset_path data/opus.nllb.en-ee/en-ee.txt/NLLB.en-ee.en --output_path out/backtranslated-ee --lang_from ee --lang_to en
```

You could also run the script with the following command:

```bash
sbatch backtranslate_job.slurm
```

- ### Convert txt file into parquet

Convert generated `.txt` files from backtranslation to `.parquet` file for training.

```bash
python scripts/convert_to_parquet.py --data_dir=out/backtranslated-ee --original_data_dir=data/opus.nllb.en-ee/en-ee.txt/NLLB.en-ee.ee --output_parquet_file=data/bt-opus.nllb.en-ee/nllb-ee-backtranslated.parquet --orig_lang=ee
```

- ### Training

Run the training script with the name of the configuration file as an argument:

```bash
python scripts/train.py train_bt_ee.yaml
```

Or run the training script with the job specification file and adjust it to your needs:

```bash
sbatch train_job.slurm
```

- ### Augment LLM data

To augment the dataset by using the LLM, run the following script and provide all the necessary arguments:

- **dataset_path**
- **output_path**
- **lang_from**

Set the language_from to the target language (e.g. Estonian, Afrikans).

```bash
python scripts/augment_data_llm.py --dataset_path=data/opus.nllb.en-ee/en-ee.txt/NLLB.en-ee.ee --output_path=out/llm-ee --lang_from=Estonian
```

Or run the script with the job specification file and adjust it to your needs:

```bash
sbatch augment_llm_job.slurm
```

- ### Join LLM augmented data

To join the LLM augmented data with the original data, run the following script and provide all the necessary arguments:

- **dataset_path**
- **output_path**

```bash
python scripts/join_llm_augmented_data.py --dataset_path=out/llm-ee --output_file_path=data/llm.nllb.en-ee/nllb-ee-llm.txt
```

- ### Translate LLM data

To translate the data using LLM model, run the following script and provide all the necessary arguments:

- **dataset_path**
- **output_path**

```bash
python scripts/translate_data_llm.py --dataset_path=data/opus.nllb.en-ee/en-ee.txt/NLLB.en-ee.en --output_path=out/translated-llm-ee
```

Or run the script with the job specification file and adjust it to your needs:

```bash
sbatch translate_llm_job.slurm
```

- ### Join LLM translated data

Run the same script as for joining LLM augmented data, but adjust parameters.

- ### Training with LLM augmented data and LLM translated data

To run the training script with the LLM augmented data and LLM translated data, run the same script as for training backtranslation data, but adjust the configuration files.