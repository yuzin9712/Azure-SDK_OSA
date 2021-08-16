# Task1
> Azure SDK Design Guidelines 부분 번역하면서 정리하기  
> [관련 자료 링크](https://azure.github.io/azure-sdk/java_introduction.html)


## Azure SDK API Design

### Supporting Types
- #### Model Types
> Model Type은 Azure 서비스로 부터 제공하거나 받아야 하는 정보를 class 단위로 정의한 것을 말함

⛔️ DO NOT!
  model 클래스를 위한 builder 클래스를 따로 만들지 않는다.  
  
✅ DO!
   * 사용자가 인스턴스화 할 수 있는 모든 모델 클래스는 public 생성자를 제공한다. 하지만, 서비스로 부터 리턴되는 모델타입과 같이 사용자가 인스턴스화할 수 없는 클래스들은 어떠한 public 생성자도 사용하면 안된다.  
      
   🤔 Why?
    +++++++
    
   * final이 붙은 required property를 갖지 않았다면, 인자값이 없는 기본 생성자를 제공해야 한다.
   * 만약 모델이 required property를 가진다면, 인자를 갖는 1개 이상의 생성자를 만든다.
   +++++

🔵 Mutually Exclusive?
++++++++

```java
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

public final class TextDocumentInput {
  
  // required properties
  private final String id;
  private final String text;
  
  // optional property
  private String language;
  
  public TextDocumentInput(String id, String text) {
    this.id = Objects.requiredNonNull(id, "'id' cannot be null");
    this.text = Objects.requiredNonNull(text, "'text' cannot be null");
  }
  
  // required properties only have getters
  public String getId() {
    return this.id;
  }
  
  public String getText() {
    return this.text;
  }
  
  // optional property has both getter and fluent setter
  public String getLanguage() {
    return this.language;
  }
  
  public TextDocumentInput setLanguage(String language) {
    this.language = language;
    return this;
  }
}
}
```

++++
✅ DO!
  * fluent setter API를 사용하고, set 메소드는 this를 리턴해야 한다.  
  * extended type을 리턴하기위해 fluent type을 extends 할 때, 모든 set 메소드들을 오버라이드 해야 한다.  
  = 아래 예시 코드에서 AbandonOptions class 가 SettlementOptions class 를 상속 받았고, SettlementOptions의 모든 set 메소드를 Override하고 AbandonOptions 타입을 리턴하고 있다.
  
```java
@Fluent
public class SettlementOptions {
    private ServiceBusTransactionContext transactionContext;

    public ServiceBusTransactionContext getTransactionContext() {
        return transactionContext;
    }

    public SettlementOptions setTransactionContext(ServiceBusTransactionContext transactionContext) {
        this.transactionContext = transactionContext;
        return this;
    }
}

@Fluent
public final AbandonOptions extends SettlementOptions {
    private Map<String, Object> propertiesToModify;

    public Map<String, Object> getPropertiesToModify() {
        return propertiesToModify;
    }

    public AbandonOptions setPropertiesToModify(Map<String, Object> propertiesToModify) {
        this.propertiesToModify = propertiesToModify;
        return this;
    }

    // Override setter method of the parent class
    @Override
    public AbandonOptions setTransactionContext(ServiceBusTransactionContext transactionContext) {
        super.setTransactionContext(transactionContext);
        return this;
    }
}
  ```

✅ DO!
  * 클래스에 @Fluent 어노테이션을 붙인다.
  * JavaBean 컨벤션에 따라서 getXXX, setXXX 그리고 isXXX를 사용한다.
  * raw 데이터로 부터 새로운 모델 인스턴스가 필요할 경우, static 메소드를 포함하고, 그 형식은 from<dataformat> 이다. 예를 들어, string으로
  
⛔️ DO NOT!
  * 

  
✅ DO!
  * 서비스 반환 유형만을 리턴하고 바람직하지 못한 공개 API를 가진 model 클래스를 .implmentation.models 패키지에 넣어라. 인터페이스는 .models 패키지에 넣어야만 하고, 공개 API를 거쳐서 최종 사용자에게 리턴되는 타입이어야 한다.
  
  ??

  🔵 <code>@Fluent</code>  
  🔵 <code>Chaining of set operations</code>


- #### Enumerations
  Java에서 Enumerationsd는 편리하지만, 잘못된 사용을 하는 경우에 API에 급격한 변화를 이끌어내기도 한다. 이는 Java 컴파일러가 모든 enum 값들이 switch 문에서 열거되지 않았을 때 발생하는데, 새 값을 추가하면 새로운 버전으로 업데이트할 때 빌드가 중단되는 문제가 나타난다. 이러한 문제를 해결하기 위해, azure-core는 ExpandableStringEnum을 사용한다.
  
  ⛔️ DO NOT!
    * 다음 두 가지의 시나리오를 제외하고, Java의 enum 타입을 사용하지 않는다.  
      1) 값이 정해져서 절대 변하지 않는 경우
      2) 입력으로만 사용되는 경우로, 사용자가 변경사항을 위반할 가능성이 낮은 경우 ??
  
  ✅ DO!
    * Upper-case로 네이밍하라.  
      EnumType.FOO, EnumType.TWO_WORDS = O  
      EnumType.Foo, EnumType.twoWords = X  

 ✔️ YOU MAY?
  * azure-core에 정의되어 있는 ExpandableStringEnum을 사용하라. ??
  
  
