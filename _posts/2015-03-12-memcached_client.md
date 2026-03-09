---
layout: post
title: "memcached java client"
description: " "
keywords: "memcached, java, 缓存"
category: life
tags: [memcached, 缓存]
---

##基于alisoft-xplatform-asf-cache-2.5.1
1 依赖的jar

A. alisoft-xplatform-asf-cache-2.5.1.jar

B. commons-logging-1.0.4.jar

C. hessian-3.0.1.jar

D. log4j-1.2.9.jar

E. stax-api-1.0.1.jar

F. wstx-asl-2.0.2.jar

2  配置` memcached.xml ` 文件路径 ` /WEB-INF/classes/memcached.xml`
 
{% highlight xml%}
<?xml version="1.0" encoding="UTF-8"?>  
<memcached>  
    <!-- name 属性是程序中使用Cache的唯一标识;socketpool 属性将会关联到后面的socketpool配置; -->  
    <client name="mclient_0" compressEnable="true" defaultEncoding="UTF-8"  
        socketpool="pool_0">  
        <!-- 可选，用来处理出错情况 -->  
        <errorHandler>com.alisoft.xplatform.asf.cache.memcached.MemcachedErrorHandler  
        </errorHandler>  
    </client>  
  
    <!--  
        name 属性和client 配置中的socketpool 属性相关联。  
        maintSleep属性是后台线程管理SocketIO池的检查间隔时间，如果设置为0，则表明不需要后台线程维护SocketIO线程池，默认需要管理。  
        socketTO 属性是Socket操作超时配置，单位ms。 aliveCheck  
        属性表示在使用Socket以前是否先检查Socket状态。  
    -->  
    <socketpool name="pool_0" maintSleep="5000" socketTO="3000"  
        failover="true" aliveCheck="true" initConn="5" minConn="5" maxConn="250"  
        nagle="false">  
        <!-- 设置memcache服务端实例地址.多个地址用","隔开 -->  
        <servers>127.0.0.1:11211</servers>  
        <!--  
            可选配置。表明了上面设置的服务器实例的Load权重. 例如 <weights>3,7</weights> 表示30% load 在  
            10.2.224.36:33001, 70% load 在 10.2.224.46:33001  
          
        <weights>3,7</weights>  
        -->  
    </socketpool>  
</memcached> 

{% endhighlight %}

3  测试代码
{% highlight java%}
package com.hl.memcached.client.test;  
  
import java.util.ArrayList;  
import java.util.List;    
import com.alisoft.xplatform.asf.cache.ICacheManager;  
import com.alisoft.xplatform.asf.cache.IMemcachedCache;  
import com.alisoft.xplatform.asf.cache.memcached.CacheUtil;  
import com.alisoft.xplatform.asf.cache.memcached.MemcachedCacheManager;  
import com.hl.memcached.cache.client.TestBean;  
  
public class ClientTest {  
      
    @SuppressWarnings("unchecked")  
    public static void main(String[] args) {  
        ICacheManager<IMemcachedCache> manager;  
        manager = CacheUtil.getCacheManager(IMemcachedCache.class,  
                MemcachedCacheManager.class.getName());  
        manager.setConfigFile("memcached.xml");  
        manager.start();  
        try {  
            IMemcachedCache cache = manager.getCache("mclient_0");  
  
            TestBean bean=new TestBean();  
            bean.setName("name1");  
            bean.setAge(25);  
            cache.put("bean", bean);  
            TestBean beanClient=(TestBean)cache.get("bean");  
            System.out.println(beanClient.getName());  
              
            List<TestBean> list=new ArrayList<TestBean>();  
            list.add(bean);  
            cache.put("beanList", list);  
            List<TestBean> listClient=(List<TestBean>)cache.get("beanList");  
            if(listClient.size()>0){  
                TestBean bean4List=listClient.get(0);  
                System.out.println(bean4List.getName());  
            }  
        } finally {  
            manager.stop();  
        }  
    }  
  
}  
{% endhighlight %}

参考 [http://blog.csdn.net/seelye/article/details/8511073](http://blog.csdn.net/seelye/article/details/8511073)