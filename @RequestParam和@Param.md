@RequestParam 用于controller层
（1）解决前台参数名称与后台接收参数变量名称不一致的问题，等价于request.getParam
（2）可设置value：指定参数名 default：指定变量初始值 require（true默认/false）:指定参数是否为必传

@Param 用于dao层
个人理解为修饰参数，使得mapper.xml中的参数与后台的参数对应上，也增强了可读性
如果两者参数名一致得话，spring会自动进行封装，不一致的时候就需要手动去使其对应上。

dao层接口类方法中也可以使用@RequestParam和@Param，我自己认为使用@Param居多，使用@Param需要加上（"param参数名"）。一般在xxControl类方法中尽量使用@RequestParam/@RequestBody

xxControl类方法中可以使用@RequestParam

    @RequestMapping(value = "/test", method = RequestMethod.POST)
    public ResponseObj<Boolean> test(@RequestParam String string) {

        System.out.println("哈哈哈");
        return new ResponseObj<>(true, null);

    }
dao层使用@Param

@Mapper
public interface MenuMapper extends BaseMapper<Menu> {
    /**
     * 根据角色Id列表查询其菜单列表
     *
     * @param roleIds 角色Id列表
     * @return 菜单列表
     */
    List<Menu> selectMenuByRoleIds(@Param("roleIds") List<String> roleIds, @Param("syscode") int syscode);

}
postman请求如下
image.png

作者：墨色尘埃
链接：https://www.jianshu.com/p/2dbd06f23e3c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
