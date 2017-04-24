---
title:  "质量可视化之TEAMCITY数据"
date:   2017-04-24 11:19:06
categories: TEAMCITY
---

## **背景**

之前为了在公司推质量可视化项目，把公司持续集成平台，sonarqube平台的代码项目过程质量数据做收集并集中展示，以此来使得项目过程质量更加透明和可度量，以此来推动研发团队重视质量。

公司的持续集成平台用的是teamcity,如何把持续集成的构建相关质量数据收集起来，是本篇blog的目标。同时在调研过程中也遇到了些问题，并根据调研情况，我们也调整了相应的方案。

## **TEAMCITY REST API**

**[官方文档](https://confluence.jetbrains.com/display/TCD9/REST+API#RESTAPI-GeneralUsagePrinciples)**

**URL Structure**

The general structure of the URL in the TeamCity API is teamcityserver:port/<authType>/app/rest/<entity>, where
- teamcityserver and port define the server name and the port used by TeamCity
- <authType> is the authentication type to be used
- app means that the request will be directed to the TeamCity application
- rest means REST API
- <entity> identifies the required entity. Requests that respond with lists (e.g. .../projects, .../buildTypes, .../builds, .../changes, etc.) serve partial items with only the most important item fields and list "href" s of the items within the list. To get the full item data, use the URL constructed with the value of the "href" item attribute.

备注：其中buildType对应Build configuration

**Locator**

In a number of places, you can specify a filter string which defines what entities to filter/affect in the request. This string representation is referred to as "locator" in the scope of REST API.
The locators formats can be:
- single value: a string without the following symbols: ,:-( )
- dimension, allowing to filter entities using multiple criteria: <dimension1>:<value1>,<dimension2>:<value2>
Refer to each entity description for the supported locators.
If a request with invalid locators is sent, the error messages often hint on the error and list supported locator dimensions (only non-experimental ones) when an unknown dimension is detected.
Note: If the value contains the "," symbol, it should be enclosed into parentheses: "(<value>)".

**Examples**

- 获取所有项目
  -  http://ci.gridsum.com/httpAuth/app/rest/projects
- 获取某个项目信息
  - http://ci.gridsum.com/httpAuth/app/rest/projects/id:NewMedia_TVDissector_V2Etl

-  获取项目下的所有build configurations
  - http://ci.gridsum.com/httpAuth/app/rest/projects/id:NewMedia_TVDissector_V2Etl/buildTypes

- 获取某个build configuration的所有builds
  - http://ci.gridsum.com/httpAuth/app/rest/buildTypes/id:NewMedia_TVDissector_V2Downloader_IntegGitlab/builds

- 获取某个build configuration的所有成功的builds
  - http://ci.gridsum.com/httpAuth/app/rest/buildTypes/id:NewMedia_TVDissector_V2Downloader_IntegGitlab/builds?locator=status:SUCCESS

-  获取某个build configuration的所有失败的builds
  - http://ci.gridsum.com/httpAuth/app/rest/buildTypes/id:NewMedia_TVDissector_V2Downloader_IntegGitlab/builds?locator=status:FAILURE

- 获取从某天开始的所有成功builds
  - http://ci.gridsum.com/httpAuth/app/rest/buildTypes/id:NewMedia_TVDissector_V2Downloader_IntegGitlab/builds?locator=status:SUCCESS,sinceDate:20160501T101003%2B0800

-  获取具体某个build的信息
  - http://ci.gridsum.com/httpAuth/app/rest/buildTypes/id:NewMedia_Sample_Workflow_Feature/builds/id:26292

## **意外**

从以上资料来看，teamcity的api还是比较简单，好用。但是我在实际获取数据的时候发现数据不准确，从api获取到的数据经常要比teamcity UI和数据库里的数据要少。

最后我放弃了teamcity api获取数据，改用从teamcity的数据库里直接读取相应数据。

## **TeamCity 数据库查询SQL**

- total build counts

{% highlight c %}

select count(*) from history 
join build_type_mapping on build_type_mapping.int_id = history.build_type_id
where ext_id = 'NewMedia_TVDissector_TrackerJS_BaseIntegrationJstracker'

{% endhighlight %}

- build fail counts

{% highlight c %}

select count(*) from history 
join build_type_mapping on build_type_mapping.int_id = history.build_type_id
where ext_id = 'NewMedia_TVDissector_TrackerJS_BaseIntegrationJstracker'
and status != 1

{% endhighlight %}

- 测试总数，覆盖率

{% highlight c %}

select 
    build_data_storage.build_id, 
    build_type_mapping.ext_id as 'build_type_id',
    data_storage_dict.value_type_key as 'metric_name', 
    build_data_storage.metric_value,
    history.build_number,
    history.build_finish_time_server
from 
    build_data_storage
join 
    data_storage_dict on build_data_storage.metric_id = data_storage_dict.metric_id
join
    history on build_data_storage.build_id = history.build_id
join
    build_type_mapping on history.build_type_id = build_type_mapping.int_id
where build_type_id='bt354';

{% endhighlight %}

备注：build configuration id 即上面sql中的ext_id