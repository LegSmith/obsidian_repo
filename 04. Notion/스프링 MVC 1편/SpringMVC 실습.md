SpringBoot 2.7.14  
Java 11  
Spring Web, Lombok, Thymeleaf

# Thymeleaf 사용법

### 타임리프 사용 선언

- 코드 상단에 아래와 같이 한 줄 추가해야한다
    
    ```xml
    <html xmlns:th="http://thymeleaf.org"> 
    ```
    

대부분의 속성 변경은 th: xxx 로 변경가능하다!!

  

## 타임리프는 네추럴 템플릿(Natural Templates)

- ==_**타임리프는 th:xxx 가 붙은 부분은 서버사이드랜더링(SSR)이 되고, 기존 것을 대체**_==한다. th:xxx 가 없으면 기존 html xxx속성이 그대로 사용된다.
- 서버를 가동하지 않고 HTML 파일을 열었을때 html 코드에 th:xxx 가 있어도 th는 무시하고 열기 때문에 _**==순수 HTML 을 그대로 유지하면서 뷰 템플릿도 사용가능하다.==**_

### URL 링크 표현식 (1)

- 타임리프는 URL 링크를 사용할 경우 @{~~} 를 사용한다.
    
    ```html
    th:href="@{/css/bootstrap.min.css}"
    ```
    

### URL 링크 표현식 (2)

- 경로변수(@PathVariable)에 값을 링크 안에 넣을 때 사용한다
    
    ```html
    th:href="@{/basic/items/{itemId}(itemId=${item.id})"
    ```
    
- 쿼리 스트링(쿼리 파라미터) 도 생성한다
    
    ```html
    th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"<!--예시-->
    	<!--생성링크
    http://localhost:8080/basic/items/1?query=test
    -->
    ```
    
- URL 간단하게 표현(리터럴)
    
    ```html
    th:href="@{|/basic/items/${item.id}|}"
    ```
    

### onclick 표현 - th:onclick

```html
onclick="location.href='addForm.html'" <!-- 원래 표현 -->

th:onclick="|location.href='@{basic/items/add}'|" <!-- 타임리프 문법 -->
```

### 리터럴 대체 - | ~~ |

```html
<span th:text=" 'Welcome to our application, '+${user.name}+'!'"> <!-- 리터럴 사용x -->
<span th:text="|Welcome to our application, ${user.name}!!|"> <!-- 리터럴사용 -->
```

### 반복 출력 - th:each

- 반복은 th:each 를 사용하고, items 컬렉션 데이터가 item 변수에 하나씩 포함되고 반복문 안에서 item 변수를 사용

```html
<tr th:each="item : ${items}">
```

  

---

# 상품 도메인 개발(테스트까지)

---

### Item

- `@Data` 어노테이션은 `@Setter, @Getter @toString` 등 많은 어노테이션이 들어가있어 간편하다.
- 하지만 실제 실무에서는 `@Setter`를 사용안해야 할 때도 있고 ==**상황마다 다르기 때문에** `@Data` **어노테이션을 안쓰는게 좋다.**==
    
    ```java
    @Data
    public class Item {
    
        private Long id;
        private String itemName;
        private Integer price; // Null 을 방지하기 위해
        private Integer quantity;
    
        public Item() {
        }
    
        public Item(String itemName, Integer price, Integer quantity) {
            this.itemName = itemName;
            this.price = price;
            this.quantity = quantity;
        }
    }
    ```
    

### ItemRepository

- HashMap 은 스레드 동시성때문에 쓰면 안된다.
- sequence 도 long 보다는 AtomicLong 을 사용해야한다.
- 그리고 update() 메서드에서도 DTO 객체를 별도로 생성해서 사용해야 한다.
    - ==**실제 엔티티는 안건드는게 좋다.**==
- clearStore()는 테스트할때 사용할려고 만들었다.
    
    ```java
    @Repository
    public class ItemRepository {
    
        private static final Map<Long, Item> store = new HashMap<>();
        // 동시에 여러 스레드가 접근하면 HashMap 을 사용하면 안된다.
        // 사용할려면 ConcurrentHashMap 을 사용해야한다. 이유는?
    
        private static long sequence = 0L;
        // 얘도 동시성 문제때문에 long 이 아닌 AtomicLong 을 사용해야 한다.
    
        public Item save(Item item) {
            item.setId(++sequence);
            store.put(item.getId(),item);
    
            return item;
        }
    
        public Item findbyId(Long id){
            return store.get(id);
        }
        public List<Item> findAll(){
            return new ArrayList<>(store.values());
            //ArrayList 로 감싼 이유는 ArrayList 에 값이 추가되도 실제 store 에는 영향이 없기 때문
        }
        public void update(Long itemId, Item updateParam){
            Item findItem = findbyId(itemId);
            findItem.setItemName(updateParam.getItemName());
            findItem.setPrice(updateParam.getPrice());
            findItem.setQuantity(updateParam.getQuantity());
            //setter 보다는 별도의 DTO 객체를 만들어서 사용하는게 좋다.
        }
    
        public void clearStore(){
            store.clear(); // test 를 위해 생성
        }
    }
    ```
    

