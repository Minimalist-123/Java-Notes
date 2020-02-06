目录

服务限流
需求
算法
通过限制单位时间段内调用量来限流
通过限制系统的并发调用程度来限流
漏桶算法
令牌桶算法
代码
限流设计
环境配置
配置文件
限流服务
切面拦截
测试
测试环境
测试结果
总结
服务限流
需求
1、针对单机的服务流量进行控制，避免突发大流量造成服务异常。2、对业务无侵入。

算法
现在主流的几种限流方式：

通过限制单位时间段内调用量来限流
通过限制系统的并发调用程度来限流
使用漏桶（Leaky Bucket）算法来进行限流
（使用令牌桶（Token Bucket）算法来进行限流
通过限制单位时间段内调用量来限流
通过限制某个服务的单位时间内的调用量来进行限流。我们需要做的就是通过一个计数器统计单位时间段某个服务的访问量，如果超过了我们设定的阈值，
则该单位时间段内则不允许继续访问，或者把接下来的请求放入队列中等待到下一个单位时间段继续访问。

优点：实现简单，阈值可动态配置。
缺点：若单位时间内前一小段时间内就被大流量消耗完，则将导致该时间段内剩余的时间都拒绝服务。该现象为：“突刺消耗”。
通过限制系统的并发调用程度来限流
通过并发限制来限流，我们通过严格限制某服务的并发访问程度，其实也就限制了该服务单位时间段内的访问量，
比如限制服务的并发访问数是100，而服务处理的平均耗时是10毫秒，该服务每秒能提供( 1000 / 10 ) * 100 = 10,000 次。

优点：有更严格的限制边界，适合连接数、线程数的一个限制。
缺点：对服务来说，并发阈值调优困难，难以准确判定服务阈值设置多少合适。一般采用Semaphore实现，但Semaphore没有提供重设信号量的方法，所以阈值动态配置也是问题。
漏桶算法
请求流量以不确定速率申请资源，程序处理以恒定的速率进行，就是漏桶算法的基本原理。


漏斗有一个进水口 和 一个出水口，出水口以一定速率出水，并且有一个最大出水速率：

在漏斗中没有水的时候
如果进水速率小于等于最大出水速率，那么，出水速率等于进水速率，此时，不会积水
如果进水速率大于最大出水速率，那么，漏斗以最大速率出水，此时，多余的水会积在漏斗中
在漏斗中有水的时候
出水口以最大速率出水
如果漏斗未满，且有进水的话，那么这些水会积在漏斗中
如果漏斗已满，且有进水的话，那么这些水会溢出到漏斗之外。

优点：不管突然流量有多大，漏桶都保证了流量的常速率输出。
缺点：漏桶的出水速度是恒定的，那么意味着如果瞬时大流量的话，将有大部分请求被丢弃掉。
令牌桶算法
对于很多应用场景来说，除了要求能够限制数据的平均传输速率外，还要求允许某种程度的突发传输。这时候漏桶算法可能就不合适了，令牌桶算法更为适合。

令牌桶算法的原理是系统以恒定的速率产生令牌，然后把令牌放到令牌桶中，令牌桶有一个容量，当令牌桶满了的时候，再向其中放令牌，
那么多余的令牌会被丢弃；当想要处理一个请求的时候，需要从令牌桶中取出一个令牌，如果此时令牌桶中没有令牌，那么则拒绝该请求。



优点：令牌桶算法能够在限制调用的平均速率的同时还允许某种程度的突发调用。guava RateLimiter就是基于令牌桶算法实现，所以代码实现简易。可动态配置令牌生成速率。
缺点：
基于以上四种算法的介绍，令牌桶不仅能够限制调用的平均速率同时还允许一定程度的突发调用，不会导致突发调用大量请求被丢弃，更加灵活，且代码实现简易。综上：建议选择令牌桶算法实现限流。

代码
限流设计

通过切面拦截请求，判断限流是否开启，若开启则进行令牌的获取，获取成功则执行业务，否则丢弃该请求。
令牌获取：

无配置超时时间，直接获取结果
配置超时时间，在该段时间内不断尝试获取令牌。
环境配置
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
     <version>2.1.1.RELEASE</version>
</dependency>
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
      <version>2.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>18.0</version>
</dependency>
配置文件
ratelimit.properties
# 限流模块的配置
# 是否开启限流
ratelimit.doRateLimit=false
# 配置超时时间（配置将等待获取，不配置将直接返回）,单位毫秒
ratelimit.waitTimeout=20
# 服务限流保护，服务每秒允许的TPS（需评估单个服务所允许的最大TPS）
ratelimit.permitsPerSecond=200
RateLimitConfig
/**
 * 限流配置信息
 * 
 */
@Data
@Component
@ConfigurationProperties(prefix = "ratelimit")
@PropertySource(value = "classpath:ratelimit.properties")
public class RateLimitConfig implements Serializable {
    
    private static final long serialVersionUID = 1L;
    private boolean           doRateLimit      = false;
    private long              waitTimeout;
    private long              permitsPerSecond;
}
限流服务
/**
 * 限流服务接口
 * 
 */
public interface IRateLimitService {
    /**
     * 尝试获取许可证，获取1个，立即返回非阻塞
     * 
     * @return
     */
    boolean tryAcquire();
    /**
     * 尝试获取多个许可证，立即返回非阻塞
     * 
     * @param permits
     * @return
     */
    boolean tryAcquire(int permits);
    /**
     * 阻塞获取许可证，获取1个，若超过timeout未获取到许可证，则返回false
     * 
     * @param timeout
     * @return
     */
    boolean acquire(long timeout);
    /**
     * 阻塞获取多个许可证，若超过timeout未获取到许可证，则返回false
     * 
     * @param permits
     * @param timeout
     * @return
     */
    boolean acquire(int permits, long timeout);
}
/**
 * Guava RateLimiter的限流实现
 * 
 */
@Service
public class RateLimitServiceImpl implements IRateLimitService {
    private RateLimitConfig config;
    private RateLimiter rateLimiter;
    @Autowired
    public RateLimitServiceImpl(RateLimitConfig config) {
        this.config = config;
        this.rateLimiter = RateLimiter.create(this.config.getPermitsPerSecond());
    }
    @Override
    public boolean tryAcquire() {
        return this.tryAcquire(1);
    }
    @Override
    public boolean tryAcquire(int permits) {
        return rateLimiter.tryAcquire(permits);
    }
    @Override
    public boolean acquire(long timeout) {
        return this.acquire(1, timeout);
    }
    @Override
    public boolean acquire(int permits, long timeout) {
        long start = System.currentTimeMillis();
        for (;;) {
            boolean tryAcquire = rateLimiter.tryAcquire(permits);
            if (tryAcquire) {
                return true;
            }
            long end = System.currentTimeMillis();
            if ((end - start) >= timeout) {
                return false;
            }
        }
    }
}
切面拦截
/**
 * 限流切面
 * 
 */
@Aspect
@Order(-1)
@Component
public class RateLimitAspect {
    private static final Logger LOG = LoggerFactory.getLogger(RateLimitAspect.class);
    private RateLimitConfig config;
    private IRateLimitService rateLimitService;
    @Autowired
    public RateLimitAspect(RateLimitConfig config, IRateLimitService rateLimitService) {
        this.config = config;
        this.rateLimitService = rateLimitService;
    }
    @Pointcut("execution(public * xxx.xxx.*Controller.*(..))")
    public void executionMethod() {}
    @Around(value = "executionMethod()")
    public Object doRateLimit(ProceedingJoinPoint pjp) throws Throwable {
        if (LOG.isDebugEnabled()) {
            LOG.debug("进入限流处理切面!");
        }
        Object result = null;
        // 判断是否限流
        try {
            if (config.isDoRateLimit()) {
                // 开启限流
                boolean acquireResult = false;
                // 1.查看是否配置超时时间
                if (config.getWaitTimeout() == 0L) {
                    // 2.获取令牌
                    acquireResult = rateLimitService.tryAcquire();
                } else {
                    // 2.获取令牌，超时时间内获取令牌
                    acquireResult = rateLimitService.acquire(config.getWaitTimeout());
                }
                if (acquireResult) {
                    // 3.成功获取令牌，放行
                    result = pjp.proceed();
                } else {
                    // 3.失败获取令牌，返回错误码 429 => to many requests
                    result = ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS).build();
                }
            } else {
                // 无开启限流，直接放行
                result = pjp.proceed();
            }
        } catch (Throwable e) {
            throw e;
        }
        if (LOG.isDebugEnabled()) {
            LOG.debug("限流处理切面结束!");
        }
        return result;
    }
}
测试
测试环境
Jmeter测试
请求数据：{"name":"张三","age":12}
响应结果：{"name":"张三","age":12}
业务处理时间：100ms
设置TPS：1500
测试结果
Transaction per Second(TPS)


Hits per Second(每秒请求数)


响应时间


聚合报告
成功请求：


错误请求：


全部请求：


总结
在每秒2万多的请求下，TPS依旧稳定在1500。
采用切面的方式，应用无感知。
原文链接：http://www.cnblogs.com/chenzuyibao/p/11425139.html
