# Image Processing / BOL Image Approval

For various loads, drivers upload BOL documents that we can use to automatically verify with image processing
software. This saves our Billing department a huge amount of time, as otherwise they have to go through and manually
veryify these documents before being able to pay out the drivers.

## Moving Parts

1. [DocumentAI](#documentai)
1. [Code](#code)

## DocumentAI

GCP's DocumentAI allows us to train a model at pulling arbitraty fields out of document images. The models we use can
be found [here](https://console.cloud.google.com/ai/document-ai/locations/us/processors?project=lohi-project-546094).

These models are trained on input images go guide correctness and only require that we tell DocumentAI which model to use when uploading an image. The call will then return a breakdown that we can pull all the needed fields out to verify the image, after which we can approve the upload automatically.

If an image fails to be automatically verified, it is just left to be manually verified.

## Code

* the `src/otr/lohiLoadManager` package has a `RegisterLoadFileProcessor(...)` function that allows the developers to register a function to be called any time a file is uploaded to a Load.
* The registered function will be called asyncronously with various data round the uploaded file, including the contents.
* While the main use case for this code path is auto-approval, it could be used to do anything asynchronously with uploaded images.