### ItemRepositoryTest

- `@AfterEach` ==**어노테이션은 테스트 하나가 끝나고 실행할 로직을 실행**==시켜주는 어노테이션이다.
    
    ```java
    public class ItemRepositoryTest {
    
        ItemRepository itemRepository = new ItemRepository();
    
        @AfterEach
        void afterEach(){
            itemRepository.clearStore();
        }
    
        @Test
        void save(){
            //given
            Item item = new Item("itemA",10000,10);
    
            //when
            Item savedItem = itemRepository.save(item);
    
            //then
            Item findItem = itemRepository.findbyId(item.getId());
            assertThat(findItem).isEqualTo(savedItem);
        }
    
        @Test
        public void findAll(){
            //given
            Item item1 = new Item("itemA",10000,10);
            Item item2 = new Item("itemB",20000,20);
    
            itemRepository.save(item1);
            itemRepository.save(item2);
    
            //when
            List<Item> result = itemRepository.findAll();
    
            //then
            assertThat(result.size()).isEqualTo(2);
            assertThat(result).contains(item1,item2);
    
        }
    
        @Test
        void updateItem(){
            //given
            Item item = new Item("item1", 10000, 10);
    
            Item savedItem = itemRepository.save(item);
            Long itemId = savedItem.getId();
    
            //when
            Item updateParam = new Item("item2", 20000, 30);
            itemRepository.update(itemId, updateParam);
    
            //then
            Item findItem = itemRepository.findbyId(itemId);
            assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
            assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
            assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());
        }
    
    
    }
    ```
    

  

# 상품 목록

---

### BasicController

- `@RequireArgsConstructor` 어노테이션은 ==**final 이 붙은 변수의 생성자를 자동으로 만들어준다**==. → 즉, ItemRepository가 ==**의존관계 주입(DI)**==이 된다.
- `@PostConstruct` 어노테이션으로 ==**스프링 빈의 의존관계가 모두 주입된 후 초기화 용도**==로 호출된다.
    
    ```java
    @Controller
    @RequestMapping("/basic/items")
    @RequiredArgsConstructor
    public class BasicItemController {
    
        private final ItemRepository itemRepository;
    
        @GetMapping
        public String items(Model model){
            List<Item> items = itemRepository.findAll();
            model.addAttribute("items",items);
    
            return "basic/items";
        }
    
        /**
         * 테스트용 데이터 추가
         */
        @PostConstruct
        public void init(){
            itemRepository.save(new Item("itemA",10000,10));
            itemRepository.save(new Item("itemB",20000,20));
        }
    
    
    }
    ```
    

### items.html (Thymeleaf 사용)

- 뷰 템플릿은 정적화면이 아닌 ==**동적화면**==이기 때문에 resources/static 이 아닌 ==**resources/templates 경로에 넣어주면 된다.**==
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="utf-8">
        <link th:href="@{/css/bootstrap.min.css}"
                href="../css/bootstrap.min.css" rel="stylesheet">
    </head>
    <body>
    <div class="container" style="max-width: 600px">
        <div class="py-5 text-center">
            <h2>상품 목록</h2></div>
        <div class="row">
            <div class="col">
                <button class="btn btn-primary float-end"
                        onclick="location.href='addForm.html'"
                        th:onclick="|location.href='@{/basic/items/add}'|"
                        type="button">상품 등록
                </button>
            </div>
        </div>
        <hr class="my-4">
        <div>
            <table class="table">
                <thead>
                <tr>
                    <th>ID</th>
                    <th>상품명</th>
                    <th>가격</th>
                    <th>수량</th>
                </tr>
                </thead>
                <tbody>
                <tr th:each="item : ${items}">
                    <td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.id}">회원 ID</a></td>
                    <td><a href="item.html" th:href="@{|/basic/items/${item.id}|}" th:text="${item.itemName}">상품명</a></td>
                    <td th:text="${item.price}">가격</td>
                    <td th:text="${item.quantity}">수량</td>
                </tr>
    
                </tbody>
            </table>
        </div>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    

# 상품 상세

---

### BasicController

- Model을 이용하여 데이터를 HTML로 뿌려준다.
    
    ```java
    // ~~생략
    @GetMapping("/{itemId}")
    public String item(@PathVariable Long itemId,Model model){
        Item item = itemRepository.findbyId(itemId);
        model.addAttribute("item",item);
    
        return "/basic/item";
    }
    ```
    

