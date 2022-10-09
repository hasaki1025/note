# Mybatis源码（1）

- 入口语句

  ```java
  String config="mybatis-config.xml";
  InputStream in = Resources.getResourceAsStream(config);
  ```

- Resources类调用getResourceAsStream方法

  ```java
  public static InputStream getResourceAsStream(String resource) throws IOException {
      return getResourceAsStream(null, resource);
  }
  
  //方法重载
  public static InputStream getResourceAsStream(ClassLoader loader, String resource) throws IOException {
      InputStream in = classLoaderWrapper.getResourceAsStream(resource, loader);//此时loader为空
      if (in == null) {
          throw new IOException("Could not find resource " + resource);
      }
      return in;
  }
  ```

  - Resources中的classLoaderWrapper成员变量

    ```java
     private static ClassLoaderWrapper classLoaderWrapper = new ClassLoaderWrapper();
    
    public class ClassLoaderWrapper {//只保留构造方法和成员变量
    
      ClassLoader defaultClassLoader;
      ClassLoader systemClassLoader;
    
      ClassLoaderWrapper() {
        try {
          systemClassLoader = ClassLoader.getSystemClassLoader();
        } catch (SecurityException ignored) {
          // AccessControlException on Google App Engine
        }
      }
    }
    ```

  - Resources的getResourceAsStream方法

    ```java
    public InputStream getResourceAsStream(String resource, ClassLoader classLoader) {
        return getResourceAsStream(resource, getClassLoaders(classLoader));//此时classLoader为空，但是经过getClassLoaders方法后得到了5个ClassLoader
    }
    ```

    - getClassLoaders方法

      ```java
      ClassLoader[] getClassLoaders(ClassLoader classLoader) {
        return new ClassLoader[]{//获取所有环境的ClassLoader
            classLoader,
            defaultClassLoader,
            Thread.currentThread().getContextClassLoader(),//Web环境
            getClass().getClassLoader(),
            systemClassLoader};
      }
      ```

    - getResourceAsStream方法

      ```java
      InputStream getResourceAsStream(String resource, ClassLoader[] classLoader) {
          for (ClassLoader cl : classLoader) {//对所有可能的ClassLodaer进行遍历
              if (null != cl) {
      
                  // 通过ClassLoader中的getResourceAsStream方法获取到文件流
                  InputStream returnValue = cl.getResourceAsStream(resource);
      
                  // 如果文件流为空则在路径前添加上/再次尝试一次
                  if (null == returnValue) {
                      returnValue = cl.getResourceAsStream("/" + resource);
                  }
      
                  if (null != returnValue) {
                      return returnValue;//返回文件流
                  }
              }
          }
          return null;
      }
      ```

      - ClassLoader的getResourceAsStream方法

        ```java
        public InputStream getResourceAsStream(String name) {
            URL url = getResource(name);
            try {
                return url != null ? url.openStream() : null;
            } catch (IOException e) {
                return null;
            }
        }
        ```

        