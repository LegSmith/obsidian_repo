# BindingReuslt

---

**==스프링이 제공하는 검증 오류 처리를 도와주는 클래스이다.==**

- `BindingResult` 파라미터는 반드시 오류 검증을 하는 객체 바로 뒤에 와야 한다.

    ```java
    @PostMapping("/add")
    public String addItemV1(@ModelAttribute Item item,
                          BindingResult bindingResult,
                          RedirectAttributes redirectAttributes,
                          Model model) {
    	//생략...
    }
    ```

- `addError()` : 오류를 담아두는 메서드이다. **==FiledError 객체에 오류내용을 담아서 BindingResult 에 추가해준다.==**
    - `objectName` : 오류검증을 하는 객체 이름을 지정해준다.
    - `field` : 오류가 발생할 필드 이름
    - `defaultMessage` : 오류 메시지
- 예를 들어 상품이름을 적는 필드에 오류가 발생한다면

    - 검증 객체 이름은 item 이고 오류가 발생하는 필드이름은 `itemName` 이고 메시지를 FieldError에 담아서 bindingResult 에 추가해준다.

    ```java
    if(!StringUtils.hasText(item.getItemName())){
        bindingResult.addError(new FieldError("item","itemName","상품 이름은 필수입니다."));
    }
    ```


> `==BindingResult==` **==는 스프링이 알아서 View 까지 넘겨주기 때문에==** `==BindingResult==`**==에==** `==Error==` **==만 추가해주면 된다.==**

- 특정 필드가 아닌 오류는 ObjectError 에 담아서 보내면 된다.

    ```java
    if(item.getPrice() != null && item.getQuantity() != null){
        int resultPrice = item.getPrice() * item.getQuantity();
        if(resultPrice < 10000){
            bindingResult.addError(new ObjectError("item","가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
        }
    }
    ```


==**실제 addError 는 ObjectError 만 담는다. FiledError를 담을 수 있는 이유는 ObjectError 의 자식 클래스이기 때문이다.**==

## 타임리프 스프링 검증 오류 통합기능

==**타임리프에서는 검증오류를 표현하는 간편한 기능을 제공한다.**==

- `${#fileds.hasGlobalErrors()}` : 글로벌 오류를 처리할 때 사용

    - `#fileds` 를 통해 `BindingResult` 가 제공하는 검증오류에 접근할 수 있다.

    ```java
    <div th:if="${\#fields.hasGlobalErrors()}">
      <p class="field-error" th:each="err : ${\#fields.globalErrors()}" th:text="${err}">전체 오류 메시지</p> 
    </div>
    ```

- `th:errors` : 해당 필드에 오류가 있는 경우에 태그를 출력(th:if 와 비슷한 기능)

    - `BindingResult`에 `Error` 를 담을 때 필드네임을 담았기 때문에 바로 지정가능하다.

    ```java
    <div class="field-error" th:errors="*{quantity}">
        수량 오류
    </div>
    ```

- th:errorclass : th:field 에 지정한 필드에 오류가 있으면 class 정보를 추가한다.

    ```java
    th:errorclass="field-error"
    ```


## BindingResult 정리

**==스프링이 제공하는 검증 오류를 보관하는 객체이다==**

1. `@ModelAttribute` 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 `FieldError` 를 생성해서 `BindingResult` 에 넣어준다.
2. 개발자가 직접 로직을 짜서 넣어준다
3. `Validator` 사용

# **FieldError, ObjectError**

---

**==FieldError 와 ObjectError 에는 다양한 생성자가 있다. 사용자가 잘못된 값을 입력해도 값을 유지해주는 필드가 있다==**

- `objectName` : 오류가 발생하는 객체 이름
- `field` : 오류 필드
- `rejectedValue` : 사용자가 입력한 값을 저장해주는 필드이다.
- `bindingFailure` : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
- `codes` : 메시지 코드
- `arguments` : 메시지에서 사용하는 인자
- `defaultMessage` : 기본 오류 메시지

    ```java
    public FieldError(String objectName, 
    									String field, 
    									@Nullable Object rejectedValue, 
    									boolean bindingFailure, 
    									@Nullable String[] codes, 
    									@Nullable Object[] arguments, 
    									@Nullable String defaultMessage)
    ```


