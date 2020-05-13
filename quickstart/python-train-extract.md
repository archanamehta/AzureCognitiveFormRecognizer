# Quickstart: Train a Form Recognizer model and extract form data by using the REST API with Python

In this quickstart, you'll use the Azure Form Recognizer REST API with Python to train and score forms to extract key-value pairs and tables.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites

To complete this quickstart, you must have:
- [Python](https://www.python.org/downloads/) installed (if you want to run the sample locally).
- A set of at least five forms of the same type. You will use this data to train the model. Your forms can be of different file types but must be the same type of document. You can use a [sample data set](https://go.microsoft.com/fwlink/?linkid=2090451) for this quickstart. Upload the training files to the root of a blob storage container in an Azure Storage account.

## Create a Form Recognizer resource

[!INCLUDE [create resource](../includes/createresource.md)]


## Train a Form Recognizer model

   First, you'll need a set of training data in an Azure Storage blob container. You should have a minimum of five filled-in forms (PDF documents and/or images) of the same type/structure as your main input data. 

To train a Form Recognizer model with the documents in your Azure blob container, call the **[Train Custom Model](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-preview/operations/TrainCustomModelAsync)** API by running the following python code. Before you run the code, make these changes:

1. Replace `<SAS URL>` with the Azure Blob storage container's shared access signature (SAS) URL. To retrieve the SAS URL, open the Microsoft Azure Storage Explorer, right-click your container, and select **Get shared access signature**. Make sure the **Read** and **List** permissions are checked, and click **Create**. Then copy the value in the **URL** section. It should have the form: `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
1. Replace `<subscription key>` with the subscription key you copied from the previous step.
1. Replace `<endpoint>` with the endpoint URL for your Form Recognizer resource.
1. Replace `<Blob folder name>` with the path to the folder in blob storage where your forms are located. If your forms are at the root of your container, leave this string empty.

    ```python
    ########### Python Form Recognizer Labeled Async Train #############
    import json
    import time
    from requests import get, post
    
    # Endpoint URL
    endpoint = r"https://archie-fr.cognitiveservices.azure.com/"
    post_url = endpoint + r"/formrecognizer/v2.0-preview/custom/models"
    source = r"<SAS URL>"
    prefix = "<Blob folder name>"
    includeSubFolders = False
    useLabelFile = False
    
    headers = {
        # Request headers
        'Content-Type': 'application/json',
        'Ocp-Apim-Subscription-Key': '<subsription key>',
    }
    
    body = 	{
        "source": source,
        "sourceFilter": {
            "prefix": prefix,
            "includeSubFolders": includeSubFolders
        },
        "useLabelFile": useLabelFile
    }
    
    try:
        resp = post(url = post_url, json = body, headers = headers)
        if resp.status_code != 201:
            print("POST model failed (%s):\n%s" % (resp.status_code, json.dumps(resp.json())))
            quit()
        print("POST model succeeded:\n%s" % resp.headers)
        get_url = resp.headers["location"]
    except Exception as e:
        print("POST model failed:\n%s" % str(e))
        quit() 
    ```
1. Save the code in a file with a .py extension. For example, *form-recognizer-train.py*.
1. Open a command prompt window.
1. At the prompt, use the `python` command to run the sample. For example, `python form-recognizer-train.py`.

## Ouput of Executing the above Pythong Code ie: form-recognizer-train.py is as follows 

    ```json
      POST model succeeded:
      {
        'Content-Length': '0',
        'Location': 'https://archie-fr.cognitiveservices.azure.com/formrecognizer/v2.0-preview/custom/models/e16064cc-525e-49cf-ad08-9c594d792507',
        'x-envoy-upstream-service-time': '30',
        'apim-request-id': 'b812f718-b56b-4b14-bd8f-82d9b597e988',
        'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload',
        'x-content-type-options': 'nosniff',
        'Date': 'Wed, 13 May 2020 16:27:48 GMT'
      }
       ```


## Get training results

After you've started the train operation, you use the returned ID to get the status of the operation. Add the following code to the bottom of your Python script. This uses the ID value from the training call in a new API call. The training operation is asynchronous, so this script calls the API at regular intervals until the training status is completed. We recommend an interval of one second or more.

```python 
n_tries = 15
n_try = 0
wait_sec = 5
max_wait_sec = 60
while n_try < n_tries:
    try:
        resp = get(url = get_url, headers = headers)
        resp_json = resp.json()
        if resp.status_code != 200:
            print("GET model failed (%s):\n%s" % (resp.status_code, json.dumps(resp_json)))
            quit()
        model_status = resp_json["modelInfo"]["status"]
        if model_status == "ready":
            print("Training succeeded:\n%s" % json.dumps(resp_json))
            quit()
        if model_status == "invalid":
            print("Training failed. Model is invalid:\n%s" % json.dumps(resp_json))
            quit()
        # Training still running. Wait and retry.
        time.sleep(wait_sec)
        n_try += 1
        wait_sec = min(2*wait_sec, max_wait_sec)     
    except Exception as e:
        msg = "GET model failed:\n%s" % str(e)
        print(msg)
        quit()
print("Train operation did not complete within the allocated time.")
```

Output of the Above Code to get the Training Results is as follows:

```json
Training succeeded:
{
  "modelInfo": {
    "modelId": "e16064cc-525e-49cf-ad08-9c594d792507",
    "status": "ready",
    "createdDateTime": "2020-05-13T16:27:49Z",
    "lastUpdatedDateTime": "2020-05-13T16:27:56Z"
  },
  "trainResult": {
    "trainingDocuments": [
      {
        "documentName": "Invoice_1.pdf",
        "pages": 1,
        "errors": [
          
        ],
        "status": "succeeded"
      },
      {
        "documentName": "Invoice_2.pdf",
        "pages": 1,
        "errors": [
          
        ],
        "status": "succeeded"
      },
      {
        "documentName": "Invoice_3.pdf",
        "pages": 1,
        "errors": [
          
        ],
        "status": "succeeded"
      },
      {
        "documentName": "Invoice_4.pdf",
        "pages": 1,
        "errors": [
          
        ],
        "status": "succeeded"
      },
      {
        "documentName": "Invoice_5.pdf",
        "pages": 1,
        "errors": [
          
        ],
        "status": "succeeded"
      }
    ],
    "errors": [
      
    ]
  }
}
```

Copy the `"modelId"` value for use in the following steps.

## Analyze the Results 
Once the Training of the Model has been complete we need to Analyze results. When the process is completed, you'll receive a `200 (Success)` response with JSON content in the following format. The response has been shortened for simplicity. The main key/value pair associations and tables are in the `"pageResults"` node. If you also specified plain text extraction through the *includeTextDetails* URL parameter, then the `"readResults"` node will show the content and positions of all the text in the document.

```bash
POST analyzesucceeded: {
  'Content-Length': '0',
  'Operation-Location': 'https://archie-fr.cognitiveservices.azure.com/formrecognizer/v2.0-preview/custom/models/009ae39c-9742-4086-aa21-d6c9881b9879/analyzeresults/be6a72bb-90fc-4211-a65c-3d16b01d2879',
  'x-envoy-upstream-service-time': '87',
  'apim-request-id': 'fe81da67-997a-445d-bb8d-01fae29cca55',
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload',
  'x-content-type-options': 'nosniff',
  'Date': 'Wed, 13 May 2020 16:35:51 GMT'
}

{
  "status": "succeeded",
  "createdDateTime": "2020-05-13T16:35:52Z",
  "lastUpdatedDateTime": "2020-05-13T16:35:53Z",
  "analyzeResult": {
    "version": "2.0.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11.0,
        "unit": "inch",
        "lines": [
          {
            "text": "Contoso",
            "boundingBox": [
              0.5278,
              1.0597,
              1.4569,
              1.0597,
              1.4569,
              1.4347,
              0.5278,
              1.4347
            ],
            "words": [
              {
                "text": "Contoso",
                "boundingBox": [
                  0.5278,
                  1.0597,
                  1.4569,
                  1.0597,
                  1.4569,
                  1.4347,
                  0.5278,
                  1.4347
                ]
              }
            ]
          },
          {
            "text": "Address:",
            "boundingBox": [
              0.7972,
              1.5125,
              1.3958,
              1.5125,
              1.3958,
              1.6431,
              0.7972,
              1.6431
            ],
            "words": [
              {
                "text": "Address:",
                "boundingBox": [
                  0.7972,
                  1.5125,
                  1.3958,
                  1.5125,
                  1.3958,
                  1.6431,
                  0.7972,
                  1.6431
                ]
              }
            ]
          },
    
    more more more Json  
```
    
## Improve results
Examine the "confidence" values for each key/value result under the "pageResults" node. You should also look at the confidence scores in the "readResults" node, which correspond to the text read operation. The confidence of the read results does not affect the confidence of the key/value extraction results, so you should check both.

If the confidence scores for the read operation are low, try to improve the quality of your input documents (see Input requirements).
If the confidence scores for the key/value extraction operation are low, ensure that the documents being analyzed are of the same type as documents used in the training set. If the documents in the training set have variations in appearance, consider splitting them into different folders and training one model for each variation.
The confidence scores you target will depend on your use case, but generally it's a good practice to target a score of 80% or above. For more sensitive cases, like reading medical records or billing statements, a score of 100% is recommended.


### Custom model
Form Recognizer works on input documents that meet these requirements:

      1. Format must be JPG, PNG, PDF (text or scanned), or TIFF. Text-embedded PDFs are best because there's no possibility of error in character extraction and location.
      2. If your PDFs are password-locked, you must remove the lock before submitting them.
      3. PDF and TIFF documents must be 200 pages or less, and the total size of the training data set must be 500 pages or less.
      4. For images, dimensions must be between 600 x 100 pixels and 4200 x 4200 pixels.
      5. If scanned from paper documents, forms should be high-quality scans.
      6. Text must use the Latin alphabet (English characters).
      7. For unsupervised learning (without labeled data), data must contain keys and values.
      8. For unsupervised learning (without labeled data), keys must appear above or to the left of the values; they can't appear below or to the right.
      9. Form Recognizer doesn't currently support these types of input data:
            * Complex tables (nested tables, merged headers or cells, and so on).
            * Checkboxes or radio buttons.

### Prebuilt receipt model
The input requirements for the receipt model are slightly different.

      Format must be JPEG, PNG, PDF (text or scanned) or TIFF.
      File size must be less than 20 MB.
      Image dimensions must be between 50 x 50 pixels and 10000 x 10000 pixels.
      PDF dimensions must be at most 17 x 17 inches, corresponding to Legal or A3 paper sizes and smaller.
      For PDF and TIFF, only the first 200 pages are processed (with a free tier subscription, only the first two pages are processed).


## Next steps
In this quickstart, you used the Form Recognizer REST API with Python to train a model and run it in a sample scenario. Next, see the reference documentation to explore the Form Recognizer API in more depth.

> [REST API reference documentation](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-preview/operations/AnalyzeWithCustomForm)
