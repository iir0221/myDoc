# Spring MVC Controller

## Spring MVC Test ”Circular view path” 异常
```java
@Controller
@RequestMapping("/spittles")
public class SpittleController {

    private static final String MAX_LONG_AS_STRING = "9223372036854775807";

    private SpittleRepository spittleRepository;

    @Autowired
    public SpittleController(SpittleRepository spittleRepository) {
        this.spittleRepository = spittleRepository;
    }

    @RequestMapping(method=RequestMethod.GET)
    public List<Spittle> spittles(
        @RequestParam(value="max", defaultValue=MAX_LONG_AS_STRING) long max,
        @RequestParam(value="count", defaultValue="20") int count) {
    return spittleRepository.findSpittles(max, count);
    }

    @RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
    public String spittle(
        @PathVariable("spittleId") long spittleId, 
        Model model) {
    model.addAttribute(spittleRepository.findOne(spittleId));
    return "spittle";
    }

    @RequestMapping(method=RequestMethod.POST)
    public String saveSpittle(SpittleForm form, Model model) throws Exception {
    spittleRepository.save(new Spittle(null, form.getMessage(), new Date(), 
        form.getLongitude(), form.getLatitude()));
    return "redirect:/spittles";
    }

}
```
```java
@Test
public void shouldShowRecentSpittles() throws Exception {
    List<Spittle> expectedSpittles = createSpittleList(20);
    // SpittleRepository接口的mock实现
    SpittleRepository mockRepository = mock(SpittleRepository.class);
    // 该实现的findSpittles()方法会返回expectedSpittles
    when(mockRepository.findSpittles(Long.MAX_VALUE, 20))
        .thenReturn(expectedSpittles);

    // Mock spring mvc
    SpittleController controller = new SpittleController(mockRepository);
    MockMvc mockMvc = standaloneSetup(controller).build();

    // 发起GET请求/spittles
    mockMvc.perform(get("/spittles"))
            // 期望如下
        .andExpect(view().name("spittles"))
        .andExpect(model().attributeExists("spittleList"))
        .andExpect(model().attribute("spittleList", 
                    hasItems(expectedSpittles.toArray())));
}
```

这个test 会产生Circular view path异常
* 原因

    Mock并不使用WebMvcConfigurerAdapter接口的实现类中所声明的ViewResolver。
如果不在Test中显示声明一个ViewResolver，Spring默认注册一个JstlView作为ViewResolver。
mockMvc.perform(get("/spittles"))发起一个get请求，映射到Controller对应的方法，该方法返回的view名称为spittles，这个spittles又被映射到Controller，这就造成了一个循环。
* 解决

    * 在Test中声明一个ViewResolver，例如（resolver.setPrefix("/WEB-INF/views/");resolver.setSuffix(".jsp");这时Controller对应的方法法返回的spittles，对映射到/WEB-INF/views/spittles.jsp

    * 修改view和path，让他们不同名

[How to avoid the “Circular view path” exception with Spring MVC test](https://stackoverflow.com/questions/18813615/how-to-avoid-the-circular-view-path-exception-with-spring-mvc-test)

[如何在Spring MVC Test中避免”Circular view path” 异常](http://www.cnblogs.com/chry/p/6240965.html)