# Task 2
> Azure SDK 예제 실행해보기  
> https://docs.microsoft.com/ko-kr/azure/developer/java/sdk/overview


### 사전 준비

- Homebrew로 CLI 설치

```shell
brew update && brew install azure-cli
```

### 인증 설정

```shell
az ad sp create-for-rbac --name AzureYujinTest

# 다음과 같은 응답을 받았다.
{
  "appId": "a487e0c1-82af-47d9-9a0b-af184eb87646d",
  "displayName": "AzureJavaTest",
  "name": "http://AzureJavaTest",
  "password": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "tenant": "tttttttt-tttt-tttt-tttt-tttttttttttt"
}

# 나의 계정 정보를 확인한다.
az account show

export AZURE_SUBSCRIPTION_ID={위에서 얻은 id값}
export AZURE_CLIENT_ID={appId값}
export AZURE_CLIENT_SECRET={password값}
export AZURE_TENANT_ID={tenant값}
```


### 새 Maven 프로젝트 만들기
```shell
mkdir java-azure-test
cd java-azure-test

mvn archetype:generate -DgroupId=com.fabrikam -DartifactId=AzureApp \
-DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

# mvn이 없어서 command not found
brew install maven 하고 재실행했더니 가능

# 그러면 다음과 같은 디렉터리가 생길 것이다.
java-azure-test
   - AzureApp
      - pom.xml
      - src
```

### Azure에서 VM 만들기

#### 포탈에서 VM 직접 생성하기

![image](https://user-images.githubusercontent.com/58067265/129683704-e6438f27-54d2-4678-892d-db1cd66c0043.png)

![image](https://user-images.githubusercontent.com/58067265/129683932-646d1b2a-b99d-4c63-8613-61adcfd18bff.png)


#### 코드로 VM 생성하기 (?)

![image](https://user-images.githubusercontent.com/58067265/129684799-07a3d969-90ee-4c7f-a771-48bd4e69bd95.png)
<code>vm-ssh-key</code> 라는 이름으로 ssh 키를 발급했다. 리소스 그룹의 이름도 vm-ssh-key로 했다. 이떄 받은 퍼블릭 키를 코드에 등록하겠다.


```shell
mvn compile exec:java
```

하지만 .. 실행결과는 에러가 나왔다.🥲
![image](https://user-images.githubusercontent.com/58067265/129685572-07666f9d-b4ec-4a85-bd28-bc4fdb28b349.png)

[링크](https://www.testingdocs.com/questions/how-to-fix-source-option-5-is-no-longer-supported-use-7-or-later/)를 통해서 pom.xml을 수정해줬다.

그랬더니 다른 에러가 나왔다.😭 이건 Azure 에서 던져주는 예외 같다. (아마도..)
![image](https://user-images.githubusercontent.com/58067265/129690736-1090484d-3d34-478f-a7ee-d00672c486ef.png)

읽어보니 '' 이미 존재하는 그룹이라고 한다. withNewResourceGroup("이름..") 에 이미 내가 만들어놓은 그룹 이름을 작성하는 거라고 생각해서 이미 만든 그룹이름을 넣어줬고 그래서 다음과 같은 에러가 나왔던 것이다.

그래서 이름을 새로운걸로 다시 지어서 다시 컴파일했다. 빌드 성공했지만 다음과 같은 에러가 나와있었다!

![image](https://user-images.githubusercontent.com/58067265/129691503-50f1f401-88f8-4258-905f-ac270937dfda.png)

무슨 에러인지 잘 모르겠어서 우선 만들어졌나 확인을 했다.

```shell
az vm list --resource-group new-vm-test
```

![image](https://user-images.githubusercontent.com/58067265/129691868-42fcf5be-a3f9-4594-b373-b69070a61fa0.png)


.. 뭐지 결과는 만들어졌다!!!! 되면 이상한데 되니까 기분이 이상하다..🤔

저 에러에 대해서 알아봐야겠다..
