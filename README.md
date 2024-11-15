# prompt-challenger-step-functions

Repositório criado para compartilhar exercício com AWS Step Functions.

```json
{
  "Comment": "An example of using Bedrock to chain prompts and their responses together.",
  "StartAt": "Executar primeiro prompt",
  "States": {
    "Executar primeiro prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "cohere.command-text-v14",
        "Body": {
          "prompt.$": "$.prompt_one",
          "max_tokens": 250
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Adicionar o primeiro resultado ao histórico de conversas",
      "ResultPath": "$.result_one",
      "ResultSelector": {
        "result_one.$": "$.Body.generations[0].text"
      }
    },
    "Adicionar o primeiro resultado ao histórico de conversas": {
      "Type": "Pass",
      "Next": "Executar segundo prompt",
      "Parameters": {
        "convo_one.$": "States.Format('{}\n{}', $.prompt_one, $.result_one.result_one)"
      },
      "ResultPath": "$.convo_one"
    },
    "Executar segundo prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "cohere.command-text-v14",
        "Body": {
          "prompt.$": "States.Format('{}\n{}', $.convo_one.convo_one, $.prompt_two)",
          "max_tokens": 200
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Adicionar o segundo resultado ao histórico de conversas",
      "ResultSelector": {
        "result_two.$": "$.Body.generations[0].text"
      },
      "ResultPath": "$.result_two"
    },
    "Adicionar o segundo resultado ao histórico de conversas": {
      "Type": "Pass",
      "Next": "Executar terceiro prompt",
      "Parameters": {
        "convo_two.$": "States.Format('{}\n{}\n{}', $.convo_one.convo_one, $.prompt_two, $.result_two.result_two)"
      },
      "ResultPath": "$.convo_two"
    },
    "Executar terceiro prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "cohere.command-text-v14",
        "Body": {
          "prompt.$": "States.Format('{}\n{}', $.convo_two.convo_two, $.prompt_three)",
          "max_tokens": 1250
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "End": true,
      "ResultSelector": {
        "result_three.$": "$.Body.generations[0].text"
      }
    }
  }
}
```