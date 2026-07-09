  

```java
@AutoConfigureMockMvc
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
public class UserControllerTest extends DummyObject {

    @Autowired
    private MockMvc mvc;
    @Autowired
    private ObjectMapper om;
    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    public void setUp(){
        dataSetting();
    }


    @Test
    @DisplayName("회원가입 성공 테스트")
    public void join_success_test() throws Exception {
        //given
        JoinReqDto joinReqDto = new JoinReqDto();
        joinReqDto.setUsername("love");
        joinReqDto.setPassword("1234");
        joinReqDto.setEmail("love@nate.com");
        joinReqDto.setFullName("러브");

        String requestBody = om.writeValueAsString(joinReqDto);

        //when
        ResultActions resultActions = mvc.perform(post("/api/join").
                content(requestBody).
                contentType(MediaType.APPLICATION_JSON));

        String responseBody = resultActions.andReturn().getResponse().getContentAsString();

        //then
        resultActions.andExpect(status().isCreated());
    }

    @Test
    @DisplayName("회원가입 실패 테스트")
    public void join_fail_test() throws Exception {
        //given
        JoinReqDto joinReqDto = new JoinReqDto();
        joinReqDto.setUsername("ssar");
        joinReqDto.setPassword("1234");
        joinReqDto.setEmail("ssar@nate.com");
        joinReqDto.setFullName("쌀");

        String requestBody = om.writeValueAsString(joinReqDto);

        //when
        ResultActions resultActions = mvc.perform(post("/api/join").
                content(requestBody).
                contentType(MediaType.APPLICATION_JSON));

        String responseBody = resultActions.andReturn().getResponse().getContentAsString();

        //then
        resultActions.andExpect(status().isBadRequest());
    }


    private void dataSetting() {
        userRepository.save(newUser("ssar","쌀"));
    }
}
```