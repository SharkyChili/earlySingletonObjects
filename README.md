# earlySingletonObjects

# 三层map

第一级缓存：singletonObjects

第二级缓存：earlySingletonObjects

第三级缓存：singletonFactory


```java
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}

```

方法如图所示
![](https://user-gold-cdn.xitu.io/2020/5/29/1725dda3c8448681?w=1040&h=690&f=png&s=67165)

# 如果只有两层会怎么样
这三层map中earlySingletonObjects看起来好像意义不大，为什么要设计三层？想知道为什么设计三层，等于为什么不设计两层。那么，如果只有两层会怎么办？只有两层的话，那么就得让工厂持续生产，或者生产完毕直接放入singletonObjects
## 如果让工厂持续生产
![](https://user-gold-cdn.xitu.io/2020/5/29/1725de3fccb8a02d?w=798&h=639&f=png&s=61629)
看看工厂生产的代码
```java
	protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
		Object exposedObject = bean;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
	}
```
也就是说，工厂生产的过程就是后置处理器处理的过程。
假如我们有这么个情况，A包含B，B包含A和C，C包含A，如果工厂持续生产，那么C拿A的过程，**会生产第二次，由于beanPostProcessor是可扩展的，那么我们扩展他的时候可能会写出一些bug，例如两次生产的对象不一致等问题。**

可能会有人说，那么只让他生产一次不就可以了吗？那么怎么判断生产过没有呢？最简单的方式不就是弄一层缓存吗？这不就是earlySingletonObjects吗？

## 如果工厂生产完毕就放入singletonObjects
![](https://user-gold-cdn.xitu.io/2020/5/29/1725de918bc387a7?w=1088&h=762&f=png&s=78415)
singletonObjects存放的是bean的最终完成品，是直接可以在外部getBean（）得到的bean。
举个例子，Bean创建的过程包含属性填充这个阶段，如果生产完毕就放入singletonObjects，那么属性填充怎么办呢？怎么区分singletonObjects中的bean属性填充了没有？

# 感想
我真想惩罚下我这个脑子，有这功夫学点别的不香吗？
