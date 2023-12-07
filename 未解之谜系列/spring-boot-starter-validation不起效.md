## spring-boot-starter-validation不起效

后端Spring boot有一个validation库，做参数校验的，就是在bean类里加一些注解做字段值的校验，比如这种：

```java
public class YwTempManagement implements Serializable {
    /**
     * 公告单位
     */
    @NotBlank(message = "公告单位不能为空")
    private String noticeUnit;
}
```

然后在使用的Controller中，给Controller类也加上@Validated注解：

```java
@Validated
@RestController
@RequestMapping("/yw/tempManagement")
public class YwTempManagementController extends BaseController {
```

在需要验证的地方再加上注解即可
```java
@PostMapping
public AjaxResult add(@Validated @RequestBody YwTempManagement ywTempManagement) {
    //检查参数
    String errMsg = checkAvailable(ywTempManagement);
    if (errMsg != null) {
        return AjaxResult.error(errMsg);
    }
    //其他业务代码...
}
```



问题是，这一套在我的电脑上不work，同样的代码，跑在别人电脑上就可以work，我的就不会校验这些参数，比如一个非空校验的值，如果调用的时候传了null，应该要报指定的错误才对，但是我电脑上起来的服务，只会报空指针异常。

搞了半天没搞定，网上也查不到其他有用的信息，放弃了~