### item.html (Thymeleaf)

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        } </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 상세</h2></div>
    <div>
        <label for="itemId">상품 ID</label>
        <input type="text" id="itemId" name="itemId" class="form-control" value="1" th:value="${item.id}" readonly>
    </div>
    <div>
        <label for="itemName">상품명</label>
        <input type="text" id="itemName" name="itemName" class="form-control" value="상품A" th:value="${item.itemName}" readonly></div>
    <div>
        <label for="price">가격</label>
        <input type="text" id="price" name="price" class="form-control" value="10000" th:value="${item.price}" readonly>
    </div>
    <div>
        <label for="quantity">수량</label>
        <input type="text" id="quantity" name="quantity" class="form-control" value="10" th:value="${item.quantity}" readonly>
    </div>
    <hr class="my-4">
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-primary btn-lg"
                    onclick="location.href='editForm.html'"
                    th:onclick="|location.href='@{/basic/items/{itemId}/edit(itemId=${item.id})}'|"
                    type="button">상품 수정
            </button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg"
                    onclick="location.href='items.html'"
                    th:onclick="|location.href='@{/basic/items}'|"
                    type="button">목록으로
            </button>
        </div>
    </div>
</div> <!-- /container -->
</body>
</html>
```

# 상품 등록

---

### BasicItemController

- 우선 url 이 /basic/items/add 로 오면 Get방식으로 addForm.html 뷰를 보여준다.
- 그리고 뷰에있는 Form 에 데이터를 넣고 전송버튼을 누르면 Post방식으로 똑같은 url로 오게한다.
    
    ```java
    // ...생략
    @GetMapping("/add")
    public String addForm(){
        return "basic/addForm";
    }
    
    @PostMapping("/add")
    public String save(){
        //로직
    }
    ```
    
- 상품 등록을 POST로 처리할때는 2가지가 있다.
- 첫번째는 @RequestParam으로 파라미터 하나하나를 받아 setter를 이용해 객체를 만드는것.
- 두번째는 @ModelAttribute 어노테이션을 사용하는것이다.
    
    ```java
    //@PostMapping("/add")
        public String addItemV1(@RequestParam String itemName,
                           @RequestParam int price,
                           @RequestParam Integer quantity,
                           Model model){
            Item item = new Item();
            item.setItemName(itemName);
            item.setPrice(price);
            item.setQuantity(quantity);
    
            itemRepository.save(item);
    
            model.addAttribute("item", item);
    
            return "basic/item";
        }
    
        //@PostMapping("/add")
        public String addItemV2(@ModelAttribute("item") Item item, Model model){
            /**
             * @ModelAttribute 어노테이션이 아래와 같은 로직을 실행시켜줌
             * model.addAttribute() 메서드도 같이 해준다.
                 Item item = new Item();
                 item.setItemName(itemName);
                 item.setPrice(price);
                 item.setQuantity(quantity);
    
                 model.addAttribute("item", item);
             */
    
            itemRepository.save(item);
    
            return "basic/item";
        }
    
        //@PostMapping("/add")
        public String addItemV3(@ModelAttribute Item item){
            /**
             * @ModelAttribute 어노테이션은 key 값이 없으면 클래스명의 앞글자를 소문자로 바꿔서 매핑한다
             * Item -> item 이 model.addAttribute('item', item) 이렇게 담긴다.
             */
            itemRepository.save(item);
    
            return "basic/item";
        }
    
        @PostMapping("/add")
        public String addItemV4(Item item){
            /**
             * @ModelAttribute 은 생략가능하다.
             * String, int 등등이 아닌 게 파라미터로 오면 알아서 @ModelAttribute 로 인식해준다.
             */
            itemRepository.save(item);
    
            return "basic/item";
        }
    ```
    
- 똑같은 url로 올때는 Thymeleaf에서
    
    `th:action="/basic/items/add"` → `th:action` 로 표현할 수 있기 때문에 간결해진다.
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="utf-8">
        <link th:href="@{/css/bootstrap.min.css}"
              href="../css/bootstrap.min.css" rel="stylesheet">
        <style>
            .container {
                max-width: 560px;
            } </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5 text-center">
            <h2>상품 등록 폼</h2>
        </div>
        <h4 class="mb-3">상품 입력</h4>
    
        <form action="item.html" th:action method="post">
            <div>
                <label for="itemName">상품명</label>
                <input type="text" id="itemName" name="itemName" class="form-control" placeholder="이름을 입력하세요"></div>
            <div>
                <label for="price">가격</label>
                <input type="text" id="price" name="price" class="form-control" placeholder="가격을 입력하세요">
            </div>
            <div>
                <label for="quantity">수량</label>
                <input type="text" id="quantity" name="quantity" class="form-control" placeholder="수량을 입력하세요"></div>
            <hr class="my-4">
            <div class="row">
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg" type="submit">상품 등록
                    </button>
                </div>
                <div class="col">
                    <button class="w-100 btn btn-secondary btn-lg"
                            onclick="location.href='items.html'"
                            th:onclick="|location.href='@{/basic/items}'|"
                            type="button">취소
                    </button>
                </div>
            </div>
        </form>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    
      
    

# 상품 수정

### BasicItemController

- 이번에도 상품수정 View를 보여주는 부분과 직접 수정 로직을 실행하는 부분의 url을 통일 시켰다.
    - ==**HTTP Form 전송이라 수정인데도 PUT, PATCH를 사용하지 않고 POST를 사용했다.**==

```java
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model){
    Item item = itemRepository.findbyId(itemId);
    model.addAttribute("item",item);
    return "basic/editForm";
}

