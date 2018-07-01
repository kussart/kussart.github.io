---
layout: post
comments: true
title:  "Swagger for your Spring REST API?"
---


Sometimes you sit and do some part of your work, you set up your beautiful server, adding new functionality, new controllers, logic and so on. 

And while you're debugging everything, a group of interested  front-end  developers sits literally in anticipation, when they can finally fix the administrative part so that users enjoy browsing into their account with a new and amazing functionality.
And there are questions:

*-What did you do there, Artem? How does it work, how should we understand what your new controller is waiting for and what it will return, and if NULL, where to show the modal window?*

*-Damn!*


My server was not described by any API, really, there was no description absolutely, everything was done by sorting out some special cases, the problems were solved on the knee and it was violated the balance of the peaceful developers universe. 

*-This is unforgivable!*

I decided to describe the whole thing, but then I was horrified by the number of controllers and variants, examples, etc. Do not write it all in hand, it can not be for me.

Of course there is already a long time convenient tools for automatic API generation for your services, which will do everything for you with the correct configuration.
In our case, I'll talk about the configuration of *Swagger 2* + *Spring Framework* (with XML config) + *Shiro* + *Maven* to show our API at *.../rest/swagger-ui.html*



To begin with, let's connect the libraries via Maven, adding to *pom.xml*:

	<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.7.0</version>
	</dependency>
	<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.7.0</version>
	</dependency>

Ok, now let's add a *configuration class for Swagger*:

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig extends WebMvcConfigurationSupport {

@Bean
public Docket api(){
	return new Docket(DocumentationType.SWAGGER_2)
		.select()		
		.apis(RequestHandlerSelectors
			.basePackage("com.home.blog.my_controller_package"))
		.paths(PathSelectors.any())
		.build()
		.apiInfo(metaData());				
	}

private ApiInfo metaData() {
	return new ApiInfoBuilder()
			.title("My REST API")
			.description("It's a cool REST API")
			.version("1.0-release")
			.license("")
			.licenseUrl("")
			.contact(new Contact("", "", ""))
			.build();
		}
	}
```
Great, now let's add this bean to the configuration of our servlets, in my case the servlet that handles rest requests is described in the file *spring-mvc-servlet.xml*:
```xml
<bean class="com.home.blog.my_config_package.SwaggerConfig"/>

<mvc:resources mapping="swagger-ui.html" location="classpath:/META-INF/resources/"/>
<mvc:resources mapping="/webjars/**" location="classpath:/META-INF/resources/webjars/"/>
```

Since we use Shiro to filter requests, let's add permissions for Swagger access, note, we need to give access *to all these resources* (if you use a different protection configuration, not Shiro, then see where and how to describe these paths) :
```xml
<bean id="shiroFilter" 	class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	<property name="filterDefinitions">
		<value>
			/rest/swagger-ui.html = anon
			/rest/v2/api-docs = anon
			/rest/configuration/** = anon
			/rest/swagger-resources/** = anon
			/rest/webjars/** = anon
		</value>
	</property> 
</bean>
```

Cool, there was one important point - to sign our controllers in the right way. I want to draw your attention to the fact that I encountered a problem when the controller had an annotation of the form:
```java
@RequestMapping("/test")
```
Such a thing did *not work*, as well as the lack of annotation at all. 
Therefore, it is important that you have:
```java
@RequestMapping(value = "/test")
```
Then everything will be all right.
```java
@Controller
@RequestMapping(value = "/test")
@Api(value="test actions", description="Test from my blog")
public class TestController {

@Autowired
private TestService testService;

@ApiOperation(value = "View my Test list", response = List.class)
@RequestMapping(value = "/list", method = RequestMethod.GET)
@ResponseBody
public List<Test> get() {
	return testService.getTests();
}
```	
*@Api* and *@ApiOperation* annotations help to correct the text that is output in swagger-ui.html, you can also describe in detail each method, returned errors, models, and so on. But let's leave it outside the framework of this post.

All, now, if you did everything right, after launching your application, you can go to the browser on the path you_path/rest/swagger-ui.html
and see the description of each controller and the ability to send test queries.
For more details you can visit [https://swagger.io/specification/](https://swagger.io/specification/)	

{% if page.comments %}
<div id="disqus_thread"></div>
<script>
(function() { 
var d = document, s = d.createElement('script');
s.src = 'https://https-kussart-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
{% endif %}