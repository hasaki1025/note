# SpringMVC源码（三）

- ## DispatchServlet初始化

  - DispatchServlet类图

    ![image-20221101105918770](C:\Users\pcdn\AppData\Roaming\Typora\typora-user-images\image-20221101105918770.png)

  - 调用httpServletBean类的init方法

  ```java
  public final void init() throws ServletException {
  
      // Set bean properties from init parameters.
      PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
      if (!pvs.isEmpty()) {
          try {
              BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
              ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
              bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
              initBeanWrapper(bw);
              bw.setPropertyValues(pvs, true);
          }
          catch (BeansException ex) {
              if (logger.isErrorEnabled()) {
                  logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
              }
              throw ex;
          }
      }
  
      // Let subclasses do whatever initialization they like.
      initServletBean();
  }
  ```

  