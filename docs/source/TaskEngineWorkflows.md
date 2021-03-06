# TASK ENGINE WORKFLOWS

## Vodstream

This workflow will create a server side manifest, with and/or without DRM, that can be used for on the fly delivery of VOD content via USP.

### Vodstream: Parameters

| Parameter Name    | Required |  Description | Default |
| ----------------- | -------- | ------------ | ------- |
| workflow          |Yes| Specify 'vodstream'.|
| content_id        |Yes| Unique identifier of the content. This is usually a key that allows identification of the content in the client’s system.|
| source_folder     |Yes| Location of the source files. All files to be processed will need to be in a discrete folder, the ‘root’ folder will be specified in the client configuration.|
| delete_source     |No | This boolean indicates whether the source should be deleted from s3 after the job has completed.| false|
| encrypted         |No | This boolean indicates whether the active manifest, for a job, should be the encrypted manifest.| true|
| output_folder     |No | The folder for processed files to be placed.  The ‘root’ folder will be specified in the client configuration. | source_folder|
| drm               |No | The type of DRM that is required. This could be “playready” and/or ”widevine” and/or ”fairplay” and/or “cenc” and/or "aes". If this value isn’t present then no DRM is applied.|
| rest_endpoints    |No | Endpoints that will receive the callbacks defined in the workflow. Multiple end points can be specified.||
| create_thumbnail  |No | This boolean indicates whether a thumbnail should be created for the content.|true|
| thumbnail_time    |No | Time at which the thumbnail will be taken.|first frame|
| generate_mp4      |No | This boolean indicates whether an MP4 is generated for the VOD content.|false|
| mp4_filename      |No | Filename for the generated MP4, if generate_mp4 is st to true.|{content_id}.mp4|
| combine_sources   |No | This boolean indicates whether the isma/v/ts generated from the source content are to be combined into a single ismv before packaging the manifests.|true|
| create_dref       |No | This boolean indicates whether a dref MP4 is generated for the VOD content.|true|
| all_audio_tracks  |No | This boolean indicates whether all audio tracks are captured or only the audio tracks with the highest bitrates for each language are captured| true|

### Vodstream: JSON Payload example

```json
{
  "client": "staging",
  "job": {
    "workflow": "vodstream"
  },
  "parameters": {
    "content_id": "demo1",
    "source_folder": "mz-ast-2055fcff-8cca-4e37-85b9-9647dbe50398-1",
    "delete_source": false,
    "encrypted": true,
    "output_folder": "mz-ast-2055fcff-8cca-4e37-85b9-9647dbe50398-1",
    "drm": [
      "fairplay",
      "playready",
      "widevine"
    ],
    "rest_endpoints": [
      "https://vis.vuworkflow.staging.vualto.com/api/event/vuflow/taskenginecallback",
      "http://aaa.com/end",
      "http://bbb.com/end"
    ],
    "create_thumbnail": true,
    "thumbnail_time": "1:34.000",
    "generate_mp4": true,
    "mp4_filename": "demo_sample.mp4",
    "combine_sources": true,
    "create_dref": true,
    "all_audio_tracks": true
  }
}
```

### Vodstream: Callback properties

**Task Callback:**

Task callbacks are triggered after each task within a workflow is completed. Below is a list of the defualt properties for the callback:

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| task_id           | Unique task identifier generated by the Task Engine. |
| task_name         | Name of the task that triggered the callback. |
| workflow          | Name of the workflow being executed. |
| event             | This will identify the event that caused the callback to be triggered. It can be one of `start`, `complete` or `fail`. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Any message assoicated with the event. This will usually contain exception messages. |

**Job Callback**

Job callbacks are triggered when the entire job has completed. Below is a list of the default properties for the callback.

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| status            | This will identify the status of the job. It can be either `completed` or `failed`. |
| workflow          | Name of the workflow being executed. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Full path of the active manifest, for the generated content. |
| files             | List of files (manifests, content files, thumbnail, etc...) that have been copied to the final destination. |

## Vodcapture

This workflow allows you to create a frame accurate vod clip by passing in a start and end UTC time stamp. The result will be an MP4 on disk.

### Vodcapture: Parameters

