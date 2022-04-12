前面两篇文章，介绍了Java基础的反射与注解，Spring的IOC容器实现的核心原理便用到了反射与注解(注解形式的配置)，看完反射与注解的同学再来看这篇博客，便能对反射与注解的知识了解扎实

[注解](https://blog.csdn.net/weixin_44706647/article/details/113510784)

[反射--Java框架设计的灵魂](https://blog.csdn.net/weixin_44706647/article/details/113481175)

### 思路分析

有三个注解：

- Component：要被加载的bean
- ComponentScan：bean所在的包
- Value：该bean的一些属性赋值

流程分析：

![image-20210202154628517](http://cdn.noteblogs.cn/image-20210202154628517.png)

代码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Component {
}
```

```
/**
 * @author 四五又十
 * @create 2021/2/1 20:27
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ComponentScan {
    String[] value();
}
```

```
/**
 * @author 四五又十
 * @create 2021/2/1 20:27
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Value {
    String value();
}
```

核心代码类

```java
package cn.noteblogs.reflect;
import cn.noteblogs.Application;
import cn.noteblogs.annotation.Component;
import cn.noteblogs.annotation.ComponentScan;
import cn.noteblogs.annotation.Value;

import java.io.File;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;


public class AnnotationConfigApplicationContext {
    private String path;
    private Map<Class, Object> beanFactory = new HashMap<>();
    public AnnotationConfigApplicationContext()  {
        this.initBeanByAnnotation();
    }
    public Object getBean(Class clazz){
        return beanFactory.get(clazz);
    }
    public void initBeanByAnnotation() {
        path = AnnotationConfigApplicationContext.class.getClassLoader().getResource("").getFile();
        // /E:/IdeaProjecty/SpringIOCTest/out/production/SpringIOCTest/
        //System.out.println(path);
        Class<Application>  clazz = Application.class;
        ComponentScan componentScan = clazz.getAnnotation(ComponentScan.class);
        if(componentScan != null){
            String[] value = componentScan.value();
            for (String componentPath : value) {
                if(componentPath != null && componentPath.length() > 0){
                    // cn.noteblogs.bean
                    //System.out.println(componentPath);
                    loadClassInDefinedDir(componentPath);
                }
            }
        }

    }

    private void loadClassInDefinedDir(String componentPath)  {
        //cn.noteblogs.bean 需要将这个路径变为 /E:/IdeaProjecty/SpringIOCTest/out/production/SpringIOCTest/cn/noteblogs/bean
        componentPath = componentPath.replaceAll("\\.", "/");
        String fileDir = path + componentPath;
        ///E:/IdeaProjecty/SpringIOCTest/out/production/SpringIOCTest/cn/noteblogs/bean
        //System.out.println(classPath);
        //加载这个文件夹下的所有class类，并判断是否有@Component
        findClassByFile(new File(fileDir));
    }

    private void findClassByFile(File fileDir)  {
        if(fileDir.isDirectory()){
            File[] files = fileDir.listFiles();
            if(files == null || files.length == 0)  return;
            for (File file : files) {
                if(file.isDirectory()){
                    findClassByFile(file);
                }else{
                    loadClassByAnnotation(file);
                }

            }
        }
    }

    private void loadClassByAnnotation(File file)  {
        //   /E:/IdeaProjecty/SpringIOCTest/out/production/SpringIOCTest/cn/noteblogs/bean/user.class
        //   变为cn.noteblogs.bean.user
        String classPath = file.getAbsolutePath().substring(path.length() - 1).replaceAll("\\\\", "\\.");
        //cn.noteblogs.bean.user.class
       // System.out.println(classPath);
        if(classPath != null && classPath.length() > 0 && classPath.contains(".class")){
            classPath = classPath.replaceAll("\\.class", "");
            //System.out.println(classPath);
            Class<?> clazz = null;
            try {
                clazz = Class.forName(classPath);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
            Component component = clazz.getAnnotation(Component.class);
            if(component != null){
                Object instance = null;
                try {
                    instance = clazz.newInstance();
                } catch (InstantiationException e) {
                    e.printStackTrace();
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
                Class<?>[] interfaces = clazz.getInterfaces();
                // Map<UserDao.class, UserDaoImpl>
                if(interfaces != null && interfaces.length > 0){
                    beanFactory.put(interfaces[0], instance);
                }else{
                    beanFactory.put(clazz, instance);
                }
                //获取该对象的所有域，然后判断是否有value注解进行赋值
                Field[] declaredFields = clazz.getDeclaredFields();
                if(declaredFields != null && declaredFields.length > 0){
                    for (Field f : declaredFields) {

                        Value annotation = f.getAnnotation(Value.class);
                        if(annotation != null){
                            String value = annotation.value();
                            f.setAccessible(true);
                            try {
                                f.set(instance, value);
                            } catch (IllegalAccessException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }

            }
        }
    }
}
```

```java
@ComponentScan({"cn.noteblogs.bean","cn.noteblogs.dao"})
public class Application {
    public static void main(String[] args)  {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext();
        User user = (User)ac.getBean(User.class);
        System.out.println(user);
        UserDao dao = (UserDao)ac.getBean(UserDao.class);
        System.out.println(dao.getClass());
    }
}
```

整体项目结构

![image-20210202154913770](http://cdn.noteblogs.cn/image-20210202154913770.png)

