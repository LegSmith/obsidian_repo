## Bean Validation 이란 ?

---

_==**Bean Validation 이란 특정한 구현체가 아닌 Bean Validation 2.0(JSR-380) 이라는 기술 표준**==_

- 검증 어노테이션과 여러 인터페이스의 모음이다. → JPA 가 표준 기술이고 Hibernate 가 구현체로 있는것
- `Bean Validation` 을 구현한 기술중에는 하이버네이트 `Validator` 라는 구현체를 사용한다.

## 순수한 Bean Validation 사용하기

---

### 검증 어노테이션

==_**이전에 직접 Validator(검증기)를 생성했을 때는 itemName을 반드시 받고 price 는 1000~100000 사이 값, quantity 는 9999 이하로만 받는 규칙이 있었다. 이를 어노테이션으로 간편하게 설정할 수 있다.**_==

- `@NotBlank` : 빈값 , 공백만 있는 경우를 허용하지 않는다.
- `@NotNull` : null 을 허용하지 않는다.
- `@Range(min = 1000, max = 100000)` : 1000~ 100000 범위 안의 값이어야 한다.
- `@Max(9999)` : 최대 9999까지만 허용
    
    ```java
    @Data
    public class Item {
    
        private Long id;
    
        @NotBlank
        private String itemName;
    
        @NotNull
        @Range(min = 1000, max = 100000)
        private Integer price;
    
        @NotNull
        @Max(9999)
        private Integer quantity;
    }
    ```
    

> 검증 어노테이션 모음
> 
> > [!info] Hibernate Validator 6.2.5.Final - Jakarta Bean Validation Reference Implementation: Reference Guide  
> > Hibernate Validator, Annotation based constraints for your domain model - Reference Documentation  
> > [https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)  

### 테스트 코드

- 위에 2줄은 이해하지 않아도된다.
- 그냥 validator 에 객체를 넣어 검증하는 코드이다.
    
    ```java
    @Test
    void beanValidation() {
    		ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    		Validator validator = factory.getValidator();
    		Item item = new Item(); item.setItemName(" "); //공백
    		item.setPrice(0);
    		item.setQuantity(10000);
    		Set<ConstraintViolation<Item>> violations = validator.validate(item);
    		for (ConstraintViolation<Item> violation : violations) {
    		    System.out.println("violation=" + violation);
    		    System.out.println("violation.message=" + violation.getMessage());
    		}
    }
    ```
    
    ![[Untitled 7.png|Untitled 7.png]]
    

## Bean Validation - 스프링 통합

---

- 따로 Validator 를 생성할 필요도 없고 검증할 객체 앞에 `@Validated` , `@Valid` 만 붙혀주면 스프링이 알아서 검증을 해준다.
    
    ```java
    @Slf4j
    @Controller
    @RequestMapping("/validation/v3/items")
    @RequiredArgsConstructor
    public class ValidationItemControllerV3 {
    
        private final ItemRepository itemRepository;
    
    		//생략 ...
        @GetMapping("/add")
        public String addForm(Model model) {
            model.addAttribute("item", new Item());
            return "/validation/v3/addForm";
        }
    
        @PostMapping("/add")
        public String addItemV6(@Validated @ModelAttribute Item item,
                                BindingResult bindingResult,
                                RedirectAttributes redirectAttributes,
                                Model model) {
    
            // 검증에 실패하면 다시 입력 폼으로
            if(bindingResult.hasErrors()){
                log.info("errors = {}",bindingResult);
                return "/validation/v3/addForm";
            }
    
            // 성공 로직
            Item savedItem = itemRepository.save(item);
            redirectAttributes.addAttribute("itemId", savedItem.getId());
            redirectAttributes.addAttribute("status", true);
            return "redirect:/validation/v3/items/{itemId}";
        }
    		//생략...
    }
    ```
    
- `build.gradle` 파일에 ==_**Bean Validation 라이브러리를 추가해주면 스프링부트가 Bean Validation 을 인지하고 스프링에 통합**_==한다.
- `LocalValidatorFactoryBean` 을 글로벌 Validator 로 등록해서 `@Validated` 나 `@Valid` 가 붙은 객체를 검증을 한다.
- 검증 오류가 발생하면 BindingResult 에 등록하는 일 까지 스프링부트가 다 해준다.

### 검증 순서

1. `@ModelAttribute` 어노테이션이 각각의 필드을 엔터티에 주입
    1. 성공하면 다음단계
    2. 실패하면 typeMismatch 로 FieldError 추가
2. Validator 적용

## Bean Validation - 에러 코드

---