| Parameter Name    | Required |  Description | Default |
| ----------------- | -------- | ------------ | ------- |
| workflow          |Yes| Specify 'vodcapture'.||
| content_id        |Yes| This is the id for the resulting capture.||
| output_folder     |Yes| This is the folder where the resulting capture wil be saved on S3. This is cleared before the capture is uploaded.||
| source            |Yes| This would need to be either an HLS, MSS or Dash stream URL to the Live or Archive content. e.g. http://mydomain.com/test.ism/.m3u8 , http://mydomain.com/test.ism/manifest , http://mydomain.com/test.ism/.mpd|| 
| start             |No | UTC timestamp for the start timecode. e.g 2016-10-13T10:10:40.251Z OR Offsets e.g. “hh:mm:ss”||
| end               |No | UTC timestamp for the end timecode e.g 2016-10-13T10:20:40.251Z OR Offsets e.g. “hh:mm:ss” ||
| filter            |No | This allows you to pass filter expressions to select certain video, audio tracks. e.g. to all video bitrates below 8Mbps and all audio bitrates at 64Kbps "type==\\"video\\"&&systemBitrate==800000\|\|type==\\"audio\\"&&systemBitrate==64000"||
| encrypted         |No | This boolean allows you to set the encrypted manifest as active after the capture is complete| false|
| drm               |No | The type of Output DRM that is required. This could be “playready” and/or  ”widevine” and/or ”fairplay” and/or “cenc” and/or "aes".  If this value isn’t present then no DRM is applied.||
| frame_accurate    |No | This boolean allows the capture to be done using frame accuracy.|true|
| copy_ts           |No | This boolean indicates whether the timestamps should be included in the resulting manifests.|false|
| rest_endpoints    |No | Endpoints that will receive the callbacks defined in the workflow. Multiple end points can be specified.||
| key_id            |No | Should the stream be DRM’d we would require the KeyID ||
| content_key       |No | Should the stream be DRM’d we would require the Content Key ||
| output_file       |No | Name of the output ismv file| <content_id>_capture.ismv |
| generate_vod      |No | This boolean indicates whether VOD manifests are generated for the capture.|true|
| create_thumbnail  |No | This boolean indicates whether a thumbnail should be created for the content|true|
| thumbnail_time    |No | Time at which the thumbnail will be taken.|first frame|
| generate_mp4      |No | This boolean indicates whether an MP4 is generated for the VOD content|false|
| mp4_filename      |No | Filename for the generated MP4|{content_id}.mp4|
| create_dref       |No | This boolean indicates whether a dref MP4 is generated for the VOD content|<generate_vod>|

### Vodcapture: JSON Payload example

```json
{
  "client": "staging",
  "job": {
    "workflow": "vodcapture"
  },
  "parameters": {
    "content_id": "demo_1",
    "output_folder": "demo_1",
    "clips": [
      {
        "source": "http://mydomain.com/live.isml/manifest",
        "start": "2018-06-06T10:00:00.000",
        "end": "2018-06-06T10:30:00.000",
        "filter": "type==\"audio\"||type==\"video\"&&systemBitrate==1300000"
      }
    ],
    "encrypted": false,
    "drm": [
        "fairplay",
        "playready",
        "cenc",
        "widevine",
        "aes"
    ],
    "frame_accurate": true,
    "copy_ts": false,
    "rest_endpoints": [
	  "https://vis.vuworkflow.staging.vualto.com/api/event/vuflow/taskenginecallback",
	  "http://your.custom.endpoint"
    ],
    "key_id": "346AS5847333DDSHKFSDS7633429CD33",
    "content_key": "346AS5847333DDSHKFSDS7633429CD33",
    "output_file": "demo_sample.ismv",
    "create_thumbnail": true,
    "thumbnail_time": "1:34.000",
    "generate_mp4": true,
    "mp4_filename": "demo_sample.mp4",
    "create_dref": true
  }
}
```

### Vodcapture: Callback properties

**Task Callback**

Task callbacks are triggered after each task within a workflow is completed. Below is a list of the defualt properties for the callback:

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| task_id           | Unique task identifier generated by the Task Engine. |
| task_name         | Name of the task that triggered the callback. |
| workflow          | Name of the workflow being executed. |
| event             | This will identify the event that caused the callback to be triggered. It can be one of `start`, `complete` or `fail`. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Any message assoicated with the event. This will usually contain exception messages. |

**Job Callback**

Job callbacks are triggered when the entire job has completed. Below is a list of the default properties for the callback.

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| status            | This will identify the status of the job. It can be either `completed` or `failed`. |
| workflow          | Name of the workflow being executed. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Full path of the active manifest, for the generated content. |
| files             | List of files (manifests, content files, thumbnail, etc...) that have been copied to the final destination. |

## Voddeletes3

This workflow allows you to delete content on S3.

### Voddeletes3: Parameters

| Parameter Name    | Required |  Description | Default |
| ----------------- | -------- | ------------ | ------- |
| workflow          |Yes| Specify 'voddeletes3'.||
| content_id        |Yes| Unique identifier of the content. This is usually a key that allows identification of the content in the client’s system.||
| folder            |Yes| Folder where the content to be deleted resides||
| rest_endpoints    |No | Endpoints that will receive the callbacks defined in the workflow. Multiple end points can be specified.||

### Voddeletes3: JSON Payload example

