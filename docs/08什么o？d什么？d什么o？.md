## 《阿里巴巴Java开发手册v1.4.0（详尽版）》

#### 工程结构 --> 应用分层 --> 分层领域模型规约

https://yq.aliyun.com/articles/69327



模型规约：

- DO（Data Object）：
  - 数据库实体类
  - 此对象与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
- DTO（Data Transfer Object）：
  - 数据传输对象
  - Service 或 Manager 向外传输的对象。
- BO（Business Object）：
  - Service逻辑处理用到的POJO
  - 业务对象，由 Service 层输出的封装业务逻辑的对象。
- AO（Application Object）：
  - Web层不用于展示的POJO
  - 应用对象，在 Web 层与 Service 层之间抽象的复用对象模型，
    极为贴近展示层，复用度不高。
- VO（View Object）：
  - 显示层对象，通常是 Web 向模板渲染引擎层传输的对象。
- Query：
  - 用于查询数据库的条件封装类
  - 数据查询对象，各层接收上层的查询请求。注意超过 2 个参数的查询封装，禁止
    使用 Map 类来传输。
- POJO（ Plain Ordinary Java Object）：在本手册中， POJO专指只有setter/getter/toString的简单类，包括DO/DTO/BO/VO等。



命名规约：

- 数据对象：xxxDO，xxx即为数据表名。
- 数据传输对象：xxxDTO，xxx为业务领域相关的名称。
- 展示对象：xxxVO，xxx一般为网页名称。
- POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。