### 타임리프의 꿀 기능

- 컨트롤러 단에서 `rejectedValue` 로 사용자가 잘못된 값을 입력하여도 값을 저장해 유지하게 설정했다.
- 타임리프에서 `th:field` 는 **==정상적으로 동작할 때는 모델 객체의 값을 사용하고 오류가 발생하면 FieldError 에서 보관한 값을 사용해서 값을 출력한다.==**

# 오류 코드와 메시지 처리

---

## FieldError

- `FieldError` 의 `codes` , `arguments` 필드를 이용하여 메시지와 메시지에 들어오는 값을 지정할 수 있다.
- 우선 스프링이 제공하는 메시지기능을 이용한다.
- `resources/errors.properties` 를 생성하자

    ```xml
    required.item.itemName=상품 이름은 필수입니다.
    range.item.price=가격은 {0} ~ {1} 까지 허용합니다. 
    max.item.quantity=수량은 최대 {0} 까지 허용합니다. 
    totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
    ```

- codes 는 String[] 타입이고 arguments 는 Object[] 타입이다.

    ```java
    bindingResult.addError(new FieldError("item",
    											 "price",
    											 item.getPrice(),
    											 false,
    											 new String[]{"range.item.price"},
    											 new Object[]{1000,100000},
    											 null));
    ```


> _**코드가 매우 지저분하고 복잡해보인다는 단점이 있다.**_

## rejectValue(), reject()

_**FieldError 로 에러를 담는것은 필드가 늘어남에 따라 코드가 복잡해지기 때문에**_ `rejectValue()`_**,**_ `reject()` _**를 사용해서 코드를 간단하게 바꿀 수 있다.**_

- 우선 BindingResult 는 검증할 객체의 정보를 알고 있다. 그래서 굳이 객체명을 적을 필요가 없어서 `rejectValue()` 나 `reject()` 를 사용하면 된다.
    - `argument` 가 없는 경우에는 필드명과 에러메시지만 적어주면 된다.(에러메시지는 추후에 `messageResolver` 를 이해하면 이해가 간다.)
    - 에러메시지에서는 원래 스프링 메시지에 `required.item.itemName` 를 적어줬지만 required 만 적어주면 `messageResolver` 가 자동으로 바인딩해준다.

        ```java
        bindingResult.rejectValue("itemName", "required");
        ```

    - `argument` 가 있는 경우라면 Object[] 타입의 argument 와 디폴트 메시지를 적어주면 된다.

        ```java
        bindingResult.rejectValue("price","range",new Object[]{1000,1000000},null);
        ```

    - `reject()` 는 따로 필드명이 없기 때문에 에러메시지와 아규먼트만 적어주면 된다.

        ```java
        bindingResult.reject("totalPriceMin",new Object[]{10000,resultPrice},null);
        ```


## MessageCodesResolver

`MessageCodesResolver` _**는 검증 오류 코드로 메시지 코드들을 생성하는 인터페이스이다.**_

- 보통 구현체로 `DefalutMessageCodesResolver` 로 생성한다.
- 오브젝트 이름과 필드명으로 메시지 코드들을 생성할 수 있다.

    - 오브젝트 이름을 통해서 에러코드와 오브젝트명을 조합해서 **==디테일한것부터 먼저 만들고 심플한걸 나중에 만든다.==**

    ```java
    @Test
    void messageCodesResolverObject(){
        String[] messageCodes = codesResolver.resolveMessageCodes("required", "item");
        for (String messageCode : messageCodes) {
            System.out.println("messageCode = " + messageCode);
        }
        assertThat(messageCodes).containsExactly("required.item", "required");
    }
    ```

    ![[Untitled 6.png|Untitled 6.png]]

