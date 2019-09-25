# api接口审查报告

代码review的功能可以通过idea插件`sonarLint`来审查，也可以为项目配置`sonarQube Server`,通过其规则和配置项进行检查。

通过对接口代码风格的规范性审查，有以下几点需要提出：

1. 接口入口路由，可以统一配置在类入口上,比如:
    ```
    @RestController
    @RequestMapping("/programs")
    public class ProgramController 
    ```
2. 接口增删改查可以用请求方法来体现，尽量使请求路由简洁，如：删除：DELETE，更新：PUT，查询：GET，新增：POST，比如
程序删除接口可以这样修改
    ```
    @DeleteMapping("/{pid}")
    @ApiOperation("删除")
    ```
3. URl中尽量只使用名词指定资源，不用动词，且推荐使用复数。
    ```
    http://192.168.16.30:10005/etlapi/programs/2 程序id为2的记录
    
    http://192.168.16.30:10005/etlapi/programs?pageNumber=2&offset=10 分页查询页码为2，当前页总数为10的记录列表
    ```
4. 比较复杂的接口不能确定是使用POST还是PUT时，要看具体的业务层代码，看看接口产生的结果是否幂等，如果幂等用PUT，相反用POST

    ```
      如：接口接收到一资源，资源存在更新，不存在插入新数据，这个接口就要用PUT
    ```

5. api的版本控制，应该将API的版本号放入URL，`https://192.168.16.30:10005/etlapi/v{n}/`,
另一种做法是，将版本号放在HTTP头信息中，但不如放入URL方便和直观.
