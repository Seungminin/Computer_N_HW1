# HW1_실습

https://jddng.tistory.com/202

순서

1. 파일을 불러 들어와서 IP주소와 PORT에 대한 정보를 받는다.
2. Socket을 이용하는데 Thread를 이용한 Socket이다. 
    
    Client쪽에서는 계속 정보를 전달하고 Server에서는 Thread를 이용해서 값을 계산
    
3. Message를 전달할 때는 HTTP Protocol을 이용한다. 
    
    ex) Get/ServerHW1/1.1 - Client 
    
    ex) HTTP/1.1 200 OK -Server 
    
    먼저 Client가 Syntax Get을 buffer로 전달을 하면 Server는 앞에 Get을 보고 OK라는 문장을 보낸다. 다음으로 Client가 header부분에 domain대신 계산 수식을 전달한다.(5*2) …등등
    
    서버는 올바른 계산을 하여 Client에게 보내준다
    
    만약 Client가 close라는 말을 하면 connection을 종료시켜준다.
