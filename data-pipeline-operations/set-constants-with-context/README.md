---
description: Learn how to set constants with the Context data operation.
---

# Set constants with Context

## :bulb:What is Context?

Before getting started with the creation of your data pipeline operations, you can first set constants using a Context data operation. Constants are objects which act as placeholders. Their value is stored through a Context data operation, and can be called in all your subsequent data operations.

Constants can be used for example to replace GCP credential parameters, so they can remain confidential, GCS bucket names, account numbers, etc.

## ✅ Supported data operations

All data operations can contain the constants set with Context.

{% hint style="warning" %}
For  STS, STT, TTT and TTS data operations, you will need to add`"version" : "2"`in the JSON configuration file to be able to use constants.
{% endhint %}

## ⚙️ How it works

You set a constant as follows through a Context data operation:

```
"test_integer": {
			"value": 123456,
			"type": "integer",
			"resource": "value",
			"description": "Some random value"
		}
```

Once you have deployed your Context data operation, you can start using the constants it contains in your other data operations with the following syntax:

```
{% myConstant %}
```

## **📋 How to set and use a constant**

1. Access your **tailer** folder (created during [installation](../../getting-started/install-tailer-sdk.md)).
2. Create a working folder as you want, and create a JSON file for your Context data operation inside.
3. Set all the required constants in your JSON configuration file. Refer to this page to learn about all the [parameters](context-configuration-file.md).
4.  Access your working folder by running the following command:

    ```
    cd "[path to your working folder]"
    ```
5.  To deploy the Context data operation, run the following command:

    ```
    tailer deploy configuration your-file.json
    ```
6. Log in to [Tailer Studio](http://studio.tailer.ai) to check the status and details of your data operation.
7. You can now use your constants in other data operation configurations files using the `{% myConstant %}` syntax. Dont forget to add `"version" : "2"` for STS, STT, TTT and TTS data operations.