```json
{
  "client": "staging",
  "job": {
    "workflow": "voddeletes3"
  },
  "parameters": {
    "content_id": "demo1",
    "folder": "vualto-test-1",
    "rest_endpoints": [
			"https://vis.vuworkflow.staging.vualto.com/api/event/vuflow/taskenginecallback",
			"http://your.custom.endpoint"
		]
  }
}
```

### Voddeletes3: Callback properties

**Task Callback**

Task callbacks are triggered after each task within a workflow is completed. Below is a list of the defualt properties for the callback:

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| task_id           | Unique task identifier generated by the Task Engine. |
| task_name         | Name of the task that triggered the callback. |
| workflow          | Name of the workflow being executed. |
| event             | This will identify the event that caused the callback to be triggered. It can be one of `start`, `complete` or `fail`. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Any message assoicated with the event. This will usually contain exception messages. |

**Job Callback**

Job callbacks are triggered when the entire job has completed. Below is a list of the default properties for the callback.

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| status            | This will identify the status of the job. It can be either `completed` or `failed`. |
| workflow          | Name of the workflow being executed. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Name of the folder deleted from S3. |

## Drmswitch

This workflow allows you to toggle DRM on and off.

### Drmswitch: Parameters

| Parameter Name    | Required |  Description | Default |
| ----------------- | -------- | ------------ | ------- |
| workflow          |Yes| Specify 'drmswitch'.||
| content_id        |Yes| Unique identifier of the content. This is usually a key that allows identification of the content in the client’s system.||
| folder            |Yes| Folder where the content to be DRM toggled resides||
| rest_endpoints    |No | Endpoints that will receive the callbacks defined in the workflow. Multiple end points can be specified.||

### Drmswitch: Payload example

```json
{
  "client": "staging",
  "job": {
    "workflow": "drmswitch"
  },
  "parameters": {
    "content_id": "demo1",
    "folder": "vualto-test-1",
    		"rest_endpoints": [
			"https://vis.vuworkflow.staging.vualto.com/api/event/vuflow/taskenginecallback",
			"http://your.custom.endpoint"
		]
  }
}
```

### Drmswitch: Callback properties

**Task Callback**

Task callbacks are triggered after each task within a workflow is completed. Below is a list of the defualt properties for the callback:

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| task_id           | Unique task identifier generated by the Task Engine. |
| task_name         | Name of the task that triggered the callback. |
| workflow          | Name of the workflow being executed. |
| event             | This will identify the event that caused the callback to be triggered. It can be one of `start`, `complete` or `fail`. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Any message assoicated with the event. This will usually contain exception messages. |

**Job Callback**

Job callbacks are triggered when the entire job has completed. Below is a list of the default properties for the callback.

| Property Name     | Required |
| ----------------- | -------- |
| job_id            | Unique job identifier generated by the Task Engine. |
| status            | This will identify the status of the job. It can be either `completed` or `failed`. |
| workflow          | Name of the workflow being executed. |
| content_id        | Content ID provided when the job was submitted. |
| message           | Full path of the active manifest, for the generated content. |

## Additional Workflow Features

### Priority

The Task Engine supports ordering of jobs by priority. The priority parameter can be submitted as part of the json payload being submitted. The priority is in ascending order as follows:

1 - Top Priority  
.  
.  
5 - Default  
.  
.  
10 - Least Priority

The `"priority"` parameter needs to be submitted within the `"job"` section of the json payload as shown below:

```json
{
  "client": "demo-client",
  "job": {
    "workflow": "vodcapture",
    "priority": 3
  },
  "parameters": {
    "content_id": "demo1",
    ...
    ...
    ...
  }
}
```

Whenever an execution slot is available, the system will first check by priority and then check the submission time and date of the job. In the case where multiple jobs are executed with the same priority (eg. with the default priority 5), the Task Engine operates in a FIFO (First In First Out) manner.

### Multiple Clips

The Task Engine includes a feature that will allow multiple clips to be stitched togther into a single clip, in a single job. This can be done by defining multiple objects within the `"clips"` parameter in the json payload for `vodcapture`. This also allows a mixture of live and VoD sources to be captured and stitched toghether into a new clip. The example below shows how the `"clips"` parameter would need to be provided to achive this.

```json
{
  "client": "staging",
  "job": {
    "workflow": "vodcapture"
  },
  "parameters": {
    "content_id": "demo_1",
    "output_folder": "demo_1",
    "clips": [
      {
        "source": "http://mydomain.com/copyright.ism/manifest",
      },
      {
        "source": "http://mydomain.com/live.isml/manifest",
        "start": "2018-06-06T10:00:00.000",
        "end": "2018-06-06T10:30:00.000",
        "filter": "type==\"audio\"||type==\"video\"&&systemBitrate==1300000"
      },
      {
        "source": "http://mydomain.com/live.isml/manifest",
        "start": "2018-06-06T10:35:00.000",
        "end": "2018-06-06T11:00:00.000",
        "filter": "type==\"audio\"||type==\"video\"&&systemBitrate==1300000"
      }
    ],
    .
    .
    .
  }
}
```

