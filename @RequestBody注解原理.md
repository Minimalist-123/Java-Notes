1. @RequestBody
/**
 * Annotation indicating a method parameter should be bound to the body of the web request.
 * The body of the request is passed through an {@link HttpMessageConverter} to resolve the
 * method argument depending on the content type of the request. Optionally, automatic
 * validation can be applied by annotating the argument with {@code @Valid}.
 */
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestBody {
    /**
     * Whether body content is required.
     * <p>Default is {@code true}, leading to an exception thrown in case
     * there is no body content. Switch this to {@code false} if you prefer
     * {@code null} to be passed when the body content is {@code null}.
     * @since 3.2
     */
    boolean required() default true;
}
从源码中可以看到，@RequestBody 用在方法参数上面，用来将请求参数绑定到request body中，通过HttpMessageConverter封装为具体的JavaBean。通俗点讲就是你在一个参数上加上该注解，spring就会将request body中的json/xml对象解析成该参数类型的Javabean对象。
作为RESTful开发中经常用到的注解，研究其原理有利于我们更好地理解并掌握它。
那么spring是如何做到这一点的呢？先来看DispatcherServlet。
作为springMVC处理请求的中央调度器，DispatcherServlet本身是一个servlet，所以我们看doService():

  /**
     * Exposes the DispatcherServlet-specific request attributes and delegates to {@link #doDispatch}
     * for the actual dispatching.
     */
    @Override
    protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
        if (logger.isDebugEnabled()) {
            String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
            logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +
                    " processing " + request.getMethod() + " request for [" + getRequestUri(request) + "]");
        }
        //......
        //......
        try {
            doDispatch(request, response);
        }
        finally {
            //......
        }
    }
重点在doDispatch()方法,该方法先找到会找到合适的handler来处理当前请求:

// Determine handler adapter for the current request.
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
HandlerAdapter是一个接口，具体处理方法在RequestMappingHandlerAdapter类中:

RequestMappingHandlerAdapter.png

用@RequestMapping(包括@GetMapping、@PostMapping等)注释修饰的接口请求会用该类处理。RequestMappingHandlerAdapter实现了AbstractHandlerMethodAdapter,所以mv = ha.handle(processedRequest, response, mappedHandler.getHandler());实际上调用的是AbstractHandlerMethodAdapter类的handle()方法：
@Override
    @Nullable
    public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        return handleInternal(request, response, (HandlerMethod) handler);
    }
这里又调用了handleInternal()方法，RequestMappingHandlerAdapter重写了该方法:
进入该方法，

       if (this.synchronizeOnSession) {
            HttpSession session = request.getSession(false);
            if (session != null) {
                Object mutex = WebUtils.getSessionMutex(session);
                synchronized (mutex) {
                    mav = invokeHandlerMethod(request, response, handlerMethod);
                }
            }
            else {
                // No HttpSession available -> no mutex necessary
                mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            // No synchronization on session demanded at all...
            mav = invokeHandlerMethod(request, response, handlerMethod);
        }
可以看到最终调用的都是invokeHandlerMethod()方法,此方法会处理@RequestMapping修饰的请求

protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
            HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

        ServletWebRequest webRequest = new ServletWebRequest(request, response);
        try {
            WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
            ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

            ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
            //注意,此处往ServletInvocableHandlerMethod中设置了一系列的参数解析器
            if (this.argumentResolvers != null) {
                invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
            }
            if (this.returnValueHandlers != null) {
                invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
            }
            invocableMethod.setDataBinderFactory(binderFactory);
            invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

            ModelAndViewContainer mavContainer = new ModelAndViewContainer();
            mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
            modelFactory.initModel(webRequest, mavContainer, invocableMethod);
            mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

            AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
            asyncWebRequest.setTimeout(this.asyncRequestTimeout);

            WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
            asyncManager.setTaskExecutor(this.taskExecutor);
            asyncManager.setAsyncWebRequest(asyncWebRequest);
            asyncManager.registerCallableInterceptors(this.callableInterceptors);
            asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

            if (asyncManager.hasConcurrentResult()) {
                Object result = asyncManager.getConcurrentResult();
                mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
                asyncManager.clearConcurrentResult();
                if (logger.isDebugEnabled()) {
                    logger.debug("Found concurrent result value [" + result + "]");
                }
                invocableMethod = invocableMethod.wrapConcurrentResult(result);
            }
            //具体请求处理方法
            invocableMethod.invokeAndHandle(webRequest, mavContainer);
            if (asyncManager.isConcurrentHandlingStarted()) {
                return null;
            }

            return getModelAndView(mavContainer, modelFactory, webRequest);
        }
        finally {
            webRequest.requestCompleted();
        }
    }
进入该方法的invocableMethod.invokeAndHandle(webRequest, mavContainer);，来到ServletInvocableHandlerMethod,此类继承了InvocableHandlerMethod，可以处理请求的返回值。invokeAndHandle()方法:

public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
            Object... providedArgs) throws Exception {

        Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
        setResponseStatus(webRequest);

        if (returnValue == null) {
            if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
                mavContainer.setRequestHandled(true);
                return;
            }
        }
        else if (StringUtils.hasText(getResponseStatusReason())) {
            mavContainer.setRequestHandled(true);
            return;
        }

        mavContainer.setRequestHandled(false);
        Assert.state(this.returnValueHandlers != null, "No return value handlers");
        try {
            this.returnValueHandlers.handleReturnValue(
                    returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
        }
        catch (Exception ex) {
            if (logger.isTraceEnabled()) {
                logger.trace(getReturnValueHandlingErrorMessage("Error handling return value", returnValue), ex);
            }
            throw ex;
        }
    }
重点在Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);,通过请求调用并产生返回值。

public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
            Object... providedArgs) throws Exception {

        Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
        if (logger.isTraceEnabled()) {
            logger.trace("Invoking '" + ClassUtils.getQualifiedMethodName(getMethod(), getBeanType()) +
                    "' with arguments " + Arrays.toString(args));
        }
        Object returnValue = doInvoke(args);
        if (logger.isTraceEnabled()) {
            logger.trace("Method [" + ClassUtils.getQualifiedMethodName(getMethod(), getBeanType()) +
                    "] returned [" + returnValue + "]");
        }
        return returnValue;
    }
getMethodArgumentValues()方法的作用是获取方法参数，重点就在这里，

private Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
            Object... providedArgs) throws Exception {

        MethodParameter[] parameters = getMethodParameters();
        //创建一个Object数组用于存放解析后的参数列表
        Object[] args = new Object[parameters.length];
        for (int i = 0; i < parameters.length; i++) {
            MethodParameter parameter = parameters[i];
            parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
            args[i] = resolveProvidedArgument(parameter, providedArgs);
            if (args[i] != null) {
                continue;
            }
            //判断当前的参数解析器是否支持解析该参数
            if (this.argumentResolvers.supportsParameter(parameter)) {
                try {
                    //解析参数
                    args[i] = this.argumentResolvers.resolveArgument(
                            parameter, mavContainer, request, this.dataBinderFactory);
                    continue;
                }
                catch (Exception ex) {
                    if (logger.isDebugEnabled()) {
                        logger.debug(getArgumentResolutionErrorMessage("Failed to resolve", i), ex);
                    }
                    throw ex;
                }
            }
            if (args[i] == null) {
                throw new IllegalStateException("Could not resolve method parameter at index " +
                        parameter.getParameterIndex() + " in " + parameter.getExecutable().toGenericString() +
                        ": " + getArgumentResolutionErrorMessage("No suitable resolver for", i));
            }
        }
        return args;
    }
进入resolveArgument()方法,

public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
            NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
        //找到对应解析器
        HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
        if (resolver == null) {
            throw new IllegalArgumentException("Unknown parameter type [" + parameter.getParameterType().getName() + "]");
        }
        //解析参数
        return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
    }
@RequestBody修饰的参数会使用RequestResponseBodyMethodProcessor解析，

public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
            NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

        parameter = parameter.nestedIfOptional();
        Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());
        String name = Conventions.getVariableNameForParameter(parameter);

        if (binderFactory != null) {
            WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
            if (arg != null) {
                validateIfApplicable(binder, parameter);
                if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
                    throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
                }
            }
            if (mavContainer != null) {
                mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
            }
        }

        return adaptArgumentIfNecessary(arg, parameter);
    }
进入readWithMessageConverters()方法一路顺藤摸瓜，来到AbstractMessageConverterMethodArgumentResolver的readWithMessageConverters(),

Object body = NO_VALUE;
EmptyBodyCheckingHttpInputMessage message;
try {
            message = new EmptyBodyCheckingHttpInputMessage(inputMessage);

            for (HttpMessageConverter<?> converter : this.messageConverters) {
                Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
                GenericHttpMessageConverter<?> genericConverter =
                        (converter instanceof GenericHttpMessageConverter ? (GenericHttpMessageConverter<?>) converter : null);
                if (genericConverter != null ? genericConverter.canRead(targetType, contextClass, contentType) :
                        (targetClass != null && converter.canRead(targetClass, contentType))) {
                    if (logger.isDebugEnabled()) {
                        logger.debug("Read [" + targetType + "] as \"" + contentType + "\" with [" + converter + "]");
                    }
                    if (message.hasBody()) {
                        //解析HttpMessage
                        HttpInputMessage msgToUse =
                                getAdvice().beforeBodyRead(message, parameter, targetType, converterType);
                       //使用相应的转换器
                        body = (genericConverter != null ? genericConverter.read(targetType, contextClass, msgToUse) :
                                ((HttpMessageConverter<T>) converter).read(targetClass, msgToUse));
                        body = getAdvice().afterBodyRead(body, msgToUse, parameter, targetType, converterType);
                    }
                    else {
                        body = getAdvice().handleEmptyBody(null, message, parameter, targetType, converterType);
                    }
                    break;
                }
            }
        }
        catch (IOException ex) {
            throw new HttpMessageNotReadableException("I/O error while reading input message", ex);
        }
可以看到使用文章开头提到的HttpMessageConverter解析参数并返回，而此处的HttpMessageConverter是在RequestMappingHandlerAdapter中设置解析器的时候添加到每个解析器中的。而json格式的数据使用AbstractJackson2HttpMessageConverter进行解析，内部使用jackson进行json数据的解析。

总结
请求由DispatcherServlet处理，找到相应的HandlerAdapter进行处理，RequestMappingHandlerAdapter会处理@RequestMapping注解的请求，设置一系列参数解析器进行解析，如果参数使用@RequestBody注解，则使用RequestResponseBodyMethodProcessor进行解析，此参数解析器用HttpMessageConverter将HttpMessage封装为具体的JavaBean对象，json格式的数据使用AbstractJackson2HttpMessageConverter进行解析，内部使用jackson进行json数据的解析。

作者：疏花
链接：https://www.jianshu.com/p/c1b8315c5a03
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
