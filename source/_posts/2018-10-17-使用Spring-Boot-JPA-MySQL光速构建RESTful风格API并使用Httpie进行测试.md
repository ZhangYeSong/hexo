---
title: 使用Spring Boot+JPA+MySQL光速构建RESTful风格API并使用Httpie进行测试
date: 2018-10-17 21:13:56
categories:
 - Java
tags:
 - Java
 - Spring Boot
 - RESTful
 - JPA
---
> 笔者是个懒人，但是真没想到写一个RESTful风格的API接口可以这么快。官方文档地址：https://spring.io/guides/gs/accessing-data-rest/

#### 0. 使用Spring Initializ初始化一个Spring Boot项目

用Idea的话点击New Project选择Spring Initializ, 一路next填上项目名包名等。然后可以先选上依赖，也可以就选一个core，后面在pom文件里写。

不用Idea的话直接去https://start.spring.io，选择maven，java（你也可以试试gradle+kotlin），然后选上依赖，当然我们这里先不选，生成好后下载下来。然后再用自己的IDE或者编辑器打开。
<!-- more -->
#### 1. 在pom文件里添加依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.45</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

添加如上依赖，然后用maven sync一下把依赖包都下下来。

简单说下，spring-boot-starter-web和spring-boot-starter-test是构建web应用必须要有的;mysql-connector-java是mysql的驱动;spring-boot-starter-data-jpa是一个ORM（Object Relational Mapping）对象映射框架，简单来说就是把应用内存中的对象和数据库中的数据建立映射关系，熟悉Android的朋友们知道的GreenDao和它做的就是同一类事;spring-boot-starter-data-rest是今天的重头戏，它真的可以帮助我们以光速写一个RESTful的API接口。

#### 2.配置数据库

开启自己的Mysql服务，create一个数据库，我这里叫cloud_mall，然后在application.properties中写上连接数据库相关的参数：

```properties
spring.jpa.hibernate.ddl-auto=create
spring.datasource.url=jdbc:mysql://localhost:3306/cloud_mall?useSSL=false
spring.datasource.username=root
spring.datasource.password=xxxx
```

这其中注意第一个参数，主要有以下这几种常见选择：

- ddl-auto:create----没有表的Entity会在数据库中建表，已经有的会清空数据。

- ddl-auto:create-drop----每次程序结束的时候会清空表。

- ddl-auto:update----会根据Entity更新表的结构。
- ddl-auto:validate----运行程序会校验数据与数据库的字段类型是否相同，不同会报错。

因此第一次需要创建表的时候我们用create，后面再次运行我们用update比较好。

ddl-auto:validate----运行程序会校验数据与数据库的字段类型是否相同，不同会报错
--------------------- 
作者：万里飞鹏 
来源：CSDN 
原文：https://blog.csdn.net/zhangtongpeng/article/details/79609942 
版权声明：本文为博主原创文章，转载请附上博文链接！

#### 3.建一个叫Commodity的商品java bean类

```java
@Entity
public class  Commodity {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private int category;

    private String name;

    private String subTitle;

    private String picUrl;

    private long price;

    private int stock;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSubTitle() {
        return subTitle;
    }

    public void setSubTitle(String subTitle) {
        this.subTitle = subTitle;
    }

    public int getCategory() {
        return category;
    }

    public void setCategory(int category) {
        this.category = category;
    }

    public String getPicUrl() {
        return picUrl;
    }

    public void setPicUrl(String picUrl) {
        this.picUrl = picUrl;
    }

    public long getPrice() {
        return price;
    }

    public void setPrice(long price) {
        this.price = price;
    }

    public int getStock() {
        return stock;
    }

    public void setStock(int stock) {
        this.stock = stock;
    }
}
```

#### 4. 写一个继承PagingAndSortingRepository的Repository接口

```java
@RepositoryRestResource(collectionResourceRel = "commodity", path = "commodity")
public interface CommodityRepository extends PagingAndSortingRepository<Commodity, Long>{
    List<Commodity> findByName(@Param("name") String name);
}
```

是的，你没看错就这么简单，写完了。先看下自己的MySql数据库commodity表有没有建好，数据结构是不是和我们写的Commodity类一样。

其中findByName是自己写的，你可以按照提示写其他的查找方法，至于剩下的CURD，统统都让spring-boot-starter-data-rest框架自己实现好了。

#### 5.下载Httpie准备测试

这里为啥给大家推荐这个命令行软件呢？Postman和Restlet Client这些Chrome插件已经够好用了啊？确实如此，但是笔者最近迷恋上了命令行- -。命令行软件的好处就是快，不用点来点去，适合写完一个接口立马测试用，系统性的测试API肯定还是Postman这些好使。另外就是命令行可以在没有GUI服务器上直接使用。当然curl也可以,一开始我也是用curl，但是参数太多，用起来还是有点不太方便。

下载方式就不写了，见官方网站：https://httpie.org/

#### 6. 使用POST添加数据

现在我们直接在终端输入http localhost:8080可以试一下，没用报错，但是没数据。因此我们先用POST请求加几个数据。直接输入http POST url json参数，注意string类型的直接key=value就好，整形和浮点型用key:=value。

```bash
judy:~$ http POST localhost:8080/commodity name=IPhone7 subTitle=IPhone7 price:=2588.88 stock:=12
HTTP/1.1 201 
Content-Type: application/json;charset=UTF-8
Date: Wed, 17 Oct 2018 11:39:50 GMT
Location: http://localhost:8080/commodity/3
Transfer-Encoding: chunked

{
    "_links": {
        "commodity": {
            "href": "http://localhost:8080/commodity/3"
        }, 
        "self": {
            "href": "http://localhost:8080/commodity/3"
        }
    }, 
    "category": 0, 
    "name": "IPhone7", 
    "picUrl": null, 
    "price": 2588, 
    "stock": 12, 
    "subTitle": "IPhone7"
}
```

