# iOS深拷贝和浅拷贝

### 浅拷贝

简单点说浅拷贝就是对内存地址的复制，让目标对象指针和源对象指针指向同一片内存空间。如：

```
char *str = (char *)malloc(100);
char *str2 = str;
```

上述例子就是浅拷贝最好的实例，浅拷贝就是简单的`拷贝地址`，让几个对象共同指向`同一块内存`。当内存销毁时，指向该内存的其他指针需重新指向，否则将成为野指针

### 深拷贝

深拷贝就是拷贝地址中的内容，让目标对象产生`新的内存区域`，且将源内存区域中的内容复制到目标内存区域中

```
char *str = (char *)malloc(20);
strcpy(str, "hello world");

//复制
char *str2 = (char *)malloc(20);
strcpy(str2, str);
```

深拷贝就是产生一个新的对象，将源对象的所有内容拷贝到新的对象中，新对象和源对象各自指向自己的内存区域，相互之间不受影响。

## iOS中的深/浅拷贝

对于`浅拷贝`，只是新创建了类的空间，然后将属性的值复制一遍；对于属性所指向的内存空间并没有重新创建；因此通过浅拷贝的新旧两个对象的属性其实还是指向一块`相同的内存空间`。

而深拷贝，不仅仅新创建了类的空间，还新创建了每一个属性对应的空间，所以深拷贝也称为完全拷贝；通过深拷贝得来的新对象和旧对象，两个对象的属性都是指向`各自的内存空间`，不再共享空间。

## 自定义类的深拷贝和浅拷贝

在OC中要想自定义的类具有`拷贝功能`(也就是能用`copy`方法)则类必须遵守`NSCopying`协议，并且实现协议中的`(id)copyWithZone:(NSZone *)zone`方法;

在这个方法中我们可以根据自己的实际情况来实现，不同的实现方式，`copy`出来的对象具有不同的效果
类的深/浅拷贝也就是在这个方法中体现，具体如下

#### 浅拷贝方法的实现

```
-(id)copyWithZone:(NSZone *)zone
{
    //创建新的对象空间
    Student *stu = [[self class] allocWithZone:zone];

    //将属性复制---其实只是复制了地址
    stu.name = self.name;
    stu.sex = self.sex;
    stu.age = self.age;

    return stu;
}
```

#### 深拷贝方法的实现

```
-(id)copyWithZone:(NSZone *)zone
{
    //创建新的对象空间
    Student *stu = [[self class] allocWithZone:zone];

    //为每个属性创建新的空间，并将内容复制
    stu.name = [[NSString alloc] initWithString:self.name];
    stu.sex = [[NSString alloc] initWithString:self.sex];
    stu.age = self.age;

    return stu;
}
```

### 伪拷贝方法的实现

其实对于`-(id)copyWithZone:(NSZone *)zone`方法的实现还有第三种，我们把他叫做伪拷贝，具体实现如下：

```
-(id)copyWithZone:(NSZone *)zone
{
    //拷贝自己本身地址
    return [self retain];
}
```

## 使用中的深拷贝和浅拷贝

在我们的使用中自定义类实现的方式不一样，拷贝出来的效果也不一样，同样一段代码，深拷贝和浅拷贝的最终效果如下：

```
    Student *s1 = [[Student alloc] init];
    s1.name = @"tanyufeng";
    s1.sex = @"男";

    Student *s2 = [s1 copy];
```

#### 伪拷贝的效果

![](media/1240.)

伪拷贝效果图



#### 浅拷贝的效果

![](media/1240-1.)

浅拷贝效果图



#### 深拷贝的效果

![](media/1240-2.)


### Strong、Copy和MutableCopy的区别

1.strong：始终是浅复制。引用计数每次加一。返回对象是否可变与被复制的对象保持一致。

2.copy：对于可变对象为深复制，引用计数不改变;对于不可变对象是浅复制， 引用计数每次加一。始终返回一个不可变对象。

3.mutableCopy：始终是深复制，引用计数不改变。始终返回一个可变对象。

![](media/1240-3.)

注意：其他对象NSArray、NSMutableArray 、NSDictionary、NSMutableDictionary一样适用

![](media/1240-4.)