🔵 Enum의 문제점?
  
- #### Using Azure Core Types
  ✅ DO!
    Azure SDK 라이브러리들에 일관성 있는 행위를 제공하기 위해, Azure Core에 있는 패키지를 사용하라.
  * HttpClient, HttpPipeline, Response
  * ClientLogger
  * PagedFlux, PagedIterable
  * TokenCredential, AzureKeyCredential
  
- #### Using Primitive Types
  ⛔️ DO NOT!
  * 구 Java date library인 java.util.Date와 java.util.Calendar를 사용하지 말라. 모든 API는 JDK 8부터 나온 java.util.time 패키지를 사용하라.
  * java.net을 노출하는 API를 만들지 말라. 이는 사용하기 어렵고, 사용자들이 빈번하게 사용한다. 대신, URL을 나타낼 때 String 타입을 사용하라. String -> URL 의 과정이 필요한데 파싱에 실패할 때(MalformedURLException), 예외를 catch해서 IllegalArgumentException을 던져라.
  
  ✅ DO!
  * String이나 java.io.File 타입을 사용하지 않고, java.nio.file.Path 타입으로 파일 경로를 표현하라.
  * 하나의 필드를 포함하고 있을 지라도, 의미 있는 도메인 엔티티를 표현하도록 primity type을 감싸야 한다. 예를 들어, 전화번호의 경우 하나의 String 값일지라도, 새로운 타입을 만들어서 primitve String 타입을 감싼다. 이는 표현을 더욱 극대화 해주고, primity type보다 더 강력한 검증/보장을 제공한다.
  
```java
public final class PhoneNumber {
  private String phoneNumber;

  public PhoneNumber setPhoneNumber(String phoneNumber) {
      ...
  }

  public String getPhoneNumber() {
      ...
  }
}
  ```

### Exceptions
  예외 처리는 클라이언트 라이브러리를 구현할 때 중요한 부분이다. 문제를 사용자에게 전달하는 주요 방법으로, 적절한 예외를 개발자에게 전달해야 한다.
  
  ✅ DO!
  * 어떤 HTTP 요청이 스웨커에 등록되지 않은 성공 상태 코드가 아닌 HTTP 상태 코드와 함께 실패했을 때, 예외를 던져라.
  * unchecked 예외를 사용하라.
    * checked = 사용자가 try-catch 코드 블럭을 도입하여 특정한 예외를 직접 처리
    * unchecked = ??
  
  * 다음과 같은 Java 예외를 사용하라.  
    