### Multiple Sources

In some cases, a live stream could have multiple origins setup (eg. for load balancing the origin servers). The Task Engine, allows for both streams to be defined as the source for a capture. It is smart enough to find which live stream will provide the best output capture and use that stream as the source. If the Task Engine discovers discontinuities within the streams, it will use segments from both streams to try and generate a clip with the least number of missing fragements.

The streams can be defined in the `"sources"` parameter when executing the `vodcapture` workflow.

```json
{
  "client": "staging",
  "job": {
    "workflow": "vodcapture"
  },
  "parameters": {
    "content_id": "demo_1",
    "output_folder": "demo_1",
    "clips": [
      {
        "sources": [
          "http://mydomain.com/live_1.isml/manifest",
          "http://mydomain.com/live_2.isml/manifest"
        ],
        "start": "2018-06-06T10:00:00.000",
        "end": "2018-06-06T10:30:00.000",
        "filter": "type==\"audio\"||type==\"video\"&&systemBitrate==1300000"
      }
    ],
    .
    .
    .
  }
}
```

In this case, `"sources"`  replaces the `"source"` parameter, however; it can still be used in conjunction with other clips which only contain a single stream as shown below.

```json
{
  "client": "staging",
  "job": {
    "workflow": "vodcapture"
  },
  "parameters": {
    "content_id": "demo_1",
    "output_folder": "demo_1",
    "clips": [
      {
        "source": "http://mydomain.com/copyright.ism/manifest",
      },
      {
        "sources": [
          "http://mydomain.com/live_1.isml/manifest",
          "http://mydomain.com/live_2.isml/manifest"
        ],
        "start": "2018-06-06T10:00:00.000",
        "end": "2018-06-06T10:30:00.000",
        "filter": "type==\"audio\"||type==\"video\"&&systemBitrate==1300000"
      }
    ],
    .
    .
    .
  }
}
```

### Generate Download Clips

The Task Engine `vodcapture` workflow supports generating download clips without creating VoD assets. This is done by setting the property `"generate_vod"` to false and `"generate_mp4"` to true. It is important that if `"generate_vod"` is set to false, to not manually override the `"create_dref"` parameter. Setting `"create_dref"` to true will lead to a failed workflow as this requires VoD assets to generate DREF mp4s.

The resulting downlaod will be an MP4 containg all the video, audio and caption tracks defined using the clip's `"filter"` parameter. If no filter is defined, the resulting MP4 will contain all the tracks availble in the stream.

## Workflow Trigger Example

Example of a curl command to trigger ingest for the vodstream workflow:

    curl -X POST \
    http://vualto.demo.com/job \
    -H "API-KEY: aabbccdd-1122-3344-5566-eeff77889900" \
    -H 'Cache-Control: no-cache' \
    -H 'Content-Type: application/json' \
    -d '{
      "client": "vualto",
      "job": {
        "workflow": "vodstream"
      },
      "parameters": {
          "content_id": "demo_1",
          "source_folder": "/input/demo1",
          "delete_source": false,
          "encrypted": false,
          "output_folder": "/test",
          "drm": [
            "fairplay",
            "playready",
            "cenc",
            "widevine"
          ],"rest_endpoints": [
            "https://webhook.site/55151d14-cee1-416b-b956-a90525ae8f58",
            "https://webhook.site/bc4c13ee-f118-4d5b-a4af-7ac07890a7f1"
          ],
          "create_thumbnail": false,
          "generate_mp4": true,
          "combine_sources": true,
          "create_dref": true,
          "all_audio_tracks": false,
        }
      }

This results in the files `<content_id>_<drm_tag>_<unique_guid>.ism` and `<content_id>_<unique_guid>.ismv` being produced in the folder:

```<configured_root>(/<optional_output_folder>)/<content_id>```

The response from this call should be either a 200 OK, with the following payload:

``` { "job_id": <job_id>, "result": "accepted" } ```

or a 400 BAD REQUEST with the following payload:

``` { "error": "<description_of_error>" } ```

Assuming the call is successful, this would add an ingest job to the Task Engine queue, with the files to be ingested expected to be in the following location:

``` <input_root>/input/demo1 ```

If the process completes successfully, then the output would be the following files:

``` <output_root>/test/demo1.ism ```
``` <output_root>/test/demo1.ismv ```

If the ‘output_folder’ parameter was excluded, then the files would be output to the following locations:

``` <output_root>/demo1.ism ```
``` <output_root>/demo1.ismv ```

**NOTE:** there may be some additional files, depending on the exact processes involved, but the minimum would usually be these.