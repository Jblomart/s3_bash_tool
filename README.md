# s3_bash_tool

Interract with S3 buckets using bash and limited dependencies (openssl, curl, awk, dd, tee).

## command line arguments : 

```bash
$ ./s3_bash_tool --help
s3_bash_tool -- Tool to interract with S3 storage with limited dependencies (dd,awk,openssl,curl and tee).
------------------------------------------------------------------------------------------------------------
required arguments :
  --url         http url for the s3 service (http[s]://<dns name or ip address of the s3 service>[:<port>]).
  --bucket      name of the bucket we will work with.
  --access_key  s3 access key for authentication.
  --secret_key  s3 secret key for authentication.
  --action      action to perform, can be list,get,put,cancelmultipart,delete,test
optional arguments :
  --region      region where the bucket is hosted, defaults to us-west-1.
  --proxy       proxy to use when sending request to the s3 url.
  --debug       show debug information.

actions arguments :
  "get" action : outputs the object content.
  required arguments :
    --s3path    path of the object in S3 (aka object key).

  "list" action : show bucket content (simple xml format).
  no required arguments.

  "put" action : upload file content to s3 object.
  required arguments :
    --s3path    path of the object in S3 (aka object key).
    --file      path of local file to upload to the object.
  optional arguments :
    --mulipart  use multipart to upload the file content.
    --parallel  use parallel multipart uploads

  "putpart" action : upload part file content to s3 object for a multipart upload.
  required arguments :
    --s3path    path of the object in S3 (aka object key).
    --file      path of local file to upload to the object.
    --partnum   number of the part
    --uploadid  identifier of the multipart upload
  optional arguments :
    --mulipart  use multipart to upload the file content.

  "cancelmultipart" action : cancels a multipart upload that is not completed.
  required arguments :
    --s3path    path of the object in S3 (aka object key).
    --uploadid  Identifier of the multipart upload.

  "delete" action : deletes an s3 bucket object.
  required arguments :
    --s3path    path of the object in S3 (aka object key).

  "test" action : test creation and deletion of an s3 object.
  required arguments :
    --s3path    path of the object in S3 (aka object key).
    --size      size of the file to upload in Mb (will be generated automatically).
  optional arguments :
    --mulipart  use multipart to upload the file content.
```

## Examples : 

### Test Upload a 16 Mb file directly

```bash
$ ./s3_bash_tool --access_key ZIFAK841JFEGHU5O1VXX --secret_key [...redacted...] --url https://172.21.92.100:1446 --action test --bucket tet-test --s3path test/testfile --size 16
2022-05-20T08:54:12Z ##############################################################
2022-05-20T08:54:13Z # Checking if object test/testfile exists in bucket tet-test
2022-05-20T08:54:13Z ##############################################################

Warning: Setting custom HTTP method to HEAD with -X/--request may not work the
Warning: way you want. Consider using -I/--head instead.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0   222    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0

Curl_outputs:
=============

http_response_code:  404
----------
bytes_downloaded:  0
bytes_uploaded:  0
speed_download:  0
speed_upload:  0
----------
time_namelookup:  0.000043s
time_connect:  0.215799s
time_appconnect:  0.650904s
time_pretransfer:  0.651068s
time_redirect:  0.000000s
time_starttransfer:  0.870287s
----------
time_total:  0.870515s

curl: (18) transfer closed with 222 bytes remaining to read
2022-05-20T08:54:15Z ##############################################################
2022-05-20T08:54:15Z # Creating 16Mb File
2022-05-20T08:54:15Z ##############################################################

0+0 records in
0+0 records out
0 bytes copied, 0.0012181 s, 0.0 kB/s
2022-05-20T08:54:15Z ##############################################################
2022-05-20T08:54:15Z # Upload testfile.bin directly
2022-05-20T08:54:15Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 16.0M    0     0  100 16.0M      0   364k  0:00:45  0:00:45 --:--:--  310k

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  0
bytes_uploaded:  16777216
speed_download:  0
speed_upload:  372826
----------
time_namelookup:  0.000055s
time_connect:  0.189410s
time_appconnect:  0.573858s
time_pretransfer:  0.574046s
time_redirect:  0.000000s
time_starttransfer:  0.771540s
----------
time_total:  45.000009s

2022-05-20T08:55:02Z ##############################################################
2022-05-20T08:55:02Z # Deleting test/testfile from tet-test bucket
2022-05-20T08:55:02Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0

Curl_outputs:
=============

http_response_code:  204
----------
bytes_downloaded:  0
bytes_uploaded:  0
speed_download:  0
speed_upload:  0
----------
time_namelookup:  0.000039s
time_connect:  0.203780s
time_appconnect:  0.698777s
time_pretransfer:  0.698975s
time_redirect:  0.000000s
time_starttransfer:  0.922545s
----------
time_total:  0.923012s

2022-05-20T08:55:04Z ##############################################################
2022-05-20T08:55:04Z # Finished : upload of file of size 16 Mb to bucket tet-test/test/testfile took 51804 miliseconds.
2022-05-20T08:55:04Z ##############################################################
```

