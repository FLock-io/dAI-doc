# Pre-Requisites

- Web3 Wallet - You will use this to connect to the web app at train.flock.io

- $FML - FLock’s testnet tokens required for staking. You can check out the contract [here](https://sepolia.basescan.org/address/0xeF39985160FEdd389B46E6dB14d6aEa26cb5981A\)). You should first get whitelisted [here](https://blog.flock.io/news/trainflock), then obatin $FML at our faucet [here](https://train.flock.io/faucet).


- Base Sepolia ETH - FLock.io testnet is deployed on Base Sepolia. You will need Base Sepolia ETH for gas to execute the staking transactions.

  - If you don’t have Base Sepolia ETH, you can get some from the [Alchemy Faucet](https://www.alchemy.com/faucets/base-sepolia).


# Validator Guide

This guide provides step-by-step instructions for the entire Validator workflow. By the end you will have successfully staked as a Validator, requested task assignment(s), run a validator script, uploaded loss score(s), and claimed your staking rewards.


## 1. Stake FML as Validator from [web app](http://train.flock.io)
In order begin receiving validation assignments, you must stake FML as to a task at train.flock.io.

1. Navigate to the Validator tab on the Stake page

2. Select task to validate

3. Enter number of FML tokens to stake

4. Stake FML

![Image Alt Text](assets/val1_stake-fml.png)

Once you’ve confirmed and approved the transaction in your web3 wallet, you will see a box on the Validator tab with your stake details.

![Image Alt Text](assets/val1.1_tx-confirmation.png)


## 2. Get API key
Your API key is required for all remaining Validator steps. You can get it from the web app.

1. Select dropdown in upper right corner of web app

2. Select API

3. On the API page, copy your API key

![Image Alt Text](assets/val2_api-key.png)
![Image Alt Text](assets/val2.1_copy-api.png)

Once you have your API key, you can proceed to the next step.

## 3. Request validation assignment for task from CLI

Once you’ve staked FML for a task, you can request a random validation assignment for the given task.

1. Copy the following command into your terminal

```bash

 curl --location --request POST 'https://fed-ledger-staging.flock.io/api/v1/tasks/request-validation-assignment/<task id>' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-validator>'

```

2. Update `<task id>` with the Validator task you are staking

3. Update `<your-api-key-with-staking-on-this-task-as-validator>` with your API key

4. Run the command

If the command ran successfully, you will receive a response like this:

```bash
{"id":1,"task_submission":{"task_id":23,"data":{"hg_repo_id":"Nicoooolasweee/qwen-1.5-4b-function","base_model":"qwen1.5"},"id":11},"assigned_at":"2024-04-24T18:21:45.047503","status":"validating","data":{"base_model":"qwen1.5","context_length":512,"validation_set_url":"https://flock-fl-param.s3.amazonaws.com/23/validation_set.jsonl?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIASSFQ745NHQLBLUN4%2F20240424%2Fus-east-2%2Fs3%2Faws4_request&X-Amz-Date=20240424T182145Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=25c80fb4d22fac6760fe5821ff2dd768b1678ba930a28e8521cb3e834ca346c1"}}
```

You will need the following values from the response body:

- The first `id` key value: This is your assignment id for this task. You will need this to upload your validation score.

- `"base_model":"qwen1.5"`: This is interpreted for the templates parameter.


## 4. Set up your environment

In order to run the validation script to calculate loss and submit the score you need to set up your environment. 

We recommend using `conda` to manage the Python environment for this repo. You can do so running the following commands in your terminal:

```bash
conda create -n llm-loss-validator python==3.10.12
conda activate llm-loss-validator
pip install -r requirements.txt
```

## 5. Run validation script

After receiving a validation assignment (see Step 3) you must run the validation script in your environment (see Step 4). This script first calculates a loss score then automatically submits it. You will need 

1.  Paste the following command into your terminal

```bash
FLOCK_API_KEY="<your-api-key>" python validate.py \
--model_name_or_path <hg_repo_id> \
--base_model <base_model> \
--eval_file ./data/dummy_data.jsonl \
--context_length 128 \
--assignment_id <assignment-id> \
--validation_args_file <validation_config.json.example OR validation_config_cpu.json.example>
```

2. Replace `<your-api-key> `with your API key 

3. Replace `<hg_repo_id> `with the corresponding value from the response body in Step 3

4. Replace `<base_model>` with the corresponding value from the response body in Step 3

5. For `<eval_file>`, please navigate to the url in `validation_set_url` in the response body in Step 3 and download the file. `<eval_file>` should be pointing to where you save this file. Note that this link expires in 60 minutes so be sure to download it in time.

6. Replace `<assignment-id> `with the `<id> `value from the response body in Step 3

7. For the flag, `validation_args_file `use:

   1. `validation_config.json.example` to run on GPU

   2. `Validation_config_cpu.json.example `to run on CPU

8. Go to https://github.com/FLock-io/llm-loss-validator, git clone the repository and run the validation script from there.

You should get a response similar to the following:

```bash
evaluate result is {'eval_loss': 1.5328044891357422, 'eval_runtime': 1091.4278, 'eval_samples_per_second': 0.916, 'eval_steps_per_second': 0.458}
```

The `eval_loss` value is the loss score you’ve submitted for this validation assignment.

_NOTE: If you couldn't complete the validation task for this submission, you can manually submit a failed result to Fed Ledger so that you can continue on other validation assignments._

```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/update-validation-assignment/<assignment id>' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-validator>' \
--header 'Content-Type: application/json' \
--data '{
    "status": "failed"
}'
```


## 6. Claim rewards

Reward distribution is triggered every 24 hours at midnight UTC. You can claim your rewards via the web app.

Once you’ve connected your web3 wallet to train.flock.io, you can claim rewards with the following steps:

1. Navigate to the Validator tab on the Stake page

2. Verify you have FML available to claim for the task you are staking

3. Click Claim FML

![Image Alt Text](assets/val6_claim-fml.png)

Once you’ve approved the transaction in your web3 wallet you will see a dialogue box confirming your claim.

![Image Alt Text](assets/val6.1_claim-confirmation.png)


Congratulations! You’ve successfully completed the entire validator workflow.


