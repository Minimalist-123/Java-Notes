@Param注解的用法解析
一.xml形式
实例一 @Param注解单一属性

dao层示例

Public User selectUser(@param(“userName”) String name, @param(“userpassword”) String password);
xml映射对应示例

<select id=" selectUser" resultMap="BaseResultMap">  
    select  *  from user_user_t 
        where user_name = #{userName，jdbcType=VARCHAR} and user_password=#{userPassword,jdbcType=VARCHAR}  
</select>
注意：采用#{}的方式把@Param注解括号内的参数进行引用（括号内参数对应的是形参如 userName对应的是name）；

实例二 @Param注解JavaBean对象

dao层示例

public List<user> getUserInformation(@Param("user") User user);
xml映射对应示例

<select id="getUserInformation" parameterType="com.github.demo.vo.User" resultMap="userMapper">  
    select   
        <include refid="User_Base_Column_List"/>  
    from mo_user t where 1=1  
        <!-- 因为传进来的是对象所以这样写是取不到值得 -->  
        <if test="user.userName!=null and user.userName!=''"> and t.user_name = #{user.userName} </if>  
        <if test="user.userAge!=null and user.userAge!=''"> and t.user_age = #{user.userAge} </if>  
</select>  
二.注解形式
1，使用@Param注解

当以下面的方式进行写SQL语句时：

    @Select("select column from table where userid = #{userid} ")
    public int selectColumn(int userid);
当你使用了使用@Param注解来声明参数时，如果使用 #{} 或 ${} 的方式都可以。

    @Select("select column from table where userid = ${userid} ")
    public int selectColumn(@Param("userid") int userid);
当你不使用@Param注解来声明参数时，必须使用使用 #{}方式。如果使用 的方式，会报错。∗∗或者在mybatis中mapper文件中像这样写{_parameter}，你只需要传入一条String类型的字符串。sql语句他就可以直接执行了，所以可以在动态配置的时候使用到该参数的含义：当只有一个参数，可以使用_parameter，它就代表了这个参数，如果使用@Param的话，会使用指定的参数值代替**

    @Select("select column from table where userid = ${userid} ")
    public int selectColumn(@Param("userid") int userid);
2，不使用@Param注解

不使用@Param注解时，参数只能有一个，并且是Javabean。在SQL语句里可以引用JavaBean的属性，而且只能引用JavaBean的属性。

    // 这里id是user的属性
    @Select("SELECT * from Table where id = ${id}")
    Enchashment selectUserById(User user);
    
    
    
    原文链接：https://www.cnblogs.com/zhuhui-site/p/10088369.html