- 필드 명일때는 좀 더 많이 생성된다.

    - 에러코드와 오브젝트명 필드명을 조합해서 디테일 → 심플 순으로 메시지 코드를 생성한다.

    ```java
    @Test
    void messageCodesResolverField(){
        String[] messageCodes = codesResolver.resolveMessageCodes("required", "item", "itemName", String.class);
        for (String messageCode : messageCodes) {
            System.out.println("messageCode = " + messageCode);
        }
        assertThat(messageCodes).containsExactly(
                "required.item.itemName",
                "required.itemName",
                "required.java.lang.String",
                "required"
        );
    }
    ```

    ![[Untitled 1 5.png|Untitled 1 5.png]]


### 동작방식

- `rejectValue()`, `reject()` 내부에서 `MessageCodesResolver` 를 사용하여 메시지 코드들을 생성한다.
- 메시지 코드들을 `BindingResult` 가 가지게 되고 타임리프에서 화면을 랜더링 할때는 `th:errors` 가 실행되면서 생성된 ==**오류 메시지 코드를 스프링 메시지 파일(errors.properties ) 에서 순서대로 돌아가면서 메시지를 찾는다.**==
- 없으면 디폴트 메시지를 출력

# Validator 분리

---

_복잡한 검증 코드를 컨트롤러에 넣어두기 보다는 별도의 클래스에 몰아 넣어서 필요시에 불러와서 쓰는게 코드가 클린해진다._

- `ItemValidator` 라는 클래스를 만들고 스프링이 제공하는 `Validator` 를 상속받자(이유는 다음에)

    - `Validtor` 가 체계적으로 검증하기 위해 `supports()` , `validate()` 메서드를 제공한다.
    - supports(Class<?> clazz) : 해당 검증기를 지원하는 여부 확인
    - `validate(Object target, Errors errors)` : **==검증 대상 객체(target) 과 BindingResult(Errors 의 자식클래스)==**

    ```java
    @Component
    public class ItemValidator implements Validator {
        @Override
        public boolean supports(Class<?> clazz) {
            return Item.class.isAssignableFrom(clazz); // 자식클래스까지 커버가된다.
            //item == clazz
            //item == subItem
        }

        @Override
        public void validate(Object target, Errors errors) {
            Item item = (Item) target;
            // 검증 로직
            if(!StringUtils.hasText(item.getItemName())){
                errors.rejectValue("itemName","required");
            }
            if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000){
                errors.rejectValue("price","range",new Object[]{1000,1000000},null);
            }
            if(item.getQuantity() == null || item.getQuantity() >= 9999){
                errors.rejectValue("quantity","max", new Object[]{9999},null);
            }
            // 특정 필드가 아닌 복합 룰 검증
            if(item.getPrice() != null && item.getQuantity() != null){
                int resultPrice = item.getPrice() * item.getQuantity();
                if(resultPrice < 10000){
                    errors.reject("totalPriceMin",new Object[]{10000,resultPrice},null);
                }
            }
        }
    }
    ```


## WebDataBinder

- `WebDataBinder` 는 스프링의 파라미터 바인딩의 역할과 검증 기능이 내부에 포함되어 있다.
- `@InitBinder` 어노테이션으로 컨트롤러 실행전에 WebDataBinder 에 검증기를 추가해준다.

    ```java
    @InitBinder
    public void init(WebDataBinder dataBinder){ 
        dataBinder.addValidators(itemValidator);
    }
    ```

- 그리고 검증할 객체 앞에 @Validated 어노테이션을 붙혀주면 WebDataBinder 에 등록된 검증기를 찾아서 실행한다.

    ```java
    @PostMapping("/add")
    public String addItemV6(@Validated @ModelAttribute Item item,
                            BindingResult bindingResult,
                            RedirectAttributes redirectAttributes,
                            Model model) {
    	// 생략...
    }
    ```


만약, 여러 타입의 검증기를 등록한다면 어떤 검증기를 실행되어야 할지 구분이 필요해진다. 이 때 스프링이 제공하는 `Validator` 클래스가 제공하는 `supports()` 메서드로 검증기를 구분해준다.

## 관련 문서

- 상위 목차: [[스프링 MVC 2편]]
- 이전 문서: [[섹션 3 - 메시지, 국제화]]
- 다음 문서: [[섹션 5 - 검증2 - Bean Validation]]
