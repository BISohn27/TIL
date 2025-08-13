# Reactor 패턴과 이벤트 루프

- 동시에 여러 종류의 이벤트를 처리하기 위한 디자인 패턴
- 리액터 패턴의 흐름
  1. 관리하는 리소스의 이벤트가 발생할 때까지 대기
  2. 이벤트 발생 시 해당 이벤트를 처리할 수 있는 핸들러(handler)에게 이벤트를 디스패치(dispatch)
- 이벤트를 실제 처리 가능한 핸들러로 연결하는 형태라 **이벤트 핸들링** 패턴이라고도 부름



## Reactor

- 무한 반복문을 실행하여 이벤트가 발생할 때까지 대기
- 이벤트가 발생하면 해당 이벤트를 처리할 수 있는 핸들러에게 디스패치
- 무한 반복(loop)하며, 이벤트를 디스패치 하는 방식 때문에 **이벤트 루프**라 불림

- 리액터의 구현

  1. Selector을 생성해서 ServerSocketChannel을 등록

     ```java
     selector = Selector.open();
     serverSocketChannel = ServerSocketChannel.open();
     serverSocketChannel.socket().bind(new InetSocketAddress(port));
     serverSocketChannel.configureBlocking(false);
     SelectionKey selectionKey = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
     ```

  2. 소켓 채널을 등록한 키에 해당 채널에서 발생하는 이벤트를 처리할 핸들러 등록

     ```java
     selectionKey.attach(new AcceptHandler(selector, serverSocketChannel));
     ```

  3. 무한 루프를 돌며 `selector.select()`에서 이벤트가 발생할 때까지 대기

     ```java
     public void run() {
         try {
             while (true) {
                 selector.select();
                 Set<SelectionKey> selected = selector.selectedKeys();
                 for (SelectionKey selectionKey : selected) {
                     dispatch(selectionKey);
                 }
                 selected.clear();
             }
         } catch (IOException ex) {
             ex.printStackTrace();
         }
     }
     ```

  4. 이벤트가 발생하면, 해당 채널의 이벤트를 처리하는 핸들러(비즈니스 로직)로 이벤트 처리 위임

     ```java
     void dispatch(SelectionKey selectionKey) {
         Handler handler = (Handler) selectionKey.attachment();
         handler.handle();
     }
     ```

  5. 핸들러에서 클라이언트로부터의 TCP 연결을 받아서 실제 클라이언트와 데이터 송수신하여 처리하는 핸들러로 위임 (셀렉터에 등록된 이벤트는 `ACCEPT`)

     ```java
     class AcceptHandler implements Handler {
         final Selector selector;
         final ServerSocketChannel serverSocketChannel;
       
         AcceptHandler(Selector selector, ServerSocketChannel serverSocketChannel) {
             this.selector = selector;
             this.serverSocketChannel = serverSocketChannel;
         }
       
         @Override
         public void handle() {
             try {
                 final SocketChannel socketChannel = serverSocketChannel.accept();
                 if (socketChannel != null) {
                     new EchoHandler(selector, socketChannel);
                 }
             } catch (IOException ex) {
                 ex.printStackTrace();
             }
         }
     }
     ```

  6. TCP 연결 소켓을 셀렉터에 등록, 채널은 `read`와 `write` 양방향이 다 가능하기 때문에 두 이벤트를 번갈아 가면서 데이터를 처리

     ```java
     public class EchoHandler implements Handler {
         static final int READING = 0, SENDING = 1;
       
         final SocketChannel socketChannel;
         final SelectionKey selectionKey;
         final ByteBuffer buffer = ByteBuffer.allocate(256);
         int state = READING;
       
         EchoHandler(Selector selector, SocketChannel socketChannel) throws IOException {
             this.socketChannel = socketChannel;
             this.socketChannel.configureBlocking(false);
             // Attach a handler to handle when an event occurs in SocketChannel.
             selectionKey = this.socketChannel.register(selector, SelectionKey.OP_READ);
             selectionKey.attach(this);
             selector.wakeup();
         }
       
         @Override
         public void handle() {
             try {
                 if (state == READING) {
                     read();
                 } else if (state == SENDING) {
                     send();
                 }
             } catch (IOException ex) {
                 ex.printStackTrace();
             }
         }
       
         void read() throws IOException {
             int readCount = socketChannel.read(buffer);
             if (readCount > 0) {
                 buffer.flip();
             }
             selectionKey.interestOps(SelectionKey.OP_WRITE);
             state = SENDING;
         }
       
         void send() throws IOException {
             socketChannel.write(buffer);
             buffer.clear();
             selectionKey.interestOps(SelectionKey.OP_READ);
             state = READING;
         }
     }
     ```

     1. 클라이언트와 연결하기 위해 소켓 채널을 셀렉터에 등록
     2. 셀렉터가 대기 중일 수 있기 때문에 `selector.wakeup()`을 호출하여 셀렉터를 즉시 깨워 소켓 `READ` 이벤트를 등록
     3. 클라이언트로와 TCP 연결되면, 클라이언트로부터 데이터를 우선 읽음
     4. 데이터를 읽으면, 쓰기 모드로 전환해서 쓰기 이벤트를 받음
     5. 쓰기 이벤트가 준비되면 데이터를 쓰고 읽기 모드로 전환



## 이벤트 루프는 절대 블록하면 안된다

- 이벤트 루프의 동작 방식
  - 이벤트 발생 대기
  - 이벤트가 발생하면 해당 이벤트를 처리하는 핸들러로 디스패치
  - 핸들러에서 이벤트 처리
- 이벤트 루프 스레드는 이벤트 발생 대기, 발생 이벤트 디스패치를 반복하는 것
-  지연이 발생할 수 있는 로직은 이벤트 루프 스레드를 블록킹 시켜 이벤트 처리를 지연시킴
- 스레드 블록을 유발할 수 있는 로직은 별도의 스레드에서 처리하고, 이벤트 루프 스레드는 이벤트 루핑 작업을 할 수 있도록 하는 것이 중요
- 핸들러에 블록을 유발하는 로직이 없더라도, CPU 사용량이 많아 처리 시간이 오래 걸릴 경우, 작업 범위를 분할해서 처리해야 함