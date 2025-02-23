# COVID-19 EHR Benchmarks

> A Comprehensive Benchmark For COVID-19 Predictive Modeling Using Electronic Health Records

## Prediction Tasks

- [x] (Early) Mortality outcome prediction
- [x] Length-of-stay prediction
- [x] Multi-task/Two-stage prediction

## Model Zoo

### Machine Learning Models

- [x] Random forest (RF)
- [x] Decision tree (DT)
- [x] Gradient Boosting Decision Tree (GBDT)
- [x] XGBoost
- [x] CatBoost

### Deep Learning Models

- [x] Multi-layer perceptron (MLP)
- [x] Recurrent neural network (RNN)
- [x] Long-short term memory network (LSTM)
- [x] Gated recurrent units (GRU)
- [x] Temporal convolutional networks
- [x] Transformer

### EHR Predictive Models

- [x] RETAIN
- [x] StageNet
- [x] Dr. Agent
- [x] AdaCare
- [x] ConCare
- [x] GRASP

## Code Description

```shell
app/
    apis/
        ml_{task}.py # machine learning pipelines
        dl_{task}.py # deep learning pipelines
    core/
        evaluation/ # evaluation metrics
        utils/
    datasets/ # dataset loader scripts
    models/
        backbones/ # feature extractors
        classifiers/ # prediction heads
        losses/ # task related loss functions
        build_model.py # concat backbones and heads
configs/
    _base_/
    # common configs
        datasets/
        # dataset basic info, training epochs and dataset split strategy
            {dataset}.yaml
        db.yaml # database settings (optional)
    {config_name}.yaml # detailed model settings
checkpoints/ # model checkpoints are stored here
datasets/ # raw/processed dataset and pre-process script
main.py # main entry point
requirements.txt # code dependencies
```

## Requirements

- Python 3.7+
- PyTorch 1.10+
- Cuda 10.2+ (If you plan to use GPU)

Note:

- Most models can be run quickly on CPU.
- You are required to have a GPU with 12GB memory to run ConCare model on CDSL dataset.
- TCN model may run much faster on CPU.

## Usage

- Install requirements.

    ```bash
    pip install -r requirements.txt [-i https://pypi.tuna.tsinghua.edu.cn/simple] # [xxx] is optional
    ```

