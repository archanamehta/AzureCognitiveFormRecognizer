# Quickstart: Extract text and layout information using the Form Recognizer REST API with Python

In this quickstart, you'll use the Azure Form Recognizer REST API with Python to extract text layout information and table data from form documents.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites
To complete this quickstart, you must have:
- [Python](https://www.python.org/downloads/) installed (if you want to run the sample locally).
- A form document. You can download an image from the [sample data set](https://go.microsoft.com/fwlink/?linkid=2090451) for this quickstart.

## Create a Form Recognizer resource
[!INCLUDE [create resource](../includes/createresource.md)]


## Analyze the form layout

To start analyzing the layout, you call the **[Analyze Layout](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-preview/operations/AnalyzeLayoutAsync)** API using the Python script below. Before you run the script, make these changes:

1. Replace `<Endpoint>` with the endpoint that you obtained with your Form Recognizer subscription.
1. Replace `<path to your form>` with the path to your local form document.
1. Replace `<subscription key>` with the subscription key you copied from the previous step.

    ```python
    #Quickstart: Extract text and layout information using the Form Recognizer REST API with Python
    import json
    import time
    from requests import get, post

    # Endpoint URL
    endpoint = r"https://archie-fr.cognitiveservices.azure.com/"
    apim_key = "79442bf0df0a4ab894f2b186a0317cae"
    model_id = "009ae39c-9742-4086-aa21-d6c9881b9879"
    post_url = endpoint + "/formrecognizer/v2.0-preview/Layout/analyze"
    source = r"Invoice_1.pdf"
    params = {
        "includeTextDetails": True
    }

    headers = {
        # Request headers
        'Content-Type': 'application/pdf',
        'Ocp-Apim-Subscription-Key': apim_key,
    }
    with open(source, "rb") as f:
        data_bytes = f.read()

    try:
        resp = post(url = post_url, data = data_bytes, headers = headers)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % resp.text)
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit()
        
    ```

1. Save the code in a file with a .py extension. For example, *form-recognizer-layout.py*.
1. Open a command prompt window.
1. At the prompt, use the `python` command to run the sample. For example, `python form-recognizer-layout.py`.

You'll receive a `202 (Success)` response that includes an **Operation-Location** header, which the script will print to the console. This header contains an operation ID that you can use to query the status of the asynchronous operation and get the results. In the following example value, the string after `operations/` is the operation ID.

```console
https://archie-fr.cognitiveservices.azure.com/formrecognizer/v2.0-preview/layout/analyzeResults/c26fe8cc-3e6f-48fc-979d-cc0e70fa8d11
```

## Results of executing form-recognizer-layout.py

    POSTanalyzesucceeded: {
      'Content-Length': '0',
      'Operation-Location': 'https://archie-fr.cognitiveservices.azure.com/formrecognizer/v2.0-preview/layout/analyzeResults/c26fe8cc-3e6f-48fc-979d-cc0e70fa8d11',
      'x-envoy-upstream-service-time': '137',
      'apim-request-id': 'c26fe8cc-3e6f-48fc-979d-cc0e70fa8d11',
      'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload',
      'x-content-type-options': 'nosniff',
      'Date': 'Wed, 13 May 2020 18:23:58 GMT'
    }


## Get the layout results
After you've called the **Analyze Layout** API, you call the **[Get Analyze Layout Result](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-preview/operations/GetAnalyzeLayoutResult)** API to get the status of the operation and the extracted data. Add the following code to the bottom of your Python script. This code uses the operation ID value in a new API call. 

Append the following Python code to the 'form-recognizer-layout.py'

```python
n_tries = 10
n_try = 0
wait_sec = 6
while n_try < n_tries:
    try:
        resp = get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": apim_key})
        resp_json = json.loads(resp.text)
        if resp.status_code != 200:
            print("GET Layout results failed:\n%s" % resp_json)
            quit()
        status = resp_json["status"]
        if status == "succeeded":
            print("Layout Analysis succeeded:\n%s" % resp_json)
            quit()
        if status == "failed":
            print("Layout Analysis failed:\n%s" % resp_json)
            quit()
        # Analysis still running. Wait and retry.
        time.sleep(wait_sec)
        n_try += 1     
    except Exception as e:
        msg = "GET analyze results failed:\n%s" % str(e)
        print(msg)
        quit()
```

1. Save the script.
1. Again use the `python` command to run the sample. For example, `python form-recognizer-layout.py`.


### Examine the response

The script will print responses to the console until the **Analyze Layout** operation completes. Then, it will print the extracted data in JSON format. The `"readResults"` node contains every line of text with its respective bounding box placement on the page. The `"pageResults"` field shows every piece of text within tables, each with its row-column coordinate.

See the following invoice image and its corresponding JSON output. The output has been shortened for simplicity.

> ![Contoso invoice document with a table](../media/contoso-invoice.png)


```json
LayoutAnalysissucceeded: {
  'status': 'succeeded',
  'createdDateTime': '2020-05-13T18:23:59Z',
  'lastUpdatedDateTime': '2020-05-13T18:24:04Z',
  'analyzeResult': {
    'version': '2.0.0',
    'readResults': [
      {
        'page': 1,
        'language': 'en',
        'angle': 0,
        'width': 8.5,
        'height': 11,
        'unit': 'inch',
        'lines': [
          {
            'boundingBox': [
              0.5384,
              1.1583,
              1.4466,
              1.1583,
              1.4466,
              1.3534,
              0.5384,
              1.3534
            ],
            'text': 'Contoso',
            'words': [
              {
                'boundingBox': [
                  0.5384,
                  1.1583,
                  1.4466,
                  1.1583,
                  1.4466,
                  1.3534,
                  0.5384,
                  1.3534
                ],
                'text': 'Contoso',
                'confidence': 1
              }
            ]
          },
          {
            'boundingBox': [
              0.7994,
              1.5143,
              1.3836,
              1.5143,
              1.3836,
              1.6154,
              0.7994,
              1.6154
            ],
            'text': 'Address:',
            'words': [
              {
                'boundingBox': [
                  0.7994,
                  1.5143,
                  1.3836,
                  1.5143,
                  1.3836,
                  1.6154,
                  0.7994,
                  1.6154
                ],
                'text': 'Address:',
                'confidence': 1
              }
            ]
          },
          {
            'boundingBox': [
              4.4033,
              1.5114,
              5.8155,
              1.5114,
              5.8155,
              1.6155,
              4.4033,
              1.6155
            ],
            'text': 'Invoice For: Microsoft',
            'words': [
              {
                'boundingBox': [
                  4.4033,
                  1.5143,
                  4.8234,
                  1.5143,
                  4.8234,
                  1.6155,
                  4.4033,
                  1.6155
                ],
                'text': 'Invoice',
                'confidence': 1
              },
              {
                'boundingBox': [
                  4.8793,
                  1.5143,
                  5.1013,
                  1.5143,
                  5.1013,
                  1.6154,
                  4.8793,
                  1.6154
                ],
                'text': 'For:',
                'confidence': 1
              },
              {
                'boundingBox': [
                  5.2045,
                  1.5114,
                  5.8155,
                  1.5114,
                  5.8155,
                  1.6151,
                  5.2045,
                  1.6151
                ],
                'text': 'Microsoft',
                'confidence': 1
              }
            ]
          },
          {
            'boundingBox': [
              0.8106,
              1.7033,
              2.1445,
              1.7033,
              2.1445,
              1.8342,
              0.8106,
              1.8342
            ],
            'text': '1 Redmond way Suite',
            'words': [
            ..............
            ...............
```


## Next steps
In this quickstart, you used the Form Recognizer REST API with Python to extract the text layout of an invoice. Next, see the reference documentation to explore the Form Recognizer API in more depth.

> [!div class="nextstepaction"]
> [REST API reference documentation](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-preview/operations/AnalyzeLayoutAsync)