可以看到返回的数据中给出了我这个IPhone7的url，以及详细属性，为了证明添加成功大家可以去数据库查一下表。

#### 7.使用PUT替换数据

使用http PUT url json参数来替换数据，这里我要修改的是刚才POST上去的IPhone7, 因此url使用刚才POST返回的url。

```bash
judy:~$ http PUT localhost:8080/commodity/3 name=IPhone7Plus subTitle=IPhone7Plus price:=2588.88 stock:=12
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Wed, 17 Oct 2018 11:48:31 GMT
Location: http://localhost:8080/commodity/3
Transfer-Encoding: chunked

{
    "_links": {
        "commodity": {
            "href": "http://localhost:8080/commodity/3"
        }, 
        "self": {
            "href": "http://localhost:8080/commodity/3"
        }
    }, 
    "category": 0, 
    "name": "IPhone7Plus", 
    "picUrl": null, 
    "price": 2588, 
    "stock": 12, 
    "subTitle": "IPhone7Plus"
}

```

可以看到返回结果是没问题的。

#### 8.使用PATCH来修改数据

PUT和PATCH是很容易弄混的两个请求，PUT是整体替换，而PATCH只会改变部分属性。命令格式和PUT一样，只是由于我们只修改部分属性，因此只要把修改的属性放上来就行了，这里我修改了下价格。

```bash
judy:~$ http PATCH localhost:8080/commodity/3 price:=3588HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Wed, 17 Oct 2018 11:54:57 GMT
Transfer-Encoding: chunked

{
    "_links": {
        "commodity": {
            "href": "http://localhost:8080/commodity/3"
        }, 
        "self": {
            "href": "http://localhost:8080/commodity/3"
        }
    }, 
    "category": 0, 
    "name": "IPhone7Plus", 
    "picUrl": null, 
    "price": 3588, 
    "stock": 12, 
    "subTitle": "IPhone7Plus"
}

```

修改成功。

#### 9.使用DELETE请求删除数据

都不用猜了，肯定是http DELETE url这样的形式，来试一下：

```bash
judy:~$ http DELETE localhost:8080/commodity/3
HTTP/1.1 204 
Date: Wed, 17 Oct 2018 11:58:21 GMT

```

居然什么提示也没有，别着急，去数据库查一下是不是删除成功了呢？

#### 10.使用GET请求查找数据

GET的话是最常用的，因此可以省略不写。查找之前先用POST添加几条数据。

我们来试一下：

```BASH
judy:~$ http localhost:8080/commodity
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Wed, 17 Oct 2018 11:58:46 GMT
Transfer-Encoding: chunked

{
    "_embedded": {
        "commodity": [
            {
                "_links": {
                    "commodity": {
                        "href": "http://localhost:8080/commodity/1"
                    }, 
                    "self": {
                        "href": "http://localhost:8080/commodity/1"
                    }
                }, 
                "category": 0, 
                "name": "IPhone5", 
                "picUrl": null, 
                "price": 588, 
                "stock": 10, 
                "subTitle": "IPhone5"
            }, 
            {
                "_links": {
                    "commodity": {
                        "href": "http://localhost:8080/commodity/2"
                    }, 
                    "self": {
                        "href": "http://localhost:8080/commodity/2"
                    }
                }, 
                "category": 0, 
                "name": "IPhone6", 
                "picUrl": null, 
                "price": 988, 
                "stock": 20, 
                "subTitle": "IPhone6"
            }
        ]
    }, 
    "_links": {
        "profile": {
            "href": "http://localhost:8080/profile/commodity"
        }, 
        "search": {
            "href": "http://localhost:8080/commodity/search"
        }, 
        "self": {
            "href": "http://localhost:8080/commodity{?page,size,sort}", 
            "templated": true
        }
    }, 
    "page": {
        "number": 0, 
        "size": 20, 
        "totalElements": 2, 
        "totalPages": 1
    }
}

```

可以看到我们要的数据都能查到，并且会提示我们可以用page、size、sort这些参数来控制查询范围和排序。还提示了我们可以用http://localhost:8080/commodity/search来查询,查完会提示你用http://localhost:8080/commodity/search/findByName{?name}这个url来查询，这也是我们自己定义的一个抽象查询方法，试一下：

```bash
judy:~$ http localhost:8080/commodity/search/findByName?name=IPhone5
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Wed, 17 Oct 2018 12:04:38 GMT
Transfer-Encoding: chunked

{
    "_embedded": {
        "commodity": [
            {
                "_links": {
                    "commodity": {
                        "href": "http://localhost:8080/commodity/1"
                    }, 
                    "self": {
                        "href": "http://localhost:8080/commodity/1"
                    }
                }, 
                "category": 0, 
                "name": "IPhone5", 
                "picUrl": null, 
                "price": 588, 
                "stock": 10, 
                "subTitle": "IPhone5"
            }
        ]
    }, 
    "_links": {
        "self": {
            "href": "http://localhost:8080/commodity/search/findByName?name=IPhone5"
        }
    }
}

```

果然查到了，是不是很强大呢？当然了，用框架生成的接口灵活性还是不如自己写的接口。不过个人玩的话真的是懒人福利了。有疑问的大家可以互相交流。