- Download TJH dataset from [An interpretable mortality prediction model for COVID-19 patients](https://www.nature.com/articles/s42256-020-0180-7), unzip and put it in `datasets/tongji/raw_data/` folder.
- Run preprocessing notebook. (You can skip this step if you have already done this in the later training process)
- (The CDSL dataset is also the same process.) You need to apply for the CDSL dataset if necessary. [Covid Data Save Lives Dataset](https://www.hmhospitales.com/coronavirus/covid-data-save-lives/english-version)
- Run following commands to train models.

    ```bash
    python main.py --cfg configs/xxx.yaml [--train] [--cuda CUDA_NUM] [--db]
    # Note:
    # 1) use --train for training, only infererence stage if not
    # 2) If you plan to use CUDA, use --cuda 0/1/2/...
    # 3) If you have configured database settings, you can use --db to upload performance after training to the database.
    ```

## Data Format

The shape and meaning of the tensor fed to the models are as follows:

- `x.pkl`: (N, T, D) tensor, where N is the number of patients, T is the number of time steps, and D is the number of features. At $D$ dimention, the first $x$ features are demographic features, the next $y$ features are lab test features, where $x + y = D$
- `y.pkl`: (N, T, 2) tensor, where the 2 values are [outcome, length-of-stay] for each time step.
- `visits_length.pkl`: (N, ) tensor, where the value is the number of visits for each patient.
- `missing_mask.pkl`: same shape as `x.pkl`, tell whether features are imputed. `1`: existing, `0`: missing.

Pre-processed data are stored in `datasets/{dataset}/processed_data/` folder.

## Database preparation [Optional]

Example `db.yaml` settings, put it in `configs/_base_/db.yaml`.

```bash
engine: postgresql # or mysql
username: db_user
password: db_password
host: xx.xxx.com
port: 5432
database: db_name
```

Create `perflog` table in your database:

```sql
-- postgresql example
create table perflog
(
	id serial
		constraint perflog_pk
			primary key,
	record_time integer,
	model_name text,
	performance text,
	hidden_dim integer,
	dataset text,
	model_type text,
	config text,
	task text
);

-- mysql example
create table perflog
(
	id int auto_increment,
	record_time int null,
	model_name text null,
	task text null,
	performance text null,
	hidden_dim int null,
	dataset text null,
	model_type text null,
	config text null,
	constraint perflog_id_uindex
		unique (id)
);

alter table perflog
	add primary key (id);
```

## Configs

Below is the configurations after hyperparameter selection.

<details>

<summary>ML models</summary>

```bash
hm_los_catboost_kf10_md6_iter150_lr0.1_test
hm_los_decision_tree_kf10_md10_test
hm_los_gbdt_kf10_lr0.1_ss0.8_ne100_test
hm_los_random_forest_kf10_md10_mss2_ne100_test
hm_los_xgboost_kf10_lr0.01_md5_cw3_test
hm_outcome_catboost_kf10_md3_iter150_lr0.1_test
hm_outcome_decision_tree_kf10_md10_test
hm_outcome_gbdt_kf10_lr0.1_ss0.6_ne100_test
hm_outcome_random_forest_kf10_md20_mss10_ne100_test
hm_outcome_xgboost_kf10_lr0.1_md7_cw3_test
tj_los_catboost_kf10_md3_iter150_lr0.1_test
tj_los_decision_tree_kf10_md10_test
tj_los_gbdt_kf10_lr0.1_ss0.8_ne100_test
tj_los_random_forest_kf10_md20_mss5_ne100_test
tj_los_xgboost_kf10_lr0.01_md5_cw1_test
tj_outcome_catboost_kf10_md3_iter150_lr0.1_test
tj_outcome_decision_tree_kf10_md10_test
tj_outcome_gbdt_kf10_lr0.1_ss0.6_ne100_test
tj_outcome_random_forest_kf10_md20_mss2_ne10_test
tj_outcome_xgboost_kf10_lr0.1_md5_cw5_test
```

</details>

<details>
<summary>DL/EHR models</summary>

```bash
tj_outcome_grasp_ep100_kf10_bs64_hid64
tj_los_grasp_ep100_kf10_bs64_hid128
tj_outcome_concare_ep100_kf10_bs64_hid128
tj_los_concare_ep100_kf10_bs64_hid128
tj_outcome_agent_ep100_kf10_bs64_hid128
tj_los_agent_ep100_kf10_bs64_hid64
tj_outcome_adacare_ep100_kf10_bs64_hid64
tj_los_adacare_ep100_kf10_bs64_hid64
tj_outcome_transformer_ep100_kf10_bs64_hid128
tj_los_transformer_ep100_kf10_bs64_hid64
tj_outcome_tcn_ep100_kf10_bs64_hid128
tj_los_tcn_ep100_kf10_bs64_hid128
tj_outcome_stagenet_ep100_kf10_bs64_hid64
tj_los_stagenet_ep100_kf10_bs64_hid64
tj_outcome_rnn_ep100_kf10_bs64_hid64
tj_los_rnn_ep100_kf10_bs64_hid128
tj_outcome_retain_ep100_kf10_bs64_hid128
tj_los_retain_ep100_kf10_bs64_hid128
tj_outcome_mlp_ep100_kf10_bs64_hid64
tj_los_mlp_ep100_kf10_bs64_hid128
tj_outcome_lstm_ep100_kf10_bs64_hid64
tj_los_lstm_ep100_kf10_bs64_hid128
tj_outcome_gru_ep100_kf10_bs64_hid64
tj_los_gru_ep100_kf10_bs64_hid128
tj_multitask_rnn_ep100_kf10_bs64_hid64
tj_multitask_lstm_ep100_kf10_bs64_hid128
tj_multitask_gru_ep100_kf10_bs64_hid128
tj_multitask_transformer_ep100_kf10_bs64_hid128
tj_multitask_tcn_ep100_kf10_bs64_hid64
tj_multitask_mlp_ep100_kf10_bs64_hid128
tj_multitask_adacare_ep100_kf10_bs64_hid128
tj_multitask_agent_ep100_kf10_bs64_hid64
tj_multitask_concare_ep100_kf10_bs64_hid128
tj_multitask_stagenet_ep100_kf10_bs64_hid64
tj_multitask_grasp_ep100_kf10_bs64_hid128
tj_multitask_retain_ep100_kf10_bs64_hid64
hm_outcome_mlp_ep100_kf10_bs64_hid64
hm_los_mlp_ep100_kf10_bs64_hid128
hm_outcome_lstm_ep100_kf10_bs64_hid64
hm_los_lstm_ep100_kf10_bs64_hid128
hm_outcome_gru_ep100_kf10_bs64_hid64
hm_los_gru_ep100_kf10_bs64_hid128
hm_outcome_grasp_ep100_kf10_bs64_hid64
hm_los_grasp_ep100_kf10_bs64_hid64
hm_outcome_concare_ep100_kf10_bs64_hid128
hm_los_concare_ep100_kf10_bs64_hid64
hm_outcome_agent_ep100_kf10_bs64_hid128
hm_los_agent_ep100_kf10_bs64_hid64
hm_outcome_adacare_ep100_kf10_bs64_hid64
hm_los_adacare_ep100_kf10_bs64_hid128
hm_outcome_transformer_ep100_kf10_bs64_hid128
hm_los_transformer_ep100_kf10_bs64_hid128
hm_outcome_tcn_ep100_kf10_bs64_hid64
hm_los_tcn_ep100_kf10_bs64_hid128
hm_outcome_stagenet_ep100_kf10_bs64_hid64
hm_los_stagenet_ep100_kf10_bs64_hid64
hm_outcome_rnn_ep100_kf10_bs64_hid64
hm_los_rnn_ep100_kf10_bs64_hid128
hm_outcome_retain_ep100_kf10_bs64_hid128
hm_los_retain_ep100_kf10_bs64_hid128
hm_multitask_rnn_ep100_kf10_bs512_hid128
hm_multitask_lstm_ep100_kf10_bs512_hid64
hm_multitask_gru_ep100_kf10_bs512_hid128
hm_multitask_transformer_ep100_kf10_bs512_hid64
hm_multitask_tcn_ep100_kf10_bs512_hid64
hm_multitask_mlp_ep100_kf10_bs512_hid128
hm_multitask_adacare_ep100_kf10_bs512_hid128
hm_multitask_agent_ep100_kf10_bs512_hid128
hm_multitask_concare_ep100_kf10_bs64_hid128
hm_multitask_stagenet_ep100_kf10_bs512_hid128
hm_multitask_grasp_ep100_kf10_bs512_hid64
hm_multitask_retain_ep100_kf10_bs512_hid128
```
</details>

<details>
<summary>Two stage configs</summary>

```bash
tj_twostage_adacare_kf10.yaml
tj_twostage_agent_kf10.yaml
tj_twostage_concare_kf10.yaml
tj_twostage_gru_kf10.yaml
tj_twostage_lstm_kf10.yaml
tj_twostage_mlp_kf10.yaml
tj_twostage_retain_kf10.yaml
tj_twostage_rnn_kf10.yaml
tj_twostage_stagenet_kf10.yaml
tj_twostage_tcn_kf10.yaml
tj_twostage_transformer_kf10.yaml
tj_twostage_grasp_kf10.yaml
hm_twostage_adacare_kf10.yaml
hm_twostage_agent_kf10.yaml
hm_twostage_concare_kf10.yaml
hm_twostage_gru_kf10.yaml
hm_twostage_lstm_kf10.yaml
hm_twostage_mlp_kf10.yaml
hm_twostage_retain_kf10.yaml
hm_twostage_rnn_kf10.yaml
hm_twostage_stagenet_kf10.yaml
hm_twostage_tcn_kf10.yaml
hm_twostage_transformer_kf10.yaml
hm_twostage_grasp_kf10.yaml
```
</details>

## Contributing

We appreciate all contributions to improve covid-emr-benchmarks. Pull Requests amd Issues are welcomed!

## Contributors

[Yinghao Zhu](https://github.com/yhzhu99), [Wenqing Wang](https://github.com/ericaaaaaaaa), [Junyi Gao](https://github.com/v1xerunt)

## Citation

If you find this project useful in your research, please consider cite:

```BibTeX
@misc{https://doi.org/10.48550/arxiv.2209.07805,
  doi = {10.48550/ARXIV.2209.07805},
  url = {https://arxiv.org/abs/2209.07805},
  author = {Gao, Junyi and Zhu, Yinghao and Wang, Wenqing and Wang, Yasha and Tang, Wen and Ma, Liantao},
  keywords = {Machine Learning (cs.LG), FOS: Computer and information sciences, FOS: Computer and information sciences},
  title = {A Comprehensive Benchmark for COVID-19 Predictive Modeling Using Electronic Health Records in Intensive Care: Choosing the Best Model for COVID-19 Prognosis},
  publisher = {arXiv},
  year = {2022},
  copyright = {arXiv.org perpetual, non-exclusive license}
}
```

## License

This project is released under the [GPL-2.0 license](LICENSE).
