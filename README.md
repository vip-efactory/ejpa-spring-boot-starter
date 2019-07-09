# ejpa-spring-boot-starter
这是一个基于SpringDataJPA的项目，从Controller-->Service-->Repository-->Entity,都有相应的模板代码.
 
基于SpringBoot的最新版,2.1.6对应的DataJPA的版本  截至2019-0709时候的最新版

提供的功能有:
- CRUD
- 分页与排序 --JPA的默认实现,多字段复杂排序
- 多字段复杂条件的查询

# 如何引入使用使用
 ## 1.引入依赖:
        <ejpa.starter.version>1.0.0</ejpa.starter.version>
         <dependency>
             <groupId>vip.efactory</groupId>
             <artifactId>ejpa-spring-boot-starter</artifactId>
             <version>${ejpa.starter.version}</version>
             <type>pom</type>
         </dependency>
  ## 2.继承模板:
        public class CartonFormingEntity extends BaseEntity {}
            继承了BaseEntity 就会额外获得基本的常见属性
        public interface CartonFormingRepository extends BaseRepository<CartonFormingEntity, Long> {}
            继承了BaseRepository就会获的模板的持久层一些额外可用的方法,Long是指主键类型
        public interface ICartonFormingService extends IBaseService<CartonFormingEntity, Long> {}
            击沉了IBaseService,在service层就可以使用一些模板里的方法
        public class CartonFormingServiceImpl extends BaseServiceImpl<CartonFormingEntity, Long, CartonFormingRepository> implements ICartonFormingService {}
            在service层就可以使用一些模板里的方法实现,当然你也可以在子类重写覆盖;
        public class CartonFormingController extends BaseController<CartonFormingEntity, ICartonFormingService> {}
            在控制器层提供的一些模板方法
  ## 3.使用
     - 在service及dao层继承了就会有相关的方法实现,你可以直接使用,也可以重写覆盖;
     - 在控制器层,方法是继承了,但是需要外加处理映射,不然不能处理请求,例如:
      /**
          * Description: 使用id来获取实体
          *
          * @param [id]
          * @return com.ddb.bss.utils.R
          * @author dbdu
          * @date 19-6-11 上午8:55
          */
         @GetMapping("/{id}")
         @ApiOperation(value = "依据Id来获取对应的记录", notes = "依据Id来获取对应的记录")
         public R getById(@PathVariable("id") Long id) {
             return super.getById(id);
         }
      super.getById(id) 就是模板的方法
     - 在控制器里,你不必在显式注入service层,因为ejpa已经,替你注入,叫:entityService,你可以直接把它当成属性,去调用他的方法
       



#框架默认的分页排序请求
    - page    --表示第几页,0表式第一页;
    - size    --表示一页几条数据;
    - sort    --表示排序的字段,多个字段用逗号分隔,最后一个逗号分隔的表示排序类型,可以省略
表示排序类型,asc或者desc,默认ASC升续,具体参数参见框架的PageRequest:

     案例一:
        http://localhost:8080/carton/page?page=0&size=30&sort=id,createTime
        这个例子的意思是先对id升续后再对createTime升续,返回分页数据
     案例二:
        http://localhost:8080/carton/page?page=0&size=30&sort=id,desc&sort=createTime,modifiedTime,desc
        id          --降序
        createTime,modifiedTime  --字段降序
     案例三:
        http://localhost:8080/carton/page?page=0&size=30&sort=id,desc,createTime,modifiedTime,desc
        说明:前一个desc当成字段,后面一个desc是排序类型
        此例的意思是[id,desc,createTime,modifiedTime]这个四个字段都是降序
    总结:
    1.排序用sort参数,sort参数可以重复出现;
    2.sort参数值可用逗号分隔,最后一个逗号后面如果是desc则前面几个字段都是降序,若没有asc则默认升续!




#高级搜索的使用说明:
当业务量数据很多的时候,查询数据从各种角度查询数据就显得很重要了,每个实体每个条件都去写实现,工作量大代码相似重复也未必好!
## 三个条件来查询为例:
    三个条件的关系是什么?是满足一个条件就可以了,还是三个同时满足,这个就是下面relationType的作用
    对于每个条件,具体又是什么,比如是模糊查询还是精确的等于查询,这是searchType的决定的

## 其他说明:
     A.目前,条件之间的关系支持与/或关系;
     B.单个条件的类型支持8种常用的方式;
     C.对于between范围查询,支持类型有Long/Integer/Float/Double/Date等

## 样例条件
    {
	"relationType": 1, //  ０--或的关系，满足任意一个条件即可(默认,不写默认为0)；１--与的关系，满足所有条件；-1--非的关系，条件取反；---目前不支持
	"conditions": [    // 所有的条件集合,至少需要一个,非法属性后台会自动过滤掉
		{
			"name": "handoverTime",     // 要搜索的字段名,也就是实体对象的属性名
			"searchType": 2,            // 查询方式:默认为0,模糊查询,可选值:
			                                   FUZZY(0, "模糊查询"),
                                               EQ(1, "等于查询"),
                                               RANGE(2, "范围查询"),
                                               NE(3, "不等于查询"),
                                               LT(4, "小于查询"),
                                               LE(5, "小于等于查询"),
                                               GT(6, "大于查询"),
                                               GE(7, "大于等于查询"); 可以支持更多,但是目前已经够用的了
			"val": "12:00",             // 属性值,若是between类型(RANGE)则为开始值
			"val2": "13:00"             // 结束值,仅在between类型是才需要.
		},
		{
			"name": "id",
			"searchType": 1,
			"val": "2"
		}
	]
    }
    样例的含义是:有两个查询条件,id字段和handoverTime字段,两个字段的关系是同时满足:id=2且handoverTime值在"12:00"与"13:00"之间
使用的例子:
http://localhost:8080/carton/advanced/query 

POST的方法带上上面的Json对象即可!
