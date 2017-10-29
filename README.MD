# SpringBoot Stomp Demo

STOMP is the Simple (or Streaming) Text Orientated Messaging Protocol. http://stomp.github.io/

STOMP ����ԭ����WebSocket���ڰ�װ��һ�㡣������websocket��stomp������һ�顣

# ԭ��websocket

ԭ��socket��Ҫ�� `@ServerEndpoint` �������� ʹ�� `@OnOpen`, `@OnClose`, `@OnMessage` , `@OnError` �ȼ����¼����ɣ�

## ����websocket

```
@Configuration
public class BaseWebSocketConfig {
  @Bean
  public ServerEndpointExporter serverEndpointExporter() {
    return new ServerEndpointExporter();
  }
}
```

## ��������

ʹ�� `@ServerEndpoint` ��������

ÿһ�����ӻ��´���һ��ʵ����������ȫ���԰�Session��Ϊ���Է��������档

���bean��������Ӧ�á���prototype��������ע��������beanû�гɹ���

```Java
@ServerEndpoint(value = "/basews")
@Component
public class BaseWebSocket {

  public BaseWebSocket() {
    System.out.println("\n-----new BaseWebSocket-----\n");
  }

  // FIXME �޷�ע��bean ?
  // @Autowired
  // SessionService sessionService;

  @OnOpen
  public void onOpen(Session session) {
    SessionService.newSession(session);
  }

  @OnClose
  public void onClose(Session session) {
    SessionService.closeSession(session);
  }

  @OnMessage
  public void onMessage(String message, Session session) {
    System.out.println(session.getId() + " > OnMessage: " + message);
    SessionService.sendMessage(session, "this is a message from server!");
  }

  @OnError
  public void onError(Session session, Throwable throwable) {
    System.out.println(session.getId() + " > OnError: " + throwable.getMessage());

    throwable.printStackTrace();

    SessionService.closeSession(session);
  }

}
```

## Session����

�������˸��������д���Session��

```Java
public class SessionService {

  private static AtomicInteger count = new AtomicInteger();

  private static Map<String, Session> sessionMap = new ConcurrentHashMap<>();

  public static void newSession(Session session) {
    sessionMap.put(session.getId(), session);

    System.out.println("new session, now count=" + count.incrementAndGet());
  }

  public static void closeSession(Session session) {
    sessionMap.remove(session.getId());

    System.out.println("close session, now count=" + count.decrementAndGet());
  }

  /**
   * ������Ϣ���ͻ���
   * 
   * @param session
   * @param message
   */
  @SneakyThrows
  public static void sendMessage(Session session, String message) {
    session.getBasicRemote().sendText(message);
    // session.getAsyncRemote().sendText(message);
  }

}
```

## ����

��chrome����console�������

```
var ws = new WebSocket("ws://127.0.0.1:8080/basews");
ws.onmessage = function(msg){console.log("���ܵ���Ϣ", msg)};
ws.send("hello");

// �ڿ�һ������
var ws2 = new WebSocket("ws://127.0.0.1:8080/basews");
```

![chrome](/pictures/chromeconsole.png)

![chrome](/pictures/base-sb1.png)


# junit �ʼ�

ʹ������˿ڣ��Ϳ�������2��SpringBoot�ˡ�ʹ�� `@LocalServerPort` ע�뵱ǰ�ķ���Ķ˿ڡ�

```Java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class GreetingIntegrationTests {

  @LocalServerPort
  private int port;

}
```