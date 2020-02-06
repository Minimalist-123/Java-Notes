Swagger使用
1、描述
Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。

作用：

1.接口的文档在线自动生成。

2.功能测试。

2、运用
a) maven导入Swagger

<dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>2.6.1</version>
</dependency>
<dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger-ui</artifactId>
     <version>2.6.1</version>
</dependency>
b) 创建Swagger2配置类

/**
 * @program: jpademo
 * @description: Swagger
 * @author: ZengGuangfu
 * @create 2018-10-24 10:12
 */
​
@Configuration
@EnableSwagger2
public class Swagger {
​
   @Bean
   public Docket docket(){
       return new Docket(DocumentationType.SWAGGER_2)
         .apiInfo(apiInfo())
         .select()
         .apis(RequestHandlerSelectors.basePackage("com.example.springbootjpa.jpademo.controller"))
         .paths(PathSelectors.any())
         .build();
   }
​
   public ApiInfo apiInfo(){
       return new ApiInfoBuilder()
         .title("利用swagger2构建的API文档")
         .description("用restful风格写接口")
         .termsOfServiceUrl("")
         .version("1.0")
         .build();
   }
}
如上所示，docket() 方法创建Docket的Bean对象，apiInfo()则是创建ApiInfo的基本信息。

链式方法解析：

3、注解及其说明
@Api : 用在类上，说明该类的主要作用。

@ApiOperation：用在方法上，给API增加方法说明。

@ApiImplicitParams : 用在方法上，包含一组参数说明。

@ApiImplicitParam：用来注解来给方法入参增加说明。

apiParam.png
@ApiResponses：用于表示一组响应。

@ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息

​ l code：数字，例如400

​ l message：信息，例如"请求参数没填好"

​ l response：抛出异常的类

@ApiModel：用在返回对象类上，描述一个Model的信息（一般用在请求参数无法使用@ApiImplicitParam注解进行描述的时候）

​ l @ApiModelProperty：描述一个model的属性

以下仅仅是一个例子，其实我个人在开发中很少使用@ApiImplicitParam 作为参数的描述，这样描述在参数过多的条件下会有点麻烦。个人一般是将参数封装为一个完整对象（特别是GET方法），并利用@ApiModel注解去定义参数，如果不需要作为查询条件的，则加一个hidden = true，如果是必填属性，则增加一个required = true即可。

例子：


/**
 * @program: jpademo
 * @description: EmployeeController
 * @author: ZengGuangfu
 * @create 2018-10-23 11:07
 */
​
@RestController
@RequestMapping("emp")
@Api(value = "用户管理类")
public class EmployeeController {
​
 @Autowired
 private EmployeeReposiroty employeeReposiroty;
​
      /**
      * 增加人物
      * @param employee
      * @return
      */
     @PostMapping(value = "employee")
     @ApiOperation(value = "新增一个用户",notes = "新增之后返回对象")
     @ApiImplicitParam(paramType = "query",name = "employee",value = "用户",required = true)
     @ApiResponse(code = 400,message = "参数没有填好",response = String.class)
     public String insert(Employee employee){
         Employee employee1 = employeeReposiroty.save(employee);
         if(employee1 != null) {
             return SysNode.Judge.SUCCESS.getResult();
         }else {
             return SysNode.Judge.FAILD.getResult();
         }
     }
​
      /**
      * 删除单个用户
      * @param id
      * @return
      */
      @DeleteMapping(value = "employee/{id}")
      @ApiOperation(value = "删除用户",notes = "根据成员id删除单个用户")
      @ApiImplicitParam(paramType = "path",name = "id",value = "用户id",required = true,dataType = "Integer")
      @ApiResponse(code = 400,message = "参数没有填好",response = String.class)
      public String delete(@PathVariable("id")Integer id){
           try{
                employeeReposiroty.deleteById(id);
                return SysNode.Judge.SUCCESS.getResult();
           }catch (Exception e){
                e.printStackTrace();
               return SysNode.Judge.FAILD.getResult();
           }
      }
​
      /**
      * 修改单个成员
      * @param employee
      * @return
      */
      @PutMapping(value = "employee/{id}")
      @ApiOperation(value = "修改用户信息",notes = "根据成员id修改单个用户")
      public String update(Employee employee){
           /**
           * save方法如果参数属性缺失，会导致原本存在的数据为null
           */
           Employee employee1 = employeeReposiroty.saveAndFlush(employee);
           if (employee1 != null) {
                return SysNode.Judge.SUCCESS.getResult();
           }else {
               return SysNode.Judge.FAILD.getResult();
           }
      }
​
      /**
      * 获取所有成员,升序排列
      * @return
      */
      @GetMapping(value = "employee/sort")
      @ApiOperation(value = "查询全部用户",notes = "默认根据升序查询全部用户信息")
      public List<Employee> findAll(){
           Sort orders = new Sort(Sort.Direction.DESC,"employeeId");
           List<Employee> employeeList = employeeReposiroty.findAll(orders);
           return employeeList;
      }
​
      /**
     * 获取所有成员,升序排列
     * @return
      */
      @GetMapping(value = "employee/pageSort")
      @ApiOperation(value = "查询用户信息",notes = "查询用户信息")
      @ApiImplicitParams({
           @ApiImplicitParam(paramType = "query",name = "sort",value = "排序方式:asc|desc",dataType = "String",required = true),
           @ApiImplicitParam(paramType = "query",name = "pagenumber",value = "第几页",dataType = "Integer",required = true),
           @ApiImplicitParam(paramType = "query",name = "pageSize",value = "分页数",dataType = "Integer",required = true)
      })
      public List<Employee> findAllByPage(String sort,Integer pagenumber,Integer pageSize){
           try {
                Sort.Direction sortlast;
                if("desc".equals(sort.toLowerCase())){
                     sortlast = Sort.Direction.DESC;
               }else{          
                      sortlast = Sort.Direction.ASC;
               }
                     Sort orders = new Sort(sortlast, "employeeId");
                     Pageable pageable = new PageRequest(pagenumber, pageSize, orders);
​
                     Page<Employee> employeePage = employeeReposiroty.findAll(pageable);
                     List<Employee> employeeList = employeePage.getContent();
                     return employeeList;
           }catch (Exception e){
                e.printStackTrace();
                return null;
           }
      }
    /**
     * 自定义拓展jpa，根据用户名查找单个用户
     * @param username
     * @return
     */
     @GetMapping(value = "employee/find/{username}")
     @ApiOperation(value = "查询用户信息",notes = "根据用户登录名查询该用户信息")
     @ApiImplicitParam(paramType = "path",name = "username",value = "用户登录名",required = true,dataType = "String")
     public Employee findByUsername(@PathVariable("username") String username){
         List<Employee> employeeList = employeeReposiroty.findByUserNameOrderByEmployeeIdAsc(username);
         if (employeeList != null && !employeeList.isEmpty()){
             return employeeList.get(0);
         }
         return null;
     }
​
     /**
     * 测试用
     * @return
     */
     @GetMapping(value = "employee/grade")
     public List<Object[]> findEmployeeAndGrade(){
         Pageable pageable = new PageRequest(0,3);
​
         Page<Object[]> page = employeeReposiroty.findEmployeeAndGrade(pageable);
         System.out.println(page.getTotalElements()+"----------结果总数------------");
         System.out.println(page.getTotalPages()+"--------根据pageSize的总页数-----------");
         System.out.println(page.getNumber()+"--------当前页数，pageNumber----------");
         System.out.println(page.getNumberOfElements()+"--------当前页有几个数据--------");
         System.out.println(page.getSize()+"---------PageSize-------------");
         System.out.println(page.getSort()+"---------排序方式，没有则是'UNSORTED'----------");
​
         List<Object[]> objects = page.getContent();
         return objects;
    }
}
4、测试登录 localhost:8080/swagger-ui.html
swagger-ui.png
API 操作测试，修改


testParam.png
testResult.png
5、@ApiModel 接收对象传参
注意： 在后台采用对象接收参数时，Swagger自带的工具采用的是JSON传参， 测试时需要在参数上加入@RequestBody,正常运行采用form或URL提交时候请删除。

例子：


/**
 * @program: jpademo
 * @description: Employee
 * @author: ZengGuangfu
 * @create 2018-10-23 10:20
 */
​
@Data
@Entity
@Table(name = "employee")
@ApiModel(value = "用户对象模型")
public class Employee {
​
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   @Column(name = "employee_id")
   @Min(value = 1,groups = Employee.Children.class)
   private Integer employeeId;
​
   @Column(name = "user_name",length = 20,nullable = true)
   @ApiModelProperty(value = "userName",required = true)
   private String userName;
​
   @Column(nullable = true)
   @Size(min = 0,max = 65,message = "年龄超过范围限制",groups = Employee.Audit.class)
   @ApiModelProperty(value = "age",required = true)
   private Integer age;
​
   @Column(name="gra_id")
   @ApiModelProperty(value = "graId",required = true)
   //@Digits(integer = 12,fraction = 4)  //限制必须为一个小数，且整数部分的 位数 不能超过integer，小数部分的 位数 不能超过fraction
   private Integer graId;
​
   public interface Audit{};
​
   public interface Children{};
​
}



作者：茧铭
链接：https://www.jianshu.com/p/a0caf58b3653
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
