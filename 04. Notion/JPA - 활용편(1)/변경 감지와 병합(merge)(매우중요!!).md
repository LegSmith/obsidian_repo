### 준영속 엔티티?

영속성 컨텍스트가 더는 관리하지 않는 엔티티.

- 아래 코드에서는 book객체가 준영속 엔티티이다.
- Book객체는 이미 DB에 한번 저장되어서 식별자가 존재한다. **==개발자가 임의로 만들어낸 엔티티도 기존 식별자를 가지고 있으면 준영속 엔티티로 볼 수 있다.==**

    ```java
    @PostMapping("/items/{itemId}/edit")
        public String updateItem(@ModelAttribute("form") BookForm form){

            Book book = new Book();
            book.setId(form.getId());
            book.setName(form.getName());
            book.setPrice(form.getPrice());
            book.setStockQuantity(form.getStockQuantity());
            book.setAuthor(form.getAuthor());
            book.setIsbn(form.getIsbn());

            itemService.saveItem(book);
            return "redirect:/items";
        }
    ```


이런 준영속 엔티티는 변경감지가 일어나지 않기 때문에 book을 변경하여도 DB에 업데이트가 날라가지 않는다;;

---

# 준영속 엔티티를 수정하는 방법

1. 변경 감지 기능 사용  
2. 병합(`merge`) 사용

### 변경 감지 기능 사용

- **==영속성 컨텍스트에서 엔티티를 다시 조회하여 영속상태인 객체를 수정한다==**
- @Transactional 어노테이션에 의해서 해당 로직이 끝나면 commit이 실행되고 트랜잭션 커밋 시점에서 변경감지(Dirty Checking)이 동작해서 update sql 실행

```java
@Transactional
    public void updateItem(Long itemId, Book param){
        Item findItem = itemRepository.findOne(itemId); //영속상태이다.

        findItem.setName(param.getName());
        findItem.setPrice(param.getPrice());
        findItem.setStockQuantity(param.getStockQuantity());
    }
```

### 병합 사용

병합(`merge`)은 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용하는 기능이다.

- 병합(merge) 메커니즘
    1. merge() 함수 실행
    2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차캐시 조회( 없으면 DB에서 조회하여 저장)
    3. **==조회한 영속 엔티티의 값을 전부 준영속 엔티티 값으로 채워넣는다.==**
    4. 영속 상태인 엔티티를 반환한다.

병합(merge)의 문제점==**병합 실행시 모든 속성이 변경된다**.== 수정하는 속성중 값이 없으면 null로 교최(즉, 모든 필드를 교체한다)

> 실무에서는 필드가 많으므로 병합보다는 변경감지 기능이 좋다.

- 아래와 같이 파라미터나 dto로 받아서 service계층에서 바꾸는게 낫다.

    ```java
    @Transactional
        public void updateItem(Long itemId, String name, int price, int stockQuantity){
            Item findItem = itemRepository.findOne(itemId); //영속상태이다.

            findItem.setName(name);
            findItem.setPrice(price);
            findItem.setStockQuantity(stockQuantity);
        }
    ```


# 가장 좋은 방법

---

==**엔티티를 변경할 때는 항상 변경감지(Dirty Checking)을 사용하자 !!!**==

- 컨트롤러 계층에서 어설프게 엔티티를 생성하지 말자.
    - Book객체를 생성해서 setter를 이용하여 값을 넣는것 보다는 변경할 데이터를 명확하게 전달(파라미터 or DTO)하는게 훨씬 깔끔하고 추척하기 좋다!!

        ```java
        @PostMapping("items/{itemId}/edit")
            public String updateItem( @PathVariable Long itemId, @ModelAttribute("form") BookForm form){

        //        Book book = new Book();
        //
        //        book.setId(form.getId());
        //        book.setName(form.getName());
        //        book.setPrice(form.getPrice());
        //        book.setStockQuantity(form.getStockQuantity());
        //        book.setAuthor(form.getAuthor());
        //        book.setIsbn(form.getIsbn());
        //
        //        itemService.saveItem(book);

                itemService.updateItem(itemId,form.getName(),form.getPrice(), form.getStockQuantity());

                return "redirect:/items";

            }
        ```

- 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경하자 !!
    - 이려면 commit시점에 더티 체킹이되어 update 쿼리가 나간다!!

        ```java
        @Transactional
            public void updateItem(Long itemId, String name, int price, int stockQuantity) {
                Item findITem = itemRepository.findOne(itemId); //영속상태의 엔티티 조회

                findITem.setName(name);
                findITem.setPrice(price);
                findITem.setStockQuantity(stockQuantity);

            }
        ```

## 관련 문서

- 상위 목차: [[JPA - 활용편(1)]]
- 이전 문서: [[도메인 분석 설계]]
- 다음 문서: [[웹 계층 개발]]
