# TestNet Documentation
Documentation on training nodes and validators


## Table of Contents 

1. [Fed Ledger](#fed-ledger)  
2. [Training Node](#training-node)   
	- [How to participate a LLM training task](#how-to-participate-a-llm-training-task) 
3. [Validator](#validator)  
	- [How to participate a LLM validation task](#how-to-participate-a-llm-validation-task)   

.

## Fed Ledger
## Training Node
### How to participate a LLM training task
To interact with FLock's Fed Ledger service, first obtain an API key from `train.flock.io`.

**Staking Requirement**: To submit a model for a training task, you must stake through the Web UI. Otherwise, Fed Ledger will not accept your submission.

#### Getting Task Details
To get details of a specific task, use the following command:
```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/get?task_id=<task id>'
```

#### Submitting Your Trained Models
After training your model, you need to upload all model files to your public Hugging Face model repository. Then, provide the repo name with the following command:

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

### How to participate a LLM validation task

Similar to the training task, to act as a validator, you must stake for a given task before you can request a submission to validate.

#### Requesting a Validation Assignment
To request a random task submission for validation, use the following command:
```bash
curl --location --request POST 'https://fed-ledger-staging.flock.io/api/v1/tasks/request-validation-assignment/<task id>' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-validator>'
```

In response, you will get an `id` for this validation assignment you are assigned. You need to provide this `id` when you upload the validation result.

#### Submitting Validation Results
Once assigned a validation task, run the validation script to obtain the score, then submit the validation result with the following command:

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

**Submitting a failed result**: If you're unable to complete the validation task, you can manually submit a failed result to Fed Ledger so that you can continue on other validation assignments, with this command:

```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/update-validation-assignment/<assignment id>' \
--header 'flock-api-key: <your-api-key-with-staking-on-this-task-as-validator>' \
--header 'Content-Type: application/json' \
--data '{
    "status": "failed"
}'
```
