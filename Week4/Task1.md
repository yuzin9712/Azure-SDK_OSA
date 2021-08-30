# Issue 및 PR 정리

## 선택한 Issue
[Azure-SDK-JAVA 선정 이슈](https://github.com/Azure/azure-sdk-for-java/issues/23288)


## 내가 Issue를 등록한다면?

### Description
###### 어떤 내용인지 간략하게 작성합니다.
When i test other endpoint clouds like AzureChinaCloud, AzureUsGovernment or AzureGermanCloud, there is an error.
...


### Process
###### 어떤 방식으로 진행했는지 프로세스를 작성합니다.
1) Add code in ...
3) ...

###### 이러한 과정을 통해 나온 에러 메세지를 추가합니다.
An error occurs and shows error message like below.

```java
[ERROR] Break container lease AC illegal [match: null, noneMatch: garbage, #1] Time elapsed: 25.093s <<< ERROR!
reactor.core.Exception$ReactiveException: java.net.UnknownHostException: failed to resolve '***.blob.core.windows.net' after 2 queries at reactor ...

```

### Desired Results
###### 어떤 결과가 나타나야 하는지에 대해서 작성합니다.
There is no error even if the endpoint is a different cloud in testing.

### Expected Causes
###### 생각한 원인에 대해서 작성합니다.
I think that the excpetion is occured by [the part of the code](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/storage/test-resources.json#L8).

Different Region cloud has their own storageEndpoint suffix.
But in [the part of the code](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/storage/test-resources.json#L8), endpointSuffix is hard-coded with defaultValue "core.windows.net".

|Name | storageEndpoint  |
|---|---|
|AzureCloud   |core.windows.net   |
|AzureChinaCloud   |core.chinacloudapi.cn   |
|AzureUSGovernment   |core.usgovcloudapi.net   |
|AzureGermanCloud   |core.cloudapi.de   |

### Suggestion
###### 어떤 방식으로 이슈를 해결하면 좋을지 제안합니다.
Fix the hard-coded endpointSuffix in test-resources.json into expression that users not only use default value but also specify a suffix endpoint.



### Related Materials
###### 관련 자료를 첨부합니다.
[Check cloud list and suffix endpoint](https://docs.microsoft.com/ko-kr/cli/azure/manage-clouds-azure-cli#list-available-clouds)  
[Testing about non-hardcoding](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-test-cases#location-uses-parameter)