Bean Validation 을 적용하고 검증 오류 코드를 보면 해당 오류 코드가 어노테이션 이름으로 등록된다. 즉 ==_**어노테이션 이름을 기반으로**_== `==MessageCodeResolver==` ==_**가 메시지 코드를 생성하는 것이다.**_==

- 에러 코드를 보면 `codes [NotBlank.item.itemName, ….]` 으로 되있는걸 알 수 있다.
    
    ```bash
    Field error in object 'item' on field 'itemName': rejected value []; codes [NotBlank.item.itemName,NotBlank.itemName,NotBlank.java.lang.String,NotBlank]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [item.itemName,itemName]; arguments []; default message [itemName]]; default message [공백 안됩니다.]
    ```
    
- `errors.properties` 파일에 메시지를 아래와 같이 등록해주면 잘 작동한다.
    
    ```bash
    \#Bean Validation 추가 
    NotBlank={0} 공백X 
    Range={0}, {2} ~ {1} 허용 
    Max={0}, 최대 {1}
    ```
    

### Bean Validation 메시지 찾는 순서

1. 생성된 메시지 코드 순서대로 messageSource 에서 찾기
2. 어노테이션의 message 속성 사용 → `@NotBlank(message = “공백 안되요”)`
3. 라이브러리가 제공하는 기본 값 사용

## Bean Validation - 오브젝트 오류

---

- Bean Validation 없이 오브젝트 오류를 구현할 때는 아래와 같은 로직을 짜서 `bindingResult` 에 담아줬다.
    
    ```java
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.reject("totalPriceMin", new Object[]{10000,
                    resultPrice}, null);
        }
    }
    ```
    
- Bean Validation 에는 `@ScriptAssert()` 라는 어노테이션으로 오브젝트 검증 오류를 설정할 수 있다.
    
    ```java
    @ScriptAssert(
            lang = "javascript",
            script = "_this.price * _this.quantity >= 10000",
            message = "총합이 10,000원이 넘어야 합니다."
    )
    ```
    
- `@ScriptAssert()` 는 제약이 많고 복잡하기 떄문에 그냥 오브젝트 검증 로직을 직접 구현하는게 훨씬 대응하기 편해진다.

## Bean Validation - 한계

---

==_**수정시에 ID값을 필수로 받고 수량을 무한대로 늘릴 수 있는 요구사항이 추가됐다고 가정하자 !**_==

- Item 도메인에서 아래와 같이 수정해주면 위의 요구사항이 충족된다.
    
    ```java
    @Data
    
    public class Item {
    
        @NotNull // 수정 요구사항(수정할 떄 Id값 필수)
        private Long id;
    
        @NotBlank(message = "공백을 입력할 수 없습니다.")
        private String itemName;
    
        @NotNull
        @Range(min = 1000, max = 100000)
        private Integer price;
    
        @NotNull
        //@Max(9999) // 수정 요구사항(수정 시 수량 무제한)
        private Integer quantity;
    		
    		// 생략...
    }
    ```
    
- 하지만 ==**등록 시에는 ID 값을 받지 않기 때문에 오류가 생긴다 → Side Effect(부작용) 발생**==
- 이러한 문제가 Bean Validation 의 한계이다.

## Bean Validation - groups

---

_==**동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하는 방법이다.**==_

### 해결 방법

1. ==**BeanValidation 의 groups 기능**==을 사용
2. Item 도메인을 직접 사용하지 않고 ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용

### Group 생성

- 등록할때와 수정할때 2가지의 그룹이 필요하기 때문에 인터페이스를 2개 생성하자.
    
    ```java
    public interface UpdateCheck {
    }
    
    public interface SaveCheck{
    }
    ```
    

### Group 적용

- 등록시에는 itemName 과 price , quantity 의 검증이 필요하고 수정 시에 id, itemName, price 에 검증이 필요하므로 Item 도메인을 아래와 같이 수정해준다.
    
    ```java
    @Data
    public class Item {
    
        @NotNull(groups = UpdateCheck.class) // 수정 요구사항(수정할 떄 Id값 필수)
        private Long id;
    
        @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
        private String itemName;
    
        @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
        @Range(min = 1000, max = 100000, groups = {SaveCheck.class, UpdateCheck.class})
        private Integer price;
    
        @NotNull
        @Max(value = 9999, groups = SaveCheck.class)
        private Integer quantity;
    
    		//생략 ..
    }
    ```
    
- 그리고 수정 컨트롤러에는 `@Validated(UpdateCheck.class)` , 저장 컨트롤러에는 `@Validated(SaveCheck.class)` 로 바꿔준다.
    - 단, `_**@Valid**_` _**에는 적용이 안된다.**_

> 하지만 도메인도 복잡해지고 코드가 전반적으로 복잡해지기 때문에 groups 기능은 잘 사용하지 않는다.