### Test Upload a 16 Mb file with multipart sequencially

```bash
$ ./s3_bash_tool --access_key ZIFAK841JFEGHU5O1VXX --secret_key [...redacted...] --url https://172.21.92.100:1446 --action test --bucket tet-test --s3path test/testfile --size 16 --multipart
2022-05-20T08:56:48Z ##############################################################
2022-05-20T08:56:48Z # Checking if object test/testfile exists in bucket tet-test
2022-05-20T08:56:48Z ##############################################################

Warning: Setting custom HTTP method to HEAD with -X/--request may not work the
Warning: way you want. Consider using -I/--head instead.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0   222    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0

Curl_outputs:
=============

http_response_code:  404
----------
bytes_downloaded:  0
bytes_uploaded:  0
speed_download:  0
speed_upload:  0
----------
time_namelookup:  0.000043s
time_connect:  0.195667s
time_appconnect:  0.599725s
time_pretransfer:  0.599945s
time_redirect:  0.000000s
time_starttransfer:  0.801433s
----------
time_total:  0.801653s

curl: (18) transfer closed with 222 bytes remaining to read
2022-05-20T08:56:50Z ##############################################################
2022-05-20T08:56:50Z # Creating 16Mb File
2022-05-20T08:56:50Z ##############################################################

0+0 records in
0+0 records out
0 bytes copied, 0.0014966 s, 0.0 kB/s
2022-05-20T08:56:50Z ##############################################################
2022-05-20T08:56:50Z # Upload testfile.bin with multipart
2022-05-20T08:56:50Z ##############################################################

2022-05-20T08:56:50Z ##############################################################
2022-05-20T08:56:50Z # Starting Multipart upload
2022-05-20T08:56:51Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   252  100   252    0     0    291      0 --:--:-- --:--:-- --:--:--   291

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  252
bytes_uploaded:  0
speed_download:  291
speed_upload:  0
----------
time_namelookup:  0.000056s
time_connect:  0.218563s
time_appconnect:  0.622162s
time_pretransfer:  0.622354s
time_redirect:  0.000000s
time_starttransfer:  0.864216s
----------
time_total:  0.865460s

2022-05-20T08:56:53Z ##############################################################
2022-05-20T08:56:53Z # Splitting input file to multiple chunks (upload id 2~GwD018JvT73-TnrUCRaPnASeMmMTPy9)
2022-05-20T08:56:53Z ##############################################################

2022-05-20T08:56:53Z ##############################################################
2022-05-20T08:56:53Z # Uploading part file testfile.bin.aa (upload id 2~GwD018JvT73-TnrUCRaPnASeMmMTPy9)
2022-05-20T08:56:53Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 8192k    0     0  100 8192k      0   352k  0:00:23  0:00:23 --:--:--  449k

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  0
bytes_uploaded:  8388608
speed_download:  0
speed_upload:  360880
----------
time_namelookup:  0.000041s
time_connect:  0.294493s
time_appconnect:  0.700881s
time_pretransfer:  0.701059s
time_redirect:  0.000000s
time_starttransfer:  0.915689s
----------
time_total:  23.244833s

2022-05-20T08:57:19Z ##############################################################
2022-05-20T08:57:19Z # Uploading part file testfile.bin.ab (upload id 2~GwD018JvT73-TnrUCRaPnASeMmMTPy9)
2022-05-20T08:57:19Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 8192k    0     0  100 8192k      0   299k  0:00:27  0:00:27 --:--:--  355k

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  0
bytes_uploaded:  8388608
speed_download:  0
speed_upload:  306454
----------
time_namelookup:  0.000051s
time_connect:  0.200268s
time_appconnect:  0.607559s
time_pretransfer:  0.607717s
time_redirect:  0.000000s
time_starttransfer:  0.811139s
----------
time_total:  27.373090s

2022-05-20T08:57:49Z ##############################################################
2022-05-20T08:57:49Z # Completing multipart upload (upload id 2~GwD018JvT73-TnrUCRaPnASeMmMTPy9)
2022-05-20T08:57:49Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   666  100   314  100   352    295    331  0:00:01  0:00:01 --:--:--   627

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  314
bytes_uploaded:  352
speed_download:  295
speed_upload:  331
----------
time_namelookup:  0.000047s
time_connect:  0.195981s
time_appconnect:  0.591150s
time_pretransfer:  0.591313s
time_redirect:  0.000000s
time_starttransfer:  0.793923s
----------
time_total:  1.062387s

2022-05-20T08:57:51Z ##############################################################
2022-05-20T08:57:51Z # Deleting test/testfile from tet-test bucket
2022-05-20T08:57:52Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0

Curl_outputs:
=============

http_response_code:  204
----------
bytes_downloaded:  0
bytes_uploaded:  0
speed_download:  0
speed_upload:  0
----------
time_namelookup:  0.000038s
time_connect:  0.208290s
time_appconnect:  0.635823s
time_pretransfer:  0.636012s
time_redirect:  0.000000s
time_starttransfer:  0.894186s
----------
time_total:  0.894391s

2022-05-20T08:57:54Z ##############################################################
2022-05-20T08:57:54Z # Finished : upload of file of size 16 Mb to bucket tet-test/test/testfile took 66427 miliseconds.
2022-05-20T08:57:54Z ##############################################################
```

