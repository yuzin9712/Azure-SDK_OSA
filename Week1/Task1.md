# Task1
> Azure SDK Design Guidelines 부분 번역하면서 정리하기  
> [관련 자료 링크](https://azure.github.io/azure-sdk/java_introduction.html)


## Azure SDK API Design

### Supporting Types
- #### Model Types
> Model Type은 Azure 서비스로 부터 제공하거나 받아야 하는 정보를 class 단위로 정의한 것을 말함

⛔️ model 클래스를 위한 builder 클래스를 따로 만들지 않는다.  
  
✅ 
   * 사용자가 인스턴스화 할 수 있는 모든 모델 클래스는 public 생성자를 제공한다. 하지만, 서비스로 부터 리턴되는 모델타입과 같이 사용자가 인스턴스화할 수 없는 클래스들은 어떠한 public 생성자도 사용하면 안된다.  
      
   🤔 Why?
    +++++++
    
   * final이 붙은 required property를 갖지 않았다면, 인자값이 없는 기본 생성자를 제공해야 한다.
   * 만약 모델이 required property를 가진다면, 인자를 갖는 1개 이상의 생성자를 만든다.
   +++++

🔵 Mutually Exclusive?
++++++++

```
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
```
++++
✅
  * fluent setter API를 사용하고, set 메소드는 this를 리턴해야 한다.  
  * extended type을 리턴하기위해 fluent type을 extends 할 때, 모든 set 메소드들을 오버라이드 해야 한다.  
  = 아래 예시 코드에서 AbandonOptions가 SettlementOptions를 상속 받았고, SettlementOptions의 모든 set 메소드를 Override하고 AbandonOptions 타입을 리턴하고 있다.
  
  ```
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

✅
  * 클래스에 @Fluent 어노테이션을 붙인다.
  * JavaBean 컨벤션에 따라서 getXXX, setXXX 그리고 isXXX를 사용한다.
  * raw 데이터로 부터 새로운 모델 인스턴스가 필요할 경우, static 메소드를 포함하고, 그 형식은 from<dataformat> 이다.
  = ++++
  
⛔️
  * 
++++

🔵 @Fluent?  
🔵 Chaining of set operations?


- #### Enumerations
  
  
🔵 Enum의 문제점?
  Enum은 열거자가 암시적으로 정수로 변환된다는 특징이자 문제점이 있다.
  
- #### Using Azure Core Types
- #### Using Primitive Types

### Exceptions
### Authentication
### Namespaces
### Support for Mocking

