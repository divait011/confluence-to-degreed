# confluence-to-degreed

# Description

This repo creates an AWS Step Function that integrates specified content (via a white list of space keys) from a confluence wiki site to a Degreed LXP instance. 
# Process Sequence Diagram

The below sequence diagram shows the steps taken by the step function, but is a bit of a misnomer, as steps 20, 80 and 120 are actually all executed in parallel, via a Step Function Parallel state.

![Sequence Diagram](https://raw.githubusercontent.com/stevelitras/confluence-to-degreed/master/images/sequence.svg?sanitize=true)

# Process Activity Diagram

![Activity Diagram](https://raw.githubusercontent.com/stevelitras/confluence-to-degreed/master/images/activity.svg?sanitize=true)

# Configuration

Configuration is handled through the SSM parameter store. The Cloud Formation Template takes a "SSMPathRoot" Parameter that is the top level of the parameter hierarchy - in the diagram below, it corresponds with the "confluence-to-degreed" node. 

![Configuration Options Mindmap](https://raw.githubusercontent.com/stevelitras/confluence-to-degreed/master/images/params.svg?sanitize=true)

## Config Options

### slack container

Name | Type | Content | Caveats 
---------- | ---------- | ---------- | ---------
slack_token | SecureString | token to provide to the slack API. |  encoded with the key created by the cloud formation template
slack_channel | String | slack channel to be posting events/errors to. |

### wiki container

Name | Type | Content | Caveats 
---------- | ---------- | ---------- | ---------
url | String | the URL head for the wiki api. | Since URL's can't be natively stored in params, replace the "http" protocol name with "urlhead", and "https" with "urlheads" - the code will substitute accordingly. Do NOT include any URI beyond the root (as the URI's are handled in the code). Example urlheads://wiki.somewhere.com/
uiurl | String | the URL head for the wiki UI (the web root). | Since URL's can't be natively stored in params, replace the "http" protocol name with "urlhead", and "https" with "urlheads" - the code will substitute accordingly. Do NOT include any URI beyond the root (as the URI's are handled in the code). Example urlheads://wiki.somewhere.com/
passwd | SecureString | the password for the wiki API (using basic auth, combined with the username parameter). | encoded with the key created by the cloud formation template
username | String | the username to authenticate with against the API. |
item_limit | String | the number of items to set as the limit parameter for API pagination. This should be set at or near the maximum number of pages the API can request without expansion (in my instance of Confluence, that's 500). Setting higher than the limit will create gaps in the "all_pages" inventory, which could lead to links being removed from Degreed. |
max_labels | String | the maximum number of labels to push through to degreed as "skill tags". The code will take the first n (where n is the max_labels value). | Degreed recommends no more than 20 (but preferably 5 or less)
spaces | StringList | comma separated list of all of the "whitelisted" Space Keys. |

### degreed container

Name | Type | Content | Caveats 
---------- | ---------- | ---------- | ---------
client_id | SecureString | the client_id for authentication against the degreed API - you'll need to get this from your degreed account team. | (encoded with the key created by the cloud formation template) 
client_secret | SecureString | the client_secret for authentication against the degreed API - you'll need to get this from your degreed account team. | (encoded with the key created by the cloud formation template) 
oauthurl | String |the url for the oauth token endpoint for the degreed API. | Since URL's can't be natively stored in params, replace the "http" protocol name with "urlhead", and "https" with "urlheads" - the code will substitute accordingly. Do NOT include any URI beyond the root (as the URI's are handled in the code). Example urlheads://wiki.somewhere.com/
url | String | the url for the API endpoints - do not include any URI - just the protocol + host, like urlheads://api.degreed.com/ | Since URL's can't be natively stored in params, replace the "http" protocol name with "urlhead", and "https" with "urlheads" - the code will substitute accordingly. Do NOT include any URI beyond the root (as the URI's are handled in the code). Example urlheads://wiki.somewhere.com/
article_limit | String | the number to use for the "limit" parameter for degreed API. | This should be the max available for the API, preferably, to minimize impact of paging on time. 

### dynamo_table (optional) container

The dynamo_table container provides the user the option to store the spac e key whitelist in a dynamo db table (each record contains a single "space" attribute including the space key to be whitelisted). The default is to use the wiki/spaces parameter to contain the whitelist.

Name | Type | Content | Caveats 
---------- | ---------- | ---------- | ---------
arn | String | the arn to the table. |
name | String | the name of the table. |
