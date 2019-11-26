1) Create a new bucket
gsutil mb -p tacitknowledge-automation-lab gs://siteops369/

forensic-mc-evidence

2) Create service accounts
gcloud iam service-accounts create siteops369 --display-name "siteops369"

3) Create service accounts keys
gcloud iam service-accounts keys create sa-siteops369.json --iam-account siteops369@tacitknowledge-automation-lab.iam.gserviceaccount.com

4) Give the service account sufficient permission such that it could perform the request that the signed URL will make.
gsutil iam ch serviceAccount:siteops369@tacitknowledge-automation-lab.iam.gserviceaccount.com:objectCreator gs://siteops369
gsutil iam ch serviceAccount:siteops369@tacitknowledge-automation-lab.iam.gserviceaccount.com:objectViewer gs://siteops369

* Storage Object Creator and object Viewer permission granted.

5) gsutil signurl -c 'text/plain' -m RESUMABLE -r us sa-siteops369.json gs://siteops369/file.txt

Good one:   gsutil signurl -m PUT -d 1h -r us -c text/plain sa-siteops369.json gs://siteops369/
            gsutil signurl -c 'text/plain' -d 10m -m PUT -r us-east1 sa-siteops369.json gs://forensic-mc-evidence/

Good one: 
            curl -X PUT --upload-file file.txt "https://storage.googleapis.com/siteops369\?x-goog-signature\=530363cc6596ab6b28cad4f4cc302b097b3df4bc470dfe80f316965bb28a1d8d3de7f3f65849f3dec73792c15601ecc72eb8cd5355ea160ae884762d84ea84e58a91ed060682ee9c8b37fc5a6496cf5c998b6f459dcc2474ba911f634f72ec087bdc7a73070460c91a5bdf91d28c4b42360a570bee7a7fe3fb64ffcc14998fb291d13e6de4ba3f8569f470cfc293039e6ba3070d2afcad7f97b3d48bd5fbef8b913b2361249728c46bb3d785a8f82e6dab8b8288038c3acff64d5c4f854ed5a76970283b565d46178cb402c249d6443e37c841044bfa1f63fd81a3c0c29b29040064ff6939098ed7ec252651544b07a75792d18a6e7a74ad8246c627d9b8fddd\&x-goog-algorithm\=GOOG4-RSA-SHA256\&x-goog-credential\=siteops369%40tacitknowledge-automation-lab.iam.gserviceaccount.com%2F20191114%2Fus%2Fstorage%2Fgoog4_request\&x-goog-date\=20191114T213402Z\&x-goog-expires\=3600\&x-goog-signedheaders\=content-type%3Bhost"

            curl -X PUT --upload-file file.txt "https://storage.googleapis.com/forensic-mc-evidence\?x-goog-signature\=7880323101b5c971866593368c73741e0120c51b02264b50569708bad22c1aa8b9a4acd13d88f4c5cb835c6d636311e8ce61dae587a30aab0f1f39921cbd8effd94d9ae3bb085a7208645de76f563791986ed94fa550de009ac8cac3f4dd704b9d7302b8b1847aa2dc64a178a84387545e6fdc7ddbb16ab45cc6394d6ce3a6691507b24f42843bbe5e269f76ccb6eb68e63573d86a5c65226fd6752e8cf2ae5bc955042eba853db9119ca0d309bad12656457180245caa10d6b6d4cbf8613f3e1f94bf7665635cc783dd8b238fc17e9c2f362069c9bd020a8bd83c76ce76d615697385749b478e9687d2ed48f876bf6b289e5dc744d75fead8521c19ab58de91\&x-goog-algorithm\=GOOG4-RSA-SHA256\&x-goog-credential\=siteops369%40tacitknowledge-automation-lab.iam.gserviceaccount.com%2F20191114%2Fus-east1%2Fstorage%2Fgoog4_request\&x-goog-date\=20191114T221200Z\&x-goog-expires\=600\&x-goog-signedheaders\=content-type%3Bhost"


Error:
<?xml version='1.0' encoding='UTF-8'?><Error><Code>MalformedSecurityHeader</Code><Message>Your request has a malformed header.</Message><ParameterName>content-type</ParameterName><Details>Header was included in signedheaders, but not in the request.</Details></Error>%

Solution:
Missing double quotes to the signurl when issuing the curl command.

Error: 
<?xml version='1.0' encoding='UTF-8'?><Error><Code>InvalidBucketName</Code><Message>The specified bucket is not valid.</Message><Details>Invalid bucket name: 'forensic-mc-evidence\'</Details></Error>%

Error: 
<?xml version='1.0' encoding='UTF-8'?><Error><Code>AccessDenied</Code><Message>Access denied.</Message><Details>Anonymous caller does not have storage.objects.create access to forensic-mc-evidence/\.</Details></Error>%

Solution: 
This happends because the signurl given by gsutil was not properly copied over to curl command. Correct the url and run the curl command again.







Signed URLs = A signed URL is a URL that provides limited permission and time to make a request. Signed URLs contain authentication information in their query string, allowing users without credentials to perform specific actions on a resource. When you generate a signed URL, you specify a user or service account which must have sufficient permission to make the request that the signed URL will make. After you generate a signed URL, anyone who possesses it can use the signed URL to perform specified actions, such as reading an object, within a specified period of time.

Create a Signed URL:
1) Generate a new private key
    Give it "Storage Object Creator" role.
Note: Save the private key on your computer
2) Use the gsutil signurl command to generate the signed url:
gsutil signurl -c 'text/plain' -d 10m -m PUT -r us-east1 tacitknowledge-automation-lab-private-key-siteops369.json gs://test-mc-forensic-info-siteops-396/test.txt



gsutil signurl -d 10m Desktop/private-key.json gs://example-bucket/cat.jpeg

Note: you can get the details of the bucket url from the "overview" section of the bucket itself.

Where:

-d Specifies the duration that the signed url should be valid for, default duration is 1 hour. Times may be specified with no suffix (default hours), or with s = seconds, m = minutes, h = hours, d = days.
-m Specifies the HTTP method to be authorized for use with the signed url, default is GET.
-r specifies the region (I was asked to enter it when running the command wihout it. )


URL resultante:

URL	HTTP Method	Expiration	Signed URL
gs://test-mc-forensic-info-siteops-396/	PUT	2019-09-30 18:01:05	https://storage.googleapis.com/test-mc-forensic-info-siteops-396?x-goog-signature=1ecb184391c788418b9278a58976a7222a9554f5841d7cebda31a888b218e1c6f0e1f71f290ed56cd8e5f5fc4b7f9b01cede9490bfe2cf3943f6fee2607a15ae8ee222978e4df8be10d8e63edf2e137d44b62142d6ea177c3016cebe542a3c7da8fe4a77e614ea01e201b09360fd1b28a99e44ca3bc390abd45a0b880f7e61e7ea2440c1c3096e0d5a673981b64af7a64805c018d436910cb65b6cdec9f75782afe80e92982d4b4d9ea07188122f9cfc8a0dbc2b38c0204b0fcaf086f7bcba2b53b0d6466cda68fb7aa5fe98c512410af07fbb00dddaeb6dcc7c13e9241297c725c8f7f81441af4b1ed8d79633f8c78a97ff50a6d81048c56f753957baa2e102&x-goog-algorithm=GOOG4-RSA-SHA256&x-goog-credential=foresic-info%40tacitknowledge-automation-lab.iam.gserviceaccount.com%2F20190930%2Fus-east1%2Fstorage%2Fgoog4_request&x-goog-date=20190930T220105Z&x-goog-expires=3600&x-goog-signedheaders=host


Upload an object:

curl -X PUT -H 'content-type: text/plain' -d @test.txt https://storage.googleapis.com/test-mc-forensic-info-siteops-396?x-goog-signature=aecb9b2d3e232149eae1a0b70a1b40e0fd6fca3fd0dfd43de0edb42c41c8dab65df0d6ab079dca750de0bd3b1c3868516bd582b55350b8896c2aae9e7d9132b9799d250c3abde7ea6c588c273c40931992a17e2dd9503f02eb5c4561f6e826137e80d347b16cf4db58fa8245305b99f36cfe6ffb9605ed68219d863626c6b90b4c257f560d11f75288fcca2cf91b45ed7bb58ee90ebcb24c51ee2a815f81b16af78c1d44e527bb648bf2a89b49151548cffb8c46c1b5562fdb361bccbf29ff61a89e7bb0b37aa192717bd8828422ed73444a893351204f4ed8288b2b3039fbb1ca6078e0ac34b6ae2884bd43e16e8151acf6c971a65ebf21278d23b2e0747545&x-goog-algorithm=GOOG4-RSA-SHA256&x-goog-credential=foresic-info%40tacitknowledge-automation-lab.iam.gserviceaccount.com%2F20190930%2Fus-east1%2Fstorage%2Fgoog4_request&x-goog-date=20190930T221145Z&x-goog-expires=3600&x-goog-signedheaders=content-type%3Bhost

Where:

-X = Specifies a custom request method to use when communicating with the HTTP server
-H = Extra header to include in the request when sending HTTP to a server.
-d = Sends the specified data in a POST request to the HTTP server, in the same way that a browser does when a user has filled in an HTML form and presses  the submit button. This will cause curl to pass the data to the server using the content-type application/x-www-form-urlencoded



https://storage.googleapis.com/test-mc-forensic-info-siteops-396?x-goog-signature=aecb9b2d3e232149eae1a0b70a1b40e0fd6fca3fd0dfd43de0edb42c41c8dab65df0d6ab079dca750de0bd3b1c3868516bd582b55350b8896c2aae9e7d9132b9799d250c3abde7ea6c588c273c40931992a17e2dd9503f02eb5c4561f6e826137e80d347b16cf4db58fa8245305b99f36cfe6ffb9605ed68219d863626c6b90b4c257f560d11f75288fcca2cf91b45ed7bb58ee90ebcb24c51ee2a815f81b16af78c1d44e527bb648bf2a89b49151548cffb8c46c1b5562fdb361bccbf29ff61a89e7bb0b37aa192717bd8828422ed73444a893351204f4ed8288b2b3039fbb1ca6078e0ac34b6ae2884bd43e16e8151acf6c971a65ebf21278d23b2e0747545&x-goog-algorithm=GOOG4-RSA-SHA256&x-goog-credential=foresic-info%40tacitknowledge-automation-lab.iam.gserviceaccount.com%2F20190930%2Fus-east1%2Fstorage%2Fgoog4_request&x-goog-date=20190930T221145Z&x-goog-expires=3600&x-goog-signedheaders=content-type%3Bhost



 https://storage.googleapis.com/test_sl?GoogleAccessId=my-project-id@appspot.gserviceaccount.com&Expires=1490266627&Signature=UfKBNHWtjLKSBEcUQUKDeQtSQV6YCleE9hGG%2BCxVEjDOmkDxwkC%2BPtEg63pjDBHyKhVOnhspP1%2FAVSr%2B%2Fty8Ps7MSQ0lM2YHkbPeqjTiUcAfsbdcuXUMbe3p8FysRUFMe2dSikehBJWtbYtjb%2BNCw3L09c7fLFyAoJafIcnoIz7iJGP%2Br6gAUkSnZXgbVjr6wjN%2FIaudXIqAxyz4O6VU%2FIob8RHmfYBD3rYw8oRmZer0g%2BMTDw5Bu3%2BdjbxCriCOkOLWkvlpBGvd71hRP89lxpD%2FwX1N1pzAiRBpmKAhpKrYrq8h4kNLAtTHXYSz5j%2Bzb28UD4YMG52n8UpfyPgFIg%3D%3D  


References:
https://cloud.google.com/storage/docs/access-control/signed-urls
https://cloud.google.com/storage/docs/access-control/signing-urls-with-helpers
http://opreview.blogspot.com/2017/03/how-to-upload-to-google-cloud-storage.html
https://cloud.google.com/iam/docs/creating-managing-service-account-keys#creating_service_account_keys