@PostMapping("/{itemId}/edit")
public String edit(@PathVariable Long itemId,@ModelAttribute Item item){
    itemRepository.update(itemId,item);

    return "redirect:/basic/items/{itemId}";
}
```

- HTML Form 전송방식에는 PUT, PATCH 메서드를 사용할 수 없고 POST를 사용해야한다.
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="utf-8">
        <link th:href="@{/css/bootstrap.min.css}"
              href="../css/bootstrap.min.css" rel="stylesheet">
        <style>
            .container {
                max-width: 560px;
            } </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5 text-center">
            <h2>상품 수정 폼</h2></div>
        <form action="item.html" th:action method="post">
            <div>
                <label for="id">상품 ID</label>
                <input type="text" id="id" name="id" class="form-control" th:value="${item.id}" value="1" readonly>
            </div>
            <div>
                <label for="itemName">상품명</label>
                <input type="text" id="itemName" name="itemName" class="form-control" th:value="${item.itemName}" value="상품A">
            </div>
            <div>
                <label for="price">가격</label>
                <input type="text" id="price" name="price" class="form-control" th:value="${item.price}" value="10000">
            </div>
            <div>
                <label for="quantity">수량</label>
                <input type="text" id="quantity" name="quantity" class="form-control" th:value="${item.quantity}" value="10">
            </div>
            <hr class="my-4">
            <div class="row">
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg" type="submit">저장</button>
                </div>
                <div class="col">
                    <button class="w-100 btn btn-secondary btn-lg"
                            onclick="location.href='item.html'"
                            th:onclick="|location.href='@{/basic/items/{itemId}(itemId=${item.id})}'|"
                            type="button">취소</button>
                </div>
            </div>
        </form>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    

# PRG : POST/Redirect/Get

- 상품 등록 컨트롤러에서 큰 문제가 있다.
- 새로운 상품을 Repository에 저장하고 return 값으로 basic/item URL로 가는데
- 이렇게 되면 POST방식으로 basic/item 화면을 띄워준다.
- 만약 사용자가 그 화면에서 새로고침을 하면, ==**서버는 마지막에 전송한 데이터를 재 전송하게 된다.**==
    
    ```java
    //@PostMapping("/add")
    public String addItemV4(Item item){
      
        itemRepository.save(item);
    
        return "basic/item";
    }
    ```
    
- 그 때 사용하는것이 Redirect 이다.
- 스프링에서는 리다이렉트를 간편하게 사용할 수 있다.
    
    ```java
    @PostMapping("/add")
    public String addItemV5(Item item){
        itemRepository.save(item);
        return "redirect:/basic/items/"+item.getId();
    }
    ```
    
    ![[Untitled 40.png|Untitled 40.png]]
    
- 하지만 redirect 에서 `item.getId()`같은 변수가 URL로 들어가면 URL 인코디 오류가 생긴다. 이것을 해결하려면 `RedirectAttributes` 를 쓰면 된다.

## **RedirectAttributes**

==**RedirectAttributes 클래스는 URL 인코딩도 도와주고 PathVariable 쿼리 파라미터까지 처리해준다.**==

- URL 인코딩을 위해 itemId를 추가해주고 status는 나중에 뷰템플릿 메시지를 추가하기 위해 쓰인다.
- itemId는 PathVariable로 쓰이고 status는 쿼리 파라미터로 쓰인다.
    
    ```java
    @PostMapping("/add")
    public String addItemV6(Item item, RedirectAttributes redirectAttributes){
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId",savedItem.getId());
        redirectAttributes.addAttribute("status", true);
    
        return "redirect:/basic/items/{itemId}";
    }
    ```
    
    ![[Untitled 1 29.png|Untitled 1 29.png]]
    
- 저장완료는 Thymeleaf의 if문을 이용하여 쓴다.
    
    ```html
    <!--  상품 저장이 잘됐으면 잘됐다는 메시지 추가 -->
    <h2 th:if="${param.status}" th:text="'저장 완료'"></h2>
    ```