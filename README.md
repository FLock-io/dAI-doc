# testnet-doc
Documentation on node and validators

## Fed Ledger
## Training Node
### How to participate a LLM training task
To interact with FLock's Fed Ledger service, you have to obtain API key from `train.flock.io` first.

If you want to submit the model for a given training task, you have to stake through the Web UI for that task, or else Fed Ledger will not accept your submission.

#### Get details of a task

```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/get?task_id=<task id>'
```

#### Submit your trained model
To submit your trained model, you have to upload all model files to your public huggingface model repo first and then provide the repo name.

```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/submit-result' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-node>' \
--header 'Content-Type: application/json' \
--data '{
    "task_id": <task id>,
    "data":{
        "hg_repo_id": "your-hg-name/model-name-7b"
    }
}'
```

## Validator

### How to participate as a LLM training task validator

Similar to training task, you have to stake as a validator for a given task before you can request a task submission to validate on.

#### Request a random task submission for a given task

```bash
curl --location --request POST 'https://fed-ledger-staging.flock.io/api/v1/tasks/request-validation-assignment/<task id>' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-validator>'
```

In response, you will get a `id` for this validation assignment you are assigned. You need to provide this `id` when you upload the validation result.

After you run the validation script and get the score, you can upload the validation result through

```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/update-validation-assignment/<assignment id>' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-validator>' \
--header 'Content-Type: application/json' \
--data '{
    "status": "completed",
    "data": {
        "loss": 0.1
    }
}'
```

If you couldn't complete the validation task for this submission, you can manually submit a failed result to Fed Ledger so that you can continue on other validation assignments.

```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/update-validation-assignment/<assignment id>' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-validator>' \
--header 'Content-Type: application/json' \
--data '{
    "status": "failed"
}'
```

## Troubleshooting

- You should be able to view and generate your API keys at `train.flock.io` once you have staked $FML. If you are not able to do so, please try disconnet your wallet, clear your cookies, and connect with your wallet again. Please also make sure you are connected with the wallet which you used to stake for tasks. 