|Exception   |When to use   |
|---|---|
|IllegalArgumentException   | 메소드의 파라미터가 null 이면 안되는데, 부적절한 경우   |
|IllegalStateException   |    객체가 메소드를 계속 호출할 수 없는 경우|
|NullPointerException   |메소드 인자값이 null이고, 예기치 못한 null인 경우   |
|UnsupportedOperationException   | 객체가 메소드 호출을 지원하지 않을 때    |
  
  ⛔️ DO NOT!
  * 언어의 특정 오류 타입을 충분할 경우, 새로운 에러 타입을 만들지 말아라.
  
  ✅ DO!
  * @throws 어노테이션이 선언된 메소드 안에서 던져지는 모든 checked, unchecked 예외를 구체적으로 명시하라.
  * 새로운 예외 타입 만드는 것을 피하고, Azure core 라이브러리에 존재하는 예외 타입을 사용하라.
  
  * <code>AzureException</code>: 직접 사용하지 않고, 구체적인 하위 타입을 던져라.
    * <code>HttpResponseException</code>: 서비스 요청으로 부터 3xx, 4xx, 5xx와 같이 성공적이지 않은 http 상태 코드와 함께 실패했을 때 사용하라.
      * <code>ClientAuthenticationException</code>: 서비스의 인증에 실패했을 때 사용하라.  
      * <code>DecodeException</code>: 응답 역직렬화 도중에 오류가 발생할 때 사용하라.  
      * <code>ResourceExistsException</code>: HTTP 요청이 이미 존재하는 자원을 만드는 시도할 때 던져라.  
      * <code>ResourceModifiedException</code>: 4xx, 412 conflict과 같이 유효하지 않은 수정  
      * <code>ResourceNotFoundException</code>: 자원이 발견되지 않을 때 412(PUT), 404(GET/POST)  
      * <code>TooManyRedirectsException</code>: HTTP요청이 최대 리다이렉트 시도에 도달했을 때  
    * <code>ServiceResponseException</code>: 요청이 서비스에 전달 되었지만, 클라이언트 라이브라리가 응답을 이해할 수 없을 때 던져라  
    * <code>ServiceRequestException</code>: 커스텀 에러의 정보와 함께 유효하지 않은 응답
  
  ✅ DO!
    * 새로운 서비스에 특정된 예외를 정의 할 때, 위에 정의된 예외를 상속하고 RuntimeException을 직접 상속받지 말라.
    * public API로 부터 던져지는 예외라면, public 패키지에 예외 타입을 정의하라. implementation 패키지나 package private하게 정의된 예외를 던지지 말라.
  
  🔵 Response deserialization?
  
