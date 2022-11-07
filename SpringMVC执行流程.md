# SpringMVC执行流程

- 发起请求

  由DispatchServlet接受请求

- 调试

  - 进入HttpServlet的service方法

    ```java
    public void service(ServletRequest req, ServletResponse res)
        throws ServletException, IOException {
    
        HttpServletRequest  request;
        HttpServletResponse response;
    
        try {
            request = (HttpServletRequest) req;
            response = (HttpServletResponse) res;
        } catch (ClassCastException e) {
            throw new ServletException(lStrings.getString("http.non_http"));
        }
        service(request, response);
    }
    ```

  - 调用子类FrameworkServlet中的service方法

    ```java
    protected void service(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    
        HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
        if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
            processRequest(request, response);
        }
        else {
            super.service(request, response);
        }
    }
    ```

  - 再次调用父类service方法

    ```java
    protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
    
        String method = req.getMethod();
    
        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                } catch (IllegalArgumentException iae) {
                    // Invalid date header - proceed as if none was set
                    ifModifiedSince = -1;
                }
                if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }
    
        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);
    
        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
    
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);
    
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
    
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
    
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
    
        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //
    
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
    
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
    ```

  - 调用子类doGet方法(FrameworkServlet)

    ```java
    @Override
    protected final void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    
        processRequest(request, response);
    }
    ```

  - processRequest方法（FrameworkServlet）

    ```java
    protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    			throws ServletException, IOException {
    
    		long startTime = System.currentTimeMillis();
    		Throwable failureCause = null;
    
    		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();//LocaleContext用于获取Loacl类（jdk），local类用于代表当地的位置（没什么作用，只是个标志）
    		LocaleContext localeContext = buildLocaleContext(request);//获取request中的local对象并创建LocaleContext
    
    		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();//获取request中的参数（该Servlet中一些请求的公共属性，存储在ThreadLoacl中）
    		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);//使用ServletRequestAttributes包装
    
    		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);//异步请求管理器
    		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());
    
    		initContextHolders(request, localeContext, requestAttributes);
    
    		try {
    			doService(request, response);
    		}
    		catch (ServletException | IOException ex) {
    			failureCause = ex;
    			throw ex;
    		}
    		catch (Throwable ex) {
    			failureCause = ex;
    			throw new NestedServletException("Request processing failed", ex);
    		}
    
    		finally {
    			resetContextHolders(request, previousLocaleContext, previousAttributes);
    			if (requestAttributes != null) {
    				requestAttributes.requestCompleted();
    			}
    			logResult(request, response, failureCause, asyncManager);
    			publishRequestHandledEvent(request, response, startTime, failureCause);
    		}
    	}
    ```

  - 调用DispatchServlet的doService方法

    ```java
    ```

    