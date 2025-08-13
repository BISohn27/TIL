# Java I/O 멀티플렉싱

## Java NIO(New Input Ouput)

- 채널과 버퍼
  - 서버에서 클라이언트와 데이터를 주고 받을 때 **channel**을 통해 **ByteBuffer**를 이용해 읽고 씀
  - Channel은 양방향이 가능 (기존의 Stream은 InputStream과 OutputStream 단방향)
- 논블록킹 I/O
  - 스레드가 버퍼로 데이터를 읽어달라고 채널에 요청
  - 채널이 버퍼에 데이터를 채워 넣는 동안 스레드는 다른 작업 수행
  - 채널이 버퍼에 데이터를 채워 넣고 나면 스레드는 해당 버퍼를 이용해 데이터 처리
  - 데이터를 채널로 보내는 경우도 논블록킹으로 처리 가능

### 셀렉터

- 여러 개의 채널에서 이벤트를 모니터링할 수 있는 셀렉터 포함
- 한 개의 스레드로 여러 채널을 모니터링 가능

### 버퍼

- 버퍼에 데이터 쓰기
- 버퍼의 `flip()` 메서드 호출
- 버퍼에서 데이터 읽기
- 버퍼의 `clear()` 혹은 `compact()`메서드 호출
- 양방향 채널에서 `flip()`메서드를 호출해 쓰기 모드에서 읽기 모드로 전환
- 모든 데이터를 읽은 후 버퍼를 지우고 다시 쓸 준비를 해야 하고, `clear()`메서드를 호출해서 전체 버퍼를 지움



### 셀렉터

![img](https://vos.line-scdn.net/landpress-content-v2_1761/1672025602251.png?updatedAt=1672025604000)

- 셀럭터에 이벤트를 감시할 여러 개의 채널을 등록

- 등록된 채널 중 한 개 이상의 채널에서 이벤트 준비가 완료될 때까지 대기

- 메서드가 반환되면, 스레드는 채널의 준비 완료된 이벤트를 처리

- 하나의 스레드가 여러 채널을 관리(여러 소켓 연결을 관리)

- 셀렉터 생성 및 채널 등록

  ```java
  Selector selector = Selector.open();
  
  ServerSocketChannel channel = ServerSocketChannel.open();
  channel.bind(new InetSocketAddress("localhost", 8080));
  channel.configureBlocking(false);
  SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
  ```

- 셀렉터에 등록할 수 있는 이벤트

  - `SelectionKey.OP_CONNECT`
  - `SelectionKey.OP_ACCEPT`
  - `SelectionKey.OP_READ`
  - `SelectionKey.OP_WRITE`

- SelectionKey

  - Interest set

    - 셀렉터에 등록된 채널이 원하는 이벤트 집합

    ```java
    int interestSet = selectionKey.interestOps();
     
    boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
    boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
    boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
    boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
    ```

  - Ready set

    - 셀렉터에 등록된 채널에서 바로 처리할 수 있도록 준비된 이벤트 집합

    ```java
    int readySet = SelectionKey.readyOps();
    
    selectionKey.isAcceptable();
    selectionKey.isConnectable();
    selectionKey.isReadable();
    selectionKey.isWritable();
    ```

    - Interest set과 동일하게 비트 연산자로 확인

  - Channel & Selector

    ```java
    Channel channel  = selectionKey.channel();
    Selector selector = selectionKey.selector();
    ```

  - Attached object (optional)

    ```java
    selectionKey.attach(object);
    
    Object attachedObj = selectionKey.attachment();
    ```

    - 채널에서 사용할 버퍼나 상태 등의 추가 데이터를 쉽게 관리하기 위해 사용
    - attachment는 하나만 저장 가능
    - 클라이언트 별 버퍼를 사용하는 등

- 셀렉터를 이용한 채널 선택

  - `select()`: 채널에서 등록된 이벤트가 준비될 때까지 블록
  
  - `select(long timeout)`: timeout까지만 블록
  
  - `selectNow()`: select와 달리 준비된 채널이 없어도 블록되지 않음
  
  - `selectedKeys()`: 이벤트가 준비된 채널 집합을 받음
  
    1. `Set<SelectionKey> selectedKeys = selector.selectedKeys();`
  
    2. 이벤트가 준비된 채널을 담은 키를 반복하며, 이벤트 처리
  
       ```java
       Set<SelectionKey> selectedKeys = selector.selectedKeys();
        
       Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
        
       while(keyIterator.hasNext()) {
            
           SelectionKey key = keyIterator.next();
        
           if(key.isAcceptable()) {
               // a connection was accepted by a ServerSocketChannel.
           } else if (key.isConnectable()) {
               // a connection was established with a remote server.
           } else if (key.isReadable()) {
               // a channel is ready for reading
           } else if (key.isWritable()) {
               // a channel is ready for writing
           }
        
           keyIterator.remove();
       }
       ```



### ServerSocketChannel

- OS의 TCP 포트에 바인딩 되어 클라이언트 연결을 리스닝
- 클라이언트 요청이 들어오면 `accept()`에서 실제 클라이언트와 TCP 연결된 **SocketChannel**을 반환
- 직접 클라이언트로부터 데이터 수신하거나 클라이언트에 데이터를 송신 안함
- 블록킹 모드와 논블록킹 모드 존재
  - 논블록킹 모드에서는 실제 연결과 관계 없이 **SocketChannel**이 반환되고, 만약 연결이 되지 않으면 **null** 반환



### SocketChannel

- 실제 클라이언트와 TCP 연결된 채널
- 실제로 클라이언트와 데이터를 주고 받는 주체
- 블록킹과 논블록킹 모드 존재
  - 논블록킹 모드일 경우 `connect()` 메서드를 호출 시 실제 연결이 완료되기 전 반환되어 연결 확인 필요
  - `read()`의 경우에도 논블록킹 모드에서는 데이터를 읽지 않고 반환 가능하므로, 반환값(읽은 데이터의 바이트 수) 확인 필요