### Authentication
  Azure 서비스는 많고 다양한 인증 스키마를 클라이언트가 서비스에 접근할 수 있도록 사용한다. 개념적으로, credential과 authentication 정책이 있다. credentials는 비밀 데이터를 제공한다. authentication 정책은 서비스의 요청을 인증하기 위한 자격 증명을 제공하는 데이터를 사용한다.
  
  ✅ DO!
  * 
  
  ⛔️ DO NOT!
  * 제공된 토큰의 유효 기간을 넘어서, 토큰을 유지, 캐시, 재사용 하지 말라. 만약 캐싱이 구현되었다면, 토큰 증명이 만료되기 전에 refresh 해야 한다.
  
  ✅ DO!
  * Azure Core 라이브러리에 있는 인증 정책 구현을 사용하라.
  * 드물지만, 서비스가 Azure Active Directory 인증을 지원하지 않는 경우에 토큰 인증이 필요함을 API에 나타내라.
  
     * Azure Active Directory Oauth 서비스들은 커스텀한 인증 스키마를 제공한다. 이러한 경우에 아래 가이드를 따른다.
  
  ✅ DO!
  * 서비스가 지원하는 모든 인증 체계를 지원하라.
  * 클라이언트가 커스텀 인증 체계를 가지고 인증을 요청할 수 있는 공용 사용자 자격 증명 타입을 정의하라.
  
  ⚠️ YOU SHOULD NOT!
  * Azure Core의 토큰 자격 증명 추상 클래스를 상속받거나, 구현해서 커스텀 인증 타입을 정의하면 안된다. 특히 타입 안정성 언어에서, extend나 implement하면 타입 안정성을 손상시켜 사용자가 잘못된 서비스의 사용자 자격 증명을 인스턴스화 할 수 있도록 하기 때문에 해서는 안된다. ??
  
  ✅ DO!
  * Azure Core나 Azure Identity가 아니라 클라이언트와 같은 이름과 패키지, 또는 서비스 그룹 이름과 공유하는 패키지로 커스텀 자격 증명 타입을 정의하라. 
  
  ⛔️ DO NOT!
  * azure-identity 라이브러리에 컴파일 영역 의존성을 취하지 말라.
  
  ✅ DO!
  * 커스텀한 자격증명 유형 이름 앞에 서비스 이름이나 서비스 그룹이름을 추가하여 의도적으로 범위와 사용에 대해서 명확하게 나타내라.
  
  * 커스텀한 자격 증명 유형의 이름 끝에 Credential을 추가하라. 복수형(Credentials)이 아니라 단수형이어야 한다.
  * 커스텀 인증 프로토콜에 필요한 모든 테이터를 가지고 있는 커스텀 자격 증명 유형에 대한 생성자나 factory를 정의하라.
  * 데이터 변동을 받아들이는 update 메소드를 정의하고 원자성, 스레드 안정성을 갖추어 자격 증명 데이터를 업데이트 하라.
  
  ⛔️ DO NOT!
  * 사용자가 직접 데이터를 업데이트할 수 있게 되면 원자성을 깨버리기 때문에 public으로 필드를 set 할 수 있게 하지 말라.
  
  ⚠️ YOU SHOULD NOT!
  * 사용자가 인증체계 데이터에 직접 접근할 수 없도록, 프로퍼티를 public 형태로 정의하지 말라. 최종 사용자는 필요로 하지않는 경우가 거의 대부분이고, 스레드 안정성을 사용하기가 어렵다. 인증 데이터를 노출해야만 하는 경우, 인증 요청에 필요한 모든 데이터는 일관된 상태로 데이터를 리턴하는것을 보장하는 단일 API로 제공해야 한다.
  
  ✅ DO!
  * 모든 자격 증명 타입을 받아들이는 서비스 클라이언트 생성자나 팩토리를 제공하라.
  
  클라이언트 라이브러리는 서비스가 사용자에게 포탈이나 다른 도구들을 통해서 연결 문자열을 제공하는 경우에서만 연결 문자열을 통해서 인증체계 데이터를 제공하는 것을 지원한다. 연결 문자열은 포탈에서 복사/붙여넣기를 통해 애플리케이션에 쉽게 통합할 수 있도록 해준다. 하지만, 연결 문자열은 인증 정보가 실행중인 프로세스 내에서 순환될 수 없기 떄문에 낮은 인증 형태로 여겨진다.
  ??
  
  ⛔️ DO NOT!
  * Azure Potal 또는 CLI에서 이러한 문자열을 사용할 수 없다면, 연결 문자열을 가지고 서비스 클라이언트를 생성하는 것을 지원하지 말라.
  
  
### Namespaces
  클라우드 환경에서 서비스를 그룹화 하는 것은 검색과 참조 문서의 구조를 제공하는 것을 목표로 한다.??
  Java에서 namespace는 <code>com.azure.\<group\>.\<service\>p.\[\<feature\>\]</code> 로 정의한다.
  사용자와 맞닿아 자주 사용되는 모든 API는 패키지 구조에 존재한다.
  
  * <code>\<group\></code> 은 서비스의 그룹  
  * <code>\<service\></code> 은 한 단어로 표현되는 서비스 이름  
  * <code>\<feature\></code> 은 선택적인 서브 패키지로, 서비스를 여러 컨포넌트로 분리한 것 storage -> blob/ files/ queues  
  
  ✅ DO!
  * Azure 클라이언트 라이브러리를 나타낼 때는 com.azure 패키지로 시작하라.  
  * 패키지 이름은 모두 lowercase로 하고 스페이스, 하이픈, 언더바 등을 포함하지 않는다.  
  * 사용자가 사용하는 서비스와 연결되는 패키지로 패키지을 설정하라. 변경될 수 있는 패키지 이름은 피하라.  
  * management API는 resourcemanager 그룹에 위치시켜라. com.azure.resourcemanager.<service>로 이름을 설정하라.
 
  
### Support for Mocking
  ✅ Do!
  * mock 클라이언트가 서비스 클라이언트에 대해서 non-live 테스트가 가능하도록 지원하라. ??
  * 모든 IO 동작의 mocking이 가능하도록 지원하라.