## Form 전송 객체 분리

---

_==**별도의 DTO 를 생성하여 컨트롤러에서 받고 Repository 로 넘길 때에 Entity 로 변환하면 된다 !!**==_

- `ItemSaveForm.java` → 요구사항에 맞게 검증 어노테이션을 설정해준다.
    
    ```java
    @Data
    public class ItemSaveForm {
        @NotBlank
        private String itemName;
    
        @NotNull
        @Range(min=1000,max=1000000)
        private Integer price;
    
        @NotNull
        @Max(value = 9999)
        private Integer quantity;
    }
    ```
    
- [`ItemUpdateForm.java`](http://ItemUpdateForm.java) → 요구사항에 맞게 검증 어노테이션을 설정해준다.
    
    ```java
    @Data
    public class ItemUpdateForm {
    
        @NotNull
        private Long id;
    
        @NotBlank
        private String itemName;
    
        @NotNull
        @Range(min=1000,max=1000000)
        private Integer price;
    
        // 수정 시 수량은 자유롭게
        private Integer quantity;
    
    }
    ```
    
- 컨트롤러에서는 `@ModelAttribute` 에 키값을 조심해주면 된다. ==**뷰 템플릿에서 item 이라는 키로 매핑되어 있기 때문에 item 키 를 설정해주고 DTO 로 받아준다.**==
- Item 엔티티를 new 로 선언하여 setter로 받을 수 있지만 ==**setter 는 실무에서 쓰면 위험하기 때문에 DTO 를 파라미터로 받아 Item 객체로 변화하는 로직을 추가**==하여 컨트롤러에서 사용한다.
    
    ```java
    @Data
    public class Item {
    
    		//생략...
    
        public void toSaveForm(ItemSaveForm form){
            itemName = form.getItemName();;
            price = form.getPrice();
            quantity = form.getQuantity();
        }
    
        public void toUpdateForm(ItemUpdateForm form){
            itemName = form.getItemName();;
            price = form.getPrice();
            quantity = form.getQuantity();
        }
    }
    ```
    
- DTO → Entity 변환후 Repository 로 넘겨주면 끝 !!
    
    ```java
    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form,
                          BindingResult bindingResult,
                          RedirectAttributes redirectAttributes) {
    
        if (form.getPrice() != null && form.getQuantity() != null) {
            int resultPrice = form.getPrice() * form.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000,
                        resultPrice}, null);
            }
        }
    
        // 검증에 실패하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
            log.info("errors = {}", bindingResult);
            return "/validation/v4/addForm";
        }
        // 엔티티 변환
        Item item = new Item();
        item.toSaveForm(form);
    
        // 성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v4/items/{itemId}";
    }
    ```
    

## Bean Validation - HTTP 메시지 컨버터

---

`@ModelAttribute` 어노테이션은 HTTP 요청 파라미터를 다룰 떄 사용  
`@RequestBody` 어노테이션은 HTTP Body의 데이터를 다룰 떄 사용(주로 API JSON)

### API 통신할 때 Bean Validation

- API 통신을 할 때는 반환값이 View 가 아닌 Data 이므로 `@RestController` 를 선언해준다.
- 그리고 `@ModalAttribute` 가 아닌 `@RequestBodty` 를 객체 앞에 선언해서 ==**HTTP 메시지 바디에 있는 데이터를 객체와 바인딩한다.**==
    
    ```java
    @Slf4j
    @RestController
    @RequestMapping("/validation/api/items")
    public class ValidationItemApiController {
    
        @PostMapping("/add")
        public Object addItem(@RequestBody @Validated ItemSaveForm form, BindingResult bindingResult){
            log.debug("API 컨트롤러 호출");
    
            if(bindingResult.hasErrors()){
                log.error("검증 오류 발생 errors={}",bindingResult);
                return bindingResult.getAllErrors();
            }
            log.info("성공 로직 실행");
            return form;
        }
    }
    ```
    

### API 통신 검증시 3가지 경우

1. 성공
    
    ![[Untitled 1 6.png|Untitled 1 6.png]]
    
2. 검증 실패
    
    ![[Untitled 2 5.png|Untitled 2 5.png]]
    
3. `@RequestBody` 로 객체 바인딩 실패
    - ==**JSON 객체로 변환이 안되면 컨트롤러 진입 자체가 안되서 예외가 발생한다.**==
        
        ![[Untitled 3 3.png|Untitled 3 3.png]]
        

==**검증실패가 아닌**== `==HttpMessageConverter==` ==**단계에서 실패하면 컨트롤러 접근조차 안되기 때문에 예외처리를 해줘야 한다.**==