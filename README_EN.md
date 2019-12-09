# ejpa-spring-boot-starter
This is a SpringDataJPA-based project. From Controller-->Service-->Repository-->Entity, there is a corresponding template code.
 
Based on the latest version of SpringBoot, the version of DataJPA corresponding to 2.1.6 The latest version as of 2019-0709

The features provided are:
- CRUD
- Pagination and Sorting -- JPA's default implementation, multi-field complex sorting
- Multi-field complex conditional query

# How to use?
   ## 1. Introducing dependencies:
        <ejpa.starter.version>2.0.0</ejpa.starter.version>
         <dependency>
             <groupId>vip.efactory</groupId>
             <artifactId>ejpa-spring-boot-starter</artifactId>
             <version>${ejpa.starter.version}</version>
             <type>pom</type>
         </dependency>
   ## 2. Inheritance template:
        public class CartonFormingEntity extends BaseEntity<Long> {}
            继承了BaseEntity 就会额外获得基本的常见属性
        public interface CartonFormingRepository extends BaseRepository<CartonFormingEntity, Long> {}
            继承了BaseRepository就会获的模板的持久层一些额外可用的方法,Long是指主键类型
        public interface ICartonFormingService extends IBaseService<CartonFormingEntity, Long> {}
            击沉了IBaseService,在service层就可以使用一些模板里的方法
        public class CartonFormingServiceImpl extends BaseServiceImpl<CartonFormingEntity, Long, CartonFormingRepository> implements ICartonFormingService {}
            在service层就可以使用一些模板里的方法实现,当然你也可以在子类重写覆盖;
        public class CartonFormingController extends BaseController<CartonFormingEntity, ICartonFormingService> {}
            在控制器层提供的一些模板方法
        Public class CartonFormingEntity extends BaseEntity {}
            Inheriting BaseEntity will get additional basic common properties
        Public interface CartonFormingRepository extends BaseRepository<CartonFormingEntity, Long> {}
            Inherit the persistence layer of the template that BaseRepository will get. Some extra methods are available. Long refers to the primary key type.
        Public interface ICartonFormingService extends IBaseService<CartonFormingEntity, Long> {}
            Sinked IBaseService, you can use some methods in the template at the service layer
        Public class CartonFormingServiceImpl extends BaseServiceImpl<CartonFormingEntity, Long, CartonFormingRepository> implements ICartonFormingService {}
            In the service layer you can use some of the methods in the template, of course, you can also override the override in the subclass;
        Public class CartonFormingController extends BaseController<CartonFormingEntity, ICartonFormingService> {}
            Some template methods provided at the controller level
   ## 3.Use
     - In the service and dao layer inheritance will have related methods to achieve, you can use directly, you can also override the overlay;
     - At the controller level, the method is inherited, but an additional processing map is required, otherwise the request cannot be processed, for example:
      /**
          * Description: Use id to get entity
          *
          * @param [id]
          * @return com.ddb.bss.utils.R
          * @author dbdu
          * @date 19-6-11 8:55 AM
          */
         @GetMapping("/{id}")
         @ApiOperation(value = "Acquiring the corresponding record according to Id", notes = "Acquiring the corresponding record according to Id")
         Public R getById(@PathVariable("id") Long id) {
             Return super.getById(id);
         }
      super.getById(id) is the method of the template
     - In the controller, you don't have to explicitly inject the service layer, because ejpa already, for you, called: entityService, you can directly treat it as a property, call its method
       



# Frame default paging sort request
- page -- indicates the first page, the first page of the 0 form;
- size -- indicates a few pages of data;
- sort -- indicates the sorted field, multiple fields are separated by commas, and the last comma separated indicates the sort type, which can be omitted

Indicates the sort type, asc or desc, and the default ASC is continued. For specific parameters, see the PageRequest of the framework:

    Case 1:
        Http://localhost:8080/carton/page?page=0&size=30&sort=id,createTime
        The meaning of this example is to upgrade the id and then to createTime, and return the paging data.
    Case 2:
        Http://localhost:8080/carton/page?page=0&size=30&sort=id,desc&sort=createTime,modifiedTime,desc
        Id - descending
        createTime, modifiedTime -- field descending
    Case 3:
        Http://localhost:8080/carton/page?page=0&size=30&sort=id,desc,createTime,modifiedTime,desc
        Description: The previous desc is a field, and the next desc is a sort type.
        The meaning of this example is that the four fields [id, desc, createTime, modifiedTime] are descending.
    
to sum up:
    1. Sorting with the sort parameter, the sort parameter can be repeated;
    2. The value of the sort parameter can be separated by a comma. If the last comma is followed by desc, the first few fields are descending. If there is no asc, the default is up!


#Advanced Search Instructions:
When there is a lot of traffic data, it is very important to query the data from various angles. Each entity has to write and implement each condition. It is not necessarily good to repeat the workload with large code.

## Three conditions to query as an example:
    What is the relationship between the three conditions? Is it possible to satisfy one condition, or three simultaneous satisfactions? This is the role of relationType below.
    For each condition, what is specific, such as fuzzy query or exact equal to query, which is the decision of searchType

## other instructions:
     A. Currently, the relationship between conditions supports and/or relationships;
     B. The types of individual conditions support 8 common methods;
     C. For the range range query, the support types are Long/Integer/Float/Double/Date, etc.

## Sample conditions
    {
	"relationType": 1, //  0 - or the relationship, can satisfy any of the conditions (default, do not write the default is 0); 1-- and the relationship, meet all conditions; -1-- non-relationship, Inverse condition; --- currently not supported
	"conditions": [    //  All conditional collections need at least one, illegal property background will be automatically filtered out
		{
			"name": "handoverTime",     // the name of the field to search for, which is the attribute name of the entity object
			"searchType": 2,            // Query mode: default is 0, fuzzy query, optional values:
			                                   FUZZY(0, "模糊查询"), fuzzy query
                                               EQ(1, "等于查询"), equal to query
                                               RANGE(2, "范围查询"), range query
                                               NE(3, "不等于查询"), not equal to query
                                               LT(4, "小于查询"), less than query
                                               LE(5, "小于等于查询"), less than or equal to query
                                               GT(6, "大于查询"), greater than query
                                               GE(7, "大于等于查询");greater than or equal to query ; can support more, but it is enough now
			"val": "12:00",             // attribute value, if it is a between type (RANGE), it is the starting value
			"val2": "13:00"             // End value, only required in the between type.
		},
		{
			"name": "id",
			"searchType": 1,
			"val": "2"
		}
	]
    }

   
- The meaning of the example is: there are two query conditions, id field and handoverTime field, the relationship between the two fields is satisfied at the same time: id=2 and the handoverTime value is between "12:00" and "13:00"

Example of use:
Http://localhost:8080/carton/advanced/query

POST method can bring the above Json object!

# V2.0.0 Upgrade content:
- Updated dependent version to 2019-12-9 latest version;
- Optimized the request response body R class;
- The implementation of the primary key is not defined in BaseEntity, and is transplanted into subclasses for greater flexibility;
- Fixed a potential bug in version comparison in the tool class.