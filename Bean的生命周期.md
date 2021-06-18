# Bean的生命周期

### 对象创建

1、从xml配置的Bean,@Bean注解，或者Java代码BeanDefinitionBuilder中读取Bean的定义,（反射）实例化Bean对象；

2、设置Bean的属性；

3、注入Aware的依赖（BeanNameAware,BeanFactoryAware,ApplicationContextAware）;

4、执行通用的方法前置处理，方法： BeanPostProcessor.postProcessorBeforeInitialization()

5、执行 InitalizingBean.afterPropertiesSet() 方法

6、执行Bean自定义的初始化方法init,或者 @PostConstruct 标注的方法；

7、执行方法BeanPostProcessor.postProcessorAfterInitialization()

8、创建对象完毕；

### 对象销毁

9、执行 DisposableBean.destory() 方法；

10、执行自定义的destory方法或者 @PreDestory 标注的方法；

11、销毁对象完毕

