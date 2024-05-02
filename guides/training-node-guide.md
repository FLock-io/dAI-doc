# Training Node Guide

## Task Details

```json
{
    "title": "test task for matt",
    "description": "gaming character",
    "task_type": "training",
    "data": {
        "training_set_url": "https://flock-fl-param.s3.amazonaws.com/26/training_set.jsonl?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIASSFQ745NHQLBLUN4%2F20240429%2Fus-east-2%2Fs3%2Faws4_request&X-Amz-Date=20240429T192202Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=efc550f4528b57a6742bea6c9df5ffea32bd3414f60223baf3dcb2d241074343",
        "max_params": 3000000000,
        "context_length": 512
    },
    "duration_in_seconds": 86400,
    "id": 26,
    "status": "submission_phase"
}
```

Above is a sample response from the endpoint. You need to make sure that `status` of the task is still in `submission_phase` so that you are able to submit a model for this task.

Following is the explanantion on the fields in `data` key:

* `training_set_url` is a link for you to download the training dataset for the task. It always follows a pre-defined format. A sample can be found [Dummy Data](https://github.com/FLock-io/llm-loss-validator/blob/main/src/data/dummy_data.jsonl).

* `max_params` is the maximum number of parameters of the LLM you submitted.

* `context_length` is the context length that your submitted model should support. A larger context length will also significantly increase the VRAM required to finetune the model.

## Supported Base Models

In order to make sure validators can validate the submitted models successfully, we only accept a pre-defined set of base models. You can find the supported ones here: [Supported Base LLM](https://github.com/FLock-io/llm-loss-validator/blob/main/src/core/constant.py)

## How to train the model

FLock will not limit your options or technique to train a model. Basically you only need to make sure that your submitted model can be successfully ran by our [Validation Script](https://github.com/FLock-io/llm-loss-validator/tree/main). The easiest way to verify that is using the following command

```bash
cd /src
CUDA_VISIBILE_DEVICES=0 FLOCK_API_KEY="<your-api-key>" python validate.py \
--model_name_or_path <hg_repo_id> \
--template_name <base_model> \
--eval_file ./data/dummy_data.jsonl \
--max_seq_length 128 \
--local_test \
--validation_args_file validation_config.json.example
```

For actually fine-tuning, we recommand some well-known framework like [llama factory](https://github.com/hiyouga/LLaMA-Factory) and [axolotl](https://github.com/OpenAccess-AI-Collective/axolotl)

## How to submit a model

```bash
curl --location 'https://fed-ledger-staging.flock.io/api/v1/tasks/submit-result' \
--header 'flock-api-key: 9TUIMYK1E2JOXSF7SY7QGIWW74WS3ZX0' \
--header 'Content-Type: application/json' \
--data '{
    "task_id": 23,
    "data":{
        "hg_repo_id": "your-hg-id/repo-name",
        "base_model": "mistral"
    }
}'
```

Above is a sample request to submit a model for a given task. You need to upload all your model params (i.e. safetensor files and other relevant config/tokenizer files) to your huggingface model repo. Make sure the repo is public accessible so that validators are able to download them.

## Rate Limiting

We have enbaled a fixed submission rate limiting for training nodes. You are only allowed to submit models twice a day.