### Test Upload a 16 Mb file with multipart in parallel

```bash
$ ./s3_bash_tool --access_key ZIFAK841JFEGHU5O1VXX --secret_key [...redacted...] --url https://172.21.92.100:1446 --action test --bucket tet-test --s3path test/testfile --size 16 --multipart --parallel
2022-05-20T08:59:01Z ##############################################################
2022-05-20T08:59:01Z # Checking if object test/testfile exists in bucket tet-test
2022-05-20T08:59:01Z ##############################################################

Warning: Setting custom HTTP method to HEAD with -X/--request may not work the
Warning: way you want. Consider using -I/--head instead.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0   222    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0

Curl_outputs:
=============

http_response_code:  404
----------
bytes_downloaded:  0
bytes_uploaded:  0
speed_download:  0
speed_upload:  0
----------
time_namelookup:  0.000039s
time_connect:  0.189441s
time_appconnect:  0.574029s
time_pretransfer:  0.574111s
time_redirect:  0.000000s
time_starttransfer:  0.768677s
----------
time_total:  0.769244s

curl: (18) transfer closed with 222 bytes remaining to read
2022-05-20T08:59:04Z ##############################################################
2022-05-20T08:59:04Z # Creating 16Mb File
2022-05-20T08:59:04Z ##############################################################

0+0 records in
0+0 records out
0 bytes copied, 0.0013017 s, 0.0 kB/s
2022-05-20T08:59:04Z ##############################################################
2022-05-20T08:59:04Z # Upload testfile.bin with multipart
2022-05-20T08:59:04Z ##############################################################

2022-05-20T08:59:04Z ##############################################################
2022-05-20T08:59:04Z # Starting Multipart upload
2022-05-20T08:59:04Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   252  100   252    0     0    299      0 --:--:-- --:--:-- --:--:--   299

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  252
bytes_uploaded:  0
speed_download:  299
speed_upload:  0
----------
time_namelookup:  0.000078s
time_connect:  0.221662s
time_appconnect:  0.624389s
time_pretransfer:  0.624585s
time_redirect:  0.000000s
time_starttransfer:  0.839727s
----------
time_total:  0.840668s

2022-05-20T08:59:07Z ##############################################################
2022-05-20T08:59:07Z # Splitting input file to multiple chunks (upload id 2~4kQD71ld-D4k-j4sEjc8RkfbtfgmJmI)
2022-05-20T08:59:07Z ##############################################################

2022-05-20T08:59:07Z ##############################################################
2022-05-20T08:59:07Z # Uploading part file testfile.bin.aa (upload id 2~4kQD71ld-D4k-j4sEjc8RkfbtfgmJmI)
2022-05-20T08:59:07Z ##############################################################

2022-05-20T08:59:07Z ##############################################################
2022-05-20T08:59:07Z # Uploading part file testfile.bin.ab (upload id 2~4kQD71ld-D4k-j4sEjc8RkfbtfgmJmI)
2022-05-20T08:59:07Z ##############################################################

2022-05-20T08:59:08Z ##############################################################
2022-05-20T08:59:08Z # Waiting for parallel parts upload to finish (upload id 2~4kQD71ld-D4k-j4sEjc8RkfbtfgmJmI)
2022-05-20T08:59:08Z ##############################################################

2022-05-20T08:59:07Z ##############################################################
2022-05-20T08:59:07Z # Upload Part testfile.bin.aa number 1 upload id 2~4kQD71ld-D4k-j4sEjc8RkfbtfgmJmI
2022-05-20T08:59:08Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 8192k    0     0  100 8192k      0   422k  0:00:19  0:00:19 --:--:--  463k

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  0
bytes_uploaded:  8388608
speed_download:  0
speed_upload:  432915
----------
time_namelookup:  0.000038s
time_connect:  0.210859s
time_appconnect:  0.755829s
time_pretransfer:  0.755909s
time_redirect:  0.000000s
time_starttransfer:  1.035816s
----------
time_total:  19.377028s


2022-05-20T08:59:08Z ##############################################################
2022-05-20T08:59:08Z # Upload Part testfile.bin.ab number 2 upload id 2~4kQD71ld-D4k-j4sEjc8RkfbtfgmJmI
2022-05-20T08:59:08Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 8192k    0     0  100 8192k      0   425k  0:00:19  0:00:19 --:--:--  462k

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  0
bytes_uploaded:  8388608
speed_download:  0
speed_upload:  436025
----------
time_namelookup:  0.000050s
time_connect:  0.252097s
time_appconnect:  0.789900s
time_pretransfer:  0.790001s
time_redirect:  0.000000s
time_starttransfer:  1.063955s
----------
time_total:  19.238780s


2022-05-20T08:59:30Z ##############################################################
2022-05-20T08:59:30Z # Completing multipart upload (upload id 2~4kQD71ld-D4k-j4sEjc8RkfbtfgmJmI)
2022-05-20T08:59:30Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   666  100   314  100   352    293    329  0:00:01  0:00:01 --:--:--   623

Curl_outputs:
=============

http_response_code:  200
----------
bytes_downloaded:  314
bytes_uploaded:  352
speed_download:  293
speed_upload:  329
----------
time_namelookup:  0.000054s
time_connect:  0.198220s
time_appconnect:  0.605242s
time_pretransfer:  0.605434s
time_redirect:  0.000000s
time_starttransfer:  0.805258s
----------
time_total:  1.068983s

2022-05-20T08:59:33Z ##############################################################
2022-05-20T08:59:33Z # Deleting test/testfile from tet-test bucket
2022-05-20T08:59:33Z ##############################################################

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0

Curl_outputs:
=============

http_response_code:  204
----------
bytes_downloaded:  0
bytes_uploaded:  0
speed_download:  0
speed_upload:  0
----------
time_namelookup:  0.000042s
time_connect:  0.212207s
time_appconnect:  1.189015s
time_pretransfer:  1.189229s
time_redirect:  0.000000s
time_starttransfer:  1.459402s
----------
time_total:  1.459587s

2022-05-20T08:59:36Z ##############################################################
2022-05-20T08:59:36Z # Finished : upload of file of size 16 Mb to bucket tet-test/test/testfile took 34964 miliseconds.
2022-05-20T08:59:36Z ##############################################################
```
