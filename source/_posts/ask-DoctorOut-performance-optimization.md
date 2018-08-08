# Ask DoctorOut 接口优化



## 目的

1. 根据实战，分享一点性能调优的经验
   1. 一些对现状的讨论（比如，关于 XxxOut 的编码规范、客户端职责）



## 事件回顾

App 内有关医生列表的 UI 在第一次打开的时候，渲染比较慢。

以 `Top Menu` 的 `问医生` 为案例。

![Demo1](http://7xjzby.com1.z0.glb.clouddn.com/2018/08/07/51c56e07.jpg) | ![Demo2](http://7xjzby.com1.z0.glb.clouddn.com/2018/08/07/112bd3fb-d696-4eb3-afd8-03f46074c9ef.jpg)

Charles 调试，发现接口「/app/i/ask/sectiongroup/member」在第一次请求中，响应时间明显比较慢。大概在 1.1 s 左右。

![Charlse 慢请求](http://7xjzby.com1.z0.glb.clouddn.com/2018/08/07/charles_api_slow_record.png)



通过 Kibana 查询 API 访问记录，确认接口慢响应的情况确实存在。

#### 慢响应 (> 500 ms) 总数

![180731_Tues_slow_count](http://7xjzby.com1.z0.glb.clouddn.com/2018/08/07/180731_Tues_slow_count.png)

#### 平均响应时间

![180731_Tues_slow_average](http://7xjzby.com1.z0.glb.clouddn.com/2018/08/07/180731_Tues_slow_average.png)



通过观察统计数据，可以发现慢响应的问题确实存在。在用户访问高峰期，慢响应会明显增多。

接口的平均响应时间也并不理想，大概在 400ms 左右。



## 柯南来了!!



找到对应的代码，大概明白了，为什么第一次响应会比较慢。

```java
@CacheEnable(timeout = 300000)
```

在本地环境去掉缓存注解后，重新调试接口。接口平均响应时间在 1s +。

![Chrome API slow Debug](http://7xjzby.com1.z0.glb.clouddn.com/2018/08/07/chrome_api_slow_debug.png)

可以断定，接口内部实现中，存在导致慢响应的因素。



通过添加监控日志（通过 `StopWatch`），一点点定位到执行速度较慢的代码。

### Show The Code

```java
@RestController
@RequestMapping(value = "/view/i/sectiongroup/member")
public class SectionGroupMemberViewWebApi  {
  
  // ..
  
  @ApiOperation(value = "获取科室成员列表", notes = "支持条件筛选", response = UserOut.class)
  @RequestMapping(value = "", method = RequestMethod.GET)
  public JsonDataHolder list( // ... 参数略) {
    // ...
    
    // 30 - 150 ms， 偶尔 300 ms+
    String city = qqMapClient.getCityByLocation(location);
    
    // 1.2s => 50 ms
    selectGroupSupport.getDoctorOutByGroupBy(sectionGroupId, sectionId, city);
    
    // ...
  }
}

@Component
public class SectionGroupSupport {
  // ...
  
  public List<DoctorOut> getDoctorOutByGroupId(int sectionGroupId, int sectionId, String city) {
    // ... 
    // 发现 构造函数出现问题，我就有点方..  影响估计不止一个地方
    // 
    return doctorList.stream()
                .filter(d -> userMap.containsKey(d.getUserId()))
                .map(d -> new DoctorOut(d, userMap.get(d.getUserId()), newDoctorUserIdSet.contains(d.getUserId()), city))
                .collect(Collectors.toList());
  }
  
  // ...
}

public class DoctorOut implements Serializable {
  // ...
  
  public DoctorOut(Doctor doctor, User user) {
    // ...
    // 20 ms => < 1ms
	this.tagReferences = TagContentSupport.gerTagsReferences(doctor.getId(), doctor.getTags());
    // 30 ms => < 1ms
    this.tagNodes = TagContentSupport.getDoctorTagNodesByTagNames(doctor.getTags());
	// ...
    this.avgReplyTimeStr = joinAvgReplyTimeStr(doctor.getUserId());
	createTag(doctor);
  }
  
  private String joinAvgReplyTimeStr(int doctorId) {
    // 1 ms
    int avgReplyTime = CacheUtil.getDoctorAvgReplyTime(doctorId);
	// ...
  }
  
  private void createTag(Doctor doctor) {
	// 1 ms
	int avgReplyTime = CacheUtil.getDoctorAvgReplyTime(doctor.getUserId());
    // ...
  }
  
  // ...
}

@Component
public class TagContentSupport {
  // ...
  
  public static List<DiseaseReference> gerTagsReferences(int doctorId, String tag) {
    // ...
    // 从缓存中获取全量的疾病标签数据，测试环境大概 800 条数据
    // 15 ms
    List<DiseaseReference> diseaseTags = CacheUtil.getShowedDiseaseTags();
    // ...
  }
  
  public static List<DoctorTagNode> getDoctorTagNodesByTagNames(String tagNames) {
    // ...
    // 15 ms
    List<DiseaseReference> diseaseTags = CacheUtil.getShowedDiseaseTags();
    // ...
    // 1 ms
    Set<String> tagNameSet = tagExtractSupport.extract(tagNames);
    Map<String, String> param = OpenApiAuthSupport.generateParamWithSign();
    param.put("tag_name", StringUtils.join(tagNameSet, ","));

    // 10 ms
    String tagContentJson = HttpClientUtil.get(GET_TAG_CONTENT_LIST_URL, param);
	return JsonUtil.toList(TagSimpleContent.class, jsonArray.toString());
  }
 
  // ...
}

@Component
public class TagExtractSupport {
  // ...
  
  public static Set<String> extract(String content) {
    Set<String> tagSet = new HashSet<>();
    
    StringReader reader = new StringReader(content);
    IKSegmenter ik = new IKSegmenter(reader, true);  // 当为true时，分词器进行最大词长切分
    Lexeme lexeme;
    while ((lexeme = ik.next()) != null) {
      termSet.add(lexeme.getLexemeText());
      for (String term : termSet) {
        tagSet.add(term);
      }
    }
    
    return tagSet;
  }
  
  // ...
}
```



## 问题总结

在 DoctorOut 里，操作外部资源（HTTP 请求、Cache 服务器（非 Local Cache））、进行耗时的计算操作（分词）。

一个 DoctorOut 实例的生成，需要 1 个 10ms 的 HTTP 请求 +  2 个 1 ms 的 Cache 请求 +  2 个 15 ms 的 大 Cache 请求 + 1 个 2 ms 的分词计算 = 10 ms + 2 * 1 ms + 2 * 15 ms + 2 ms = 44 ms。

分页共 20 个实例，至少需要 44 ms * 20 = 880 ms。

假如, HTTP 资源不可用 或者 缓存服务器响应变慢，所有涉及到 DoctorOut 的业务均会受到影响。

假如，HTTP 请求没有控制（未使用连接池 / 请求超时未配置），直接导致 Tomcat 连接数被快速消耗完，进而导致 Tomcat 服务器无法响应任何请求。



## 解决方案

上面案例中，API 之所以慢，最主要的原因在于 标签处理的相关流程（为医生实例附加各种的标签）。

考虑到 `标签` 的特点：低频更新、高频使用、元数据。

本次优化，综合考虑 `变更的代码量尽可能少`、`开发的时间成本要短`、`优化后的潜在风险要尽量避免`。

需要在改造尽可能少的情况下，尽快完成第一版的优化工作。

优化方案，直接使用 LocalCache。定时缓存全量标签数据到 LocalCache。

然后，在 DoctorOut 实例内，替换原来给医生附加标签属性的相关操作。



## 讨论（重在讨论の内容，可以没有结论）

出现上面的情况，是为了代码复用，简化开发工作，所以把大部分数据填充的工作都放到了 XxxOut 的构造函数内。

进而出现了上面的问题。

### DoctorOut 职责是否需要保持单一（简单 POJO）？

不建议, 很多数据计算、填充工作都是针对 XxxOut 实例的，散落在业务代码内，会导致相关业务变更时，维护成本变大。

但也不应该在构建函数内，把所有的数据填充工作全部做完，很多业务其实这辈子也不见得会用到。




### DoctorOut 内是否允许 R / W 外部资源

构建函数内，严禁操作外部资源。


### 建议用法

```java

public class Application {

  public static void main(String[] args) {
    Doctor doctor = DAO.get(id);
    
    // 只需要常规属性
    sout(DoctorOut.of(doctor));
    
    // 需要扩展标签属性
	sout(DoctorOut.of(doctor).extendTagNodes(tagNodes));
  }
    
}



public class DoctorOut implements Serializable {
  private String name;
  private List<TagNode> tagNodes;
  
  // 在 of() 内对 `常规的属性` 以及 `可以被快速计算的属性` 赋值
  public static DoctorOut of(Doctor o) {
    DoctorOut out = new DoctorOut();
    out.name = o.name;
    return out;
  }
  
  
  // 通过扩展方法, extendXxx 完成对计算较慢 以及 需要访问外部资源 的属性复制
  public DoctorOut extendTagNodes(List<TagNode> tagNodes) {
    this.tagNodes = tagNodes;
    return this;
  }
  
}
```



### 客户端做 还是 服务器做

>  比如：UI 渲染截图、获取用户当前所在城市