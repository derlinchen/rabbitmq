### RabbitMQ工作模式 ###

#### 一、 简单模式 ####

1. 生产者
2. 消费者
3. 队列

 生产者：

	public class Sender {
		// 定义队列名称
	    private final static String QUEUE_NAME = "simple_queue";

	    public static void main(String[] args) throws IOException {
		//创建连接
		Connection connection = ConnectionUtil.getConnection();
		//创建通道
		Channel channel = connection.createChannel();

		//声明队列
		/**
		 * 队列名
		 * 是否持久化
		 *  是否排外  即只允许该channel访问该队列   一般等于true的话用于一个队列只能有一个消费者来消费的场景
		 *  是否自动删除  消费完删除
		 *  其他属性
		 *
		 */
		channel.queueDeclare(QUEUE_NAME, false, false, false, null);

		//消息内容
		/**
		 * 交换机
		 * 队列名
		 * 其他属性  路由
		 * 消息body
		 */
		String message = "错的不是我，是这个世界~";
		channel.basicPublish("", QUEUE_NAME,null,message.getBytes());
		System.out.println("[x]Sent '"+message + "'");

		//最后关闭通关和连接
		channel.close();
		connection.close();
	    }
	}

消费者：

	public class Receiver {
	    private final static String QUEUE_NAME = "simple_queue";
	 
	    public static void main(String[] args) throws IOException, InterruptedException {
	        //获取连接
	        Connection connection = ConnectionUtil.getConnection();
	        //获取通道
	        Channel channel = connection.createChannel();
	 		//声明队列
	        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        channel.basicConsume(QUEUE_NAME, true, consumer);
	 
	        while(true){
	            //该方法会阻塞
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println("[x] Received '"+message+"'");
	        }
	    }
	}


#### 二、 WORK模式 ####

1. 一个生产者
2. 多个消费者
3. 消费者根据资源从同一队列中获取消息
4. 一个消息只能被一个消费者获取

生产者：

	public class Send {
		// 生产者队列
	    private final static String QUEUE_NAME = "test_queue_work";
	
	    public static void main(String[] argv) throws Exception {
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	
	        // 声明队列
	        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	
	        for (int i = 0; i < 100; i++) {
	            // 消息内容
	            String message = "" + i;
	            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
	            System.out.println(" [x] Sent '" + message + "'");
	
	            Thread.sleep(i * 10);
	        }
	
	        channel.close();
	        connection.close();
	    }
	}

消费者1：

	public class Recv {
		// 队列名与生产者保持一致
	    private final static String QUEUE_NAME = "test_queue_work";
	
	    public static void main(String[] argv) throws Exception {
	
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	
	        // 声明队列
	        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	
	        // 同一时刻服务器只会发一条消息给消费者
	        //channel.basicQos(1);
	
	        // 定义队列的消费者
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        // 监听队列，false表示手动返回完成状态，true表示自动
	        channel.basicConsume(QUEUE_NAME, true, consumer);
	
	        // 获取消息
	        while (true) {
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println(" [y] Received '" + message + "'");
	            //休眠
	            Thread.sleep(10);
	            // 返回确认状态，注释掉表示使用自动确认模式
	            //channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	    }
	}

消费者2：

	public class Recv2 {
		// 队列与生产者保持一致
	    private final static String QUEUE_NAME = "test_queue_work";
	
	    public static void main(String[] argv) throws Exception {
	
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	
	        // 声明队列
	        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	
	        // 同一时刻服务器只会发一条消息给消费者
	        //channel.basicQos(1);
	
	        // 定义队列的消费者
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        // 监听队列，false表示手动返回完成状态，true表示自动
	        channel.basicConsume(QUEUE_NAME, true, consumer);
	
	        // 获取消息
	        while (true) {
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println(" [x] Received '" + message + "'");
	            // 休眠1秒
	            Thread.sleep(1000);
	            //下面这行注释掉表示使用自动确认模式
	            //channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	    }
	}

#### 三、订阅模式 ####

1. 1个生产者，多个消费者
2. 每个消费者都有自己的队列
3. 生产者没有将消息发送到队列，而是发送到交换机
4. 每个队列都要绑定到交换机
5. 生产者发送消息，经过交换机推送到所有的队列
6. 一个消息可被多个消费者获取的目的

生产者：

	public class Send {
		// 定义交换机名称
	    private final static String EXCHANGE_NAME = "test_exchange_fanout";
	
	    public static void main(String[] argv) throws Exception {
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	
	        // 声明exchange
	        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
	
	        // 消息内容
	        String message = "Hello World!";
	        channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
	        System.out.println(" [x] Sent '" + message + "'");
	
	        channel.close();
	        connection.close();
	    }
	}

消费者1：

	public class Recv {
		// 定义队列名称
	    private final static String QUEUE_NAME = "test_queue_work1";
		// 定义交换机名称与生产者的交换机名称保持一致
	    private final static String EXCHANGE_NAME = "test_exchange_fanout";
	
	    public static void main(String[] argv) throws Exception {
	
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	
	        // 声明队列
	        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	
	        // 绑定队列到交换机
	        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
	
	        // 同一时刻服务器只会发一条消息给消费者
	        channel.basicQos(1);
	
	        // 定义队列的消费者
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        // 监听队列，手动返回完成
	        channel.basicConsume(QUEUE_NAME, false, consumer);
	
	        // 获取消息
	        while (true) {
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println(" [Recv] Received '" + message + "'");
	            Thread.sleep(10);
	
	            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	    }
	}


消费者2：

	public class Recv2 {
		// 定义第二个队列名称
	    private final static String QUEUE_NAME = "test_queue_work2";
		// 定义交换机名称，与生产者交换机保持一致
	    private final static String EXCHANGE_NAME = "test_exchange_fanout";
	
	    public static void main(String[] argv) throws Exception {
	
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	
	        // 声明队列
	        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	
	        // 绑定队列到交换机
	        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
	
	        // 同一时刻服务器只会发一条消息给消费者
	        channel.basicQos(1);
	
	        // 定义队列的消费者
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        // 监听队列，手动返回完成
	        channel.basicConsume(QUEUE_NAME, false, consumer);
	
	        // 获取消息
	        while (true) {
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println(" [Recv2] Received '" + message + "'");
	            Thread.sleep(10);
	
	            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	    }
	}


#### 四、路由模式 ####

1. 1个生产者，多个消费者
2. 每个消费者都有自己的队列
3. 生产者没有将消息发送到队列，而是发送到交换机
4. 每个队列都要绑定到交换机
5.  交换机接受消息后向指定的队列推送消息

生产者：

	public class Sender {
	    private final static String EXCHANGE_NAME = "exchange_direct";
	    private final static String EXCHANGE_TYPE = "direct";
	 
	    public static void main(String[] args) throws IOException {
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	 
	        channel.exchangeDeclare(EXCHANGE_NAME,EXCHANGE_TYPE);
	 
	        String message = "那一定是蓝色";
	        channel.basicPublish(EXCHANGE_NAME,"key2", null, message.getBytes());
	        System.out.println("[x] Sent '"+message+"'");
	 
	        channel.close();
	        connection.close();
	    }
	}

消费者1：

	public class Receiver1 {
	    private final  static  String QUEUE_NAME = "queue_routing";
	    private final static String EXCHANGE_NAME = "exchange_direct";
	 
	    public static void main(String[] args) throws IOException, InterruptedException {
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	 
	        channel.queueDeclare(QUEUE_NAME, false,false,false,null);
	        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"key");
	 
	        channel.basicQos(1);
	 
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        channel.basicConsume(QUEUE_NAME, false, consumer);
	 
	        while(true){
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println("[x] Received1 "+message);
	            Thread.sleep(10);
	 
	            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	    }
	}

消费者2：

	public class Receiver2 {
	    private final  static  String QUEUE_NAME = "queue_routing2";
	    private final static String EXCHANGE_NAME = "exchange_direct";
	 
	    public static void main(String[] args) throws IOException, InterruptedException {
	        // 获取到连接以及mq通道
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	 
	        channel.queueDeclare(QUEUE_NAME, false,false,false,null);
	        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"key2");
	 
	        channel.basicQos(1);
	 
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        channel.basicConsume(QUEUE_NAME, false, consumer);
	 
	        while(true){
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println("[x] Received2 "+message);
	            Thread.sleep(10);
	 
	            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	 
	 
	    }
	}

#### 五、主题模式 ####

1. 模式与路由模式相同
2. 配置路由键的时候可以配置 *, # 来模糊匹配

生产者：

	public class Sender {
	    private final static String EXCHANGE_NAME = "exchange_topic";
	    private final static String EXCHANGE_TYPE = "topic";
	 
	    public static void main(String[] args) throws IOException {
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	 
	        channel.exchangeDeclare(EXCHANGE_NAME, EXCHANGE_TYPE);
	 
	        //消息内容
	        String message = "如果真爱有颜色";
	        channel.basicPublish(EXCHANGE_NAME,"key.1",null,message.getBytes());
	        System.out.println("[x] Sent '"+message+"'");
	 
	        //关通道 关连接
	        channel.close();
	        connection.close();
	    }
	}

消费者1：

	public class Receiver1 {
	    private final static String QUEUE_NAME = "queue_topic";
	    private final static String EXCHANGE_NAME = "exchange_topic";
	    private final static String EXCHANGE_TYPE = "topic";
	 
	    public static void main(String[] args) throws IOException, InterruptedException {
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	 
	        channel.queueDeclare(QUEUE_NAME, false, false,false, null);
	        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "key.*");
	 
	        channel.basicQos(1);
	 
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        channel.basicConsume(QUEUE_NAME, false, consumer);
	 
	        while(true){
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println("[x] Received1 '"+message + "'");
	            Thread.sleep(10);
	 
	            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	    }
	}

消费者2：

	public class Receiver2 {
	    private final static String QUEUE_NAME = "queue_topic2";
	    private final static String EXCHANGE_NAME = "exchange_topic";
	    private final static String EXCHANGE_TYPE = "topic";
	 
	    public static void main(String[] args) throws IOException, InterruptedException {
	        Connection connection = ConnectionUtil.getConnection();
	        Channel channel = connection.createChannel();
	 
	        channel.queueDeclare(QUEUE_NAME, false, false,false, null);
	        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "*.*");
	 
	        channel.basicQos(1);
	 
	        QueueingConsumer consumer = new QueueingConsumer(channel);
	        channel.basicConsume(QUEUE_NAME, false, consumer);
	 
	        while(true){
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
	            String message = new String(delivery.getBody());
	            System.out.println("[x] Received2 '"+message + "'");
	            Thread.sleep(10);
	 
	            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
	        }
	    }
	}
