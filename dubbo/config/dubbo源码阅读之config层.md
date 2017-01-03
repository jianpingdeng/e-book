#dubbo源码阅读之config层
dubbo配置层有两个主要点：ServiceConfig和ReferenceConfig  
# ServiceConfig #
## 动态端口 ##
第一个映入我眼帘的就是getRandomPort(String protocol)方法，因为在刚开始使用dubbo，项目启动dubbo获取端口的时候经常遇到端口冲突，这让我很闹心，所以这次从这个方法揭开dubbo的面纱。
	
	private static Integer getRandomPort(String protocol) {
        protocol = protocol.toLowerCase();
        if (RANDOM_PORT_MAP.containsKey(protocol)) {
            return RANDOM_PORT_MAP.get(protocol);
        }
        return Integer.MIN_VALUE;
    }
接下来你会看到getRandomPort(String protocol)方法中就是很简单的从一个RANDOM_PORT_MAP中获取，那么他是如何做到动态的呢？莫慌，咱们看一下啊这个RANDOM_PORT_MAP是从哪里装载值的，很快我们发现putRandomPort方法，这个方法给RANDOM_PORT_MAP进行了赋值，不断跟下去，就到了这里：   
  
	public static int getAvailablePort(int port) {
    	if (port <= 0) {
    		return getAvailablePort();
    	}
    	for(int i = port; i < MAX_PORT; i ++) {
    		ServerSocket ss = null;
            try {
                ss = new ServerSocket(i);
                return i;
            } catch (IOException e) {
            	// continue
            } finally {
                if (ss != null) {
                    try {
                        ss.close();
                    } catch (IOException e) {
                    }
                }
            }
    	}
    	return port;
    }  
	/**这里就是动态获取端口的方法**/  
	public static int getAvailablePort() {
        ServerSocket ss = null;
        try {
            ss = new ServerSocket();
            ss.bind(null);
            return ss.getLocalPort();
        } catch (IOException e) {
            return getRandomPort();
        } finally {
            if (ss != null) {
                try {
                    ss.close();
                } catch (IOException e) {
                }
            }
        }
    }
	/**如果发生异常则调用以下方法,该方法返回一个30000 + 10000以内的整数，为什么这样写，我想应该是这段以上的端口很少被占用 **/
    private static final int RND_PORT_START = 30000;
    
    private static final int RND_PORT_RANGE = 10000;
    
    private static final Random RANDOM = new Random(System.currentTimeMillis());

	public static int getRandomPort() {
        return RND_PORT_START + RANDOM.nextInt(RND_PORT_RANGE);
    }
总结，两种方式，一种是通过ServerSocket来获取一个端口，另一种方式就是暴力的直接返回一个整数。

