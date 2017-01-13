# spring-mvc

## spring-mvc test faq

*  Q1: java.lang.NoSuchMethodError: javax.servlet.http.HttpServletRequest.isAsyncStarted()
  
   Background: 
   
>    Spring version: 4.3.2.RELEASE

>    servlet-api:2.5

>    spring-mock:2.0.8

>    mockito-all:1.9.5

>    hamcrest-core:1.3

>    junit:4.12
   
   Solution: 
   
    upgrade the servlet-api version less than 3.0, problem fixed

* Q2:java.lang.AssertionError: Status expected:<200> but was:<404> 

  Background:
  
>    Spring version: 4.3.2.RELEASE

>    servlet-api:3.0

>    spring-mock:2.0.8

>    mockito-all:1.9.5

>    hamcrest-core:1.3

>    junit:4.12

```java
  //This is my controller
  @Controller
  public class HomeController{
     @RequestMapping("index")
     public String index(){ return "index"}
  } 
 
 //This is my test case
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration   //调用javaWEB的组件，比如自动注入ServletContext Bean等等
@ContextConfiguration( locations = {"classpath*:spring/spring-config.xml","classpath*:spring/spring-context.xml"})
public class BaseTestCase {

   @Autowired
    private WebApplicationContext wac;
    private MockMvc mockMvc;



    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void indextest() throws Exception {
               this.mockMvc.perform(post("index"))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(status().isOk())
                .andReturn().getResponse().getContentAsString()
        ;
    }
}

```

After running the test ,i got 404. When I debugged the springmvc test source code, I got no found uri .

Soulution: 
   add / before the uri, the source code below

```java
 @Test
    public void indextest() throws Exception {
               this.mockMvc.perform(post("/index"))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(status().isOk())
                .andReturn().getResponse().getContentAsString()
        ;
    }
```

* Q3:java.lang.AssertionError: Status expected:<200> but was:<400> 

Backgound: as same as Q2

```java
  //this is my query class
  public class Query{
   private int id;
   private int age;
   //...setter and getter
  }
  
  //This is my controller
  @Controller
  public class HomeController{
     @RequestMapping("index")
     public String index(@RequestBody Query query){ return "index"}
  } 
 
 //This is my test case
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration   //调用javaWEB的组件，比如自动注入ServletContext Bean等等
@ContextConfiguration( locations = {"classpath*:spring/spring-config.xml","classpath*:spring/spring-context.xml"})
public class BaseTestCase {

   @Autowired
    private WebApplicationContext wac;
    private MockMvc mockMvc;



    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void indextest() throws Exception {
               this.mockMvc.perform(post("/index").
                 content(MediaType.APPLICATION_JSON).content("{id:1,age:12}") )
                .andDo(MockMvcResultHandlers.print())
                .andExpect(status().isOk())
                .andReturn().getResponse().getContentAsString()
        ;
    }
}

```

Soulution:
   problem is that input param is not match @ReponseBody param

```java
@Test
    public void indextest() throws Exception {
        Query query = new Query();
        query.setId(1);
        query.setAge(2);
               this.mockMvc.perform(post("/index").
                 content(MediaType.APPLICATION_JSON).content(JSONObject.parse(query))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(status().isOk())
                .andReturn().getResponse().getContentAsString()
        ;
    }
```
