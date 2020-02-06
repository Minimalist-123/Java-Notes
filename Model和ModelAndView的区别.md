Model只是用来传输数据的，并不会进行业务的寻址。

ModelAndView却是可以进行业务寻址的，就是设置对应的要请求的静态文件，这里的静态文件指的是类似jsp的文件。

两者还有一个最大的区别，那就是Model是每一次请求都必须会带着的，但是ModelAndView是需要我们自己去新建的


[java] view plain copy
public String index(@RequestParam(value = "menuId", required = false) Long menuId, Model model, SysMenuPojo sysMenuPojo) {  
    if(null != menuId) {  
        sysMenuPojo = sysMenuService.findById(menuId);  
    }  
        model.addAttribute("sysMenuPojo",sysMenuPojo);  
        return "sysMenu/index";  
  
    }  

[java] view plain copy
@RequestMapping("/index")  
//    @RequiresPermissions("sysMenu:index")  
    public ModelAndView index(@RequestParam(value = "menuId", required = false) Long menuId, Model model, SysMenuPojo sysMenuPojo) {  
    if(null != menuId) {  
        sysMenuPojo = sysMenuService.findById(menuId);  
    }  
        ModelAndView m=new ModelAndView();  
        m.addObject("sysMenuPojo",sysMenuPojo);  
        m.setViewName("/sysMenu/index");  
        m.addObject("sysMenuPojo",sysMenuPojo);  
        return m;  
  
    }  




   首先是Model传递数据。

@Controller
public class FreemarkerController {
    @SuppressWarnings("unchecked")
    @RequestMapping(method = RequestMethod.POST, value = "/freemarker")
    public String getFtl(Model model) {
        // 构造填充数据的Map
        Map map = new HashMap();
        List<TestVo> testVos = new ArrayList<>();
        TestVo testVo = new TestVo();
        testVo.setName("fulei");
        TestVo testVo1 = new TestVo();
        testVo1.setName("wangmeng");
        testVos.add(testVo);
        testVos.add(testVo1);
        map.put("user", "love");
        map.put("url", "http://www.baidu.com/");
        map.put("name", "百度");
        map.put("testVos", testVos);
        model.addAllAttributes(map);
        return "test";
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
        其次就是ModelAndView。

    @RequestMapping(method = RequestMethod.POST, value = "/freemarker")
    public ModelAndView getFtlByModelAndView() {
        ModelAndView modelAndView = new ModelAndView();
        // 构造填充数据的Map
        Map map = new HashMap();
        List<TestVo> testVos = new ArrayList<>();
        TestVo testVo = new TestVo();
        testVo.setName("fulei");
        TestVo testVo1 = new TestVo();
        testVo1.setName("wangmeng");
        testVos.add(testVo);
        testVos.add(testVo1);
        map.put("user", "love");
        map.put("url", "http://www.baidu.com/");
        map.put("name", "百度");
        map.put("testVos", testVos);
        modelAndView.addAllObjects(map);
        return modelAndView;
    }
    
    
    【原文链接】：https://blog.csdn.net/opera95/article/details/78498812
