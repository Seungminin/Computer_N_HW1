# HW1_실습

# HW1_실습

https://jddng.tistory.com/202

### 해야  하는 작업

1. 파일을 불러 들어와서 IP주소와 PORT에 대한 정보를 받는다.
    
    ```java
    FileReader fr1=new FileReader("C:\\server\\Server_info.txt"); //get IP,PORT from Server_info.txt. 
    			BufferedReader br1 = new BufferedReader(fr1);
    			String str;
    			try {				
    				while((str=br1.readLine())!=null) {
    					System.out.println(str);
    					String[] temp = str.split(" "); //Split space " ", get IP, Port from text file.
    					IP=temp[0];
    					PORT=Integer.parseInt(temp[1]);
    					System.out.println("IP : "+IP+" Port : "+ PORT);
    ```
    
2.  Server는 여러 Client와 연결이 돼야 하는 Thread를 이용한다.
    
    Client쪽에서는 계속 정보를 전달하고 Server에서는 Thread를 이용해서 값을 계산
    
    ```java
    ServerSocket server = new ServerSocket(PORT); //Server Open and wait for clients
    			System.out.println("Server is connected Client...");
    			ExecutorService pool= Executors.newFixedThreadPool(20); //Enable Thread for Multiple Connections
    			while(true) {
    				Socket socket = server.accept(); //Server connect Clients.
    				pool.execute(new Capitalizer(socket)); 
    			}
    .
    .
    .
    .
    .
    Capitalizer implements Runnable{
    		private Socket socket;
    		Capitalizer(Socket socket){
    			this.socket=socket;
    		}
    		@Override 
    		//Use Thread 
    		public void run() {
    			// TODO Auto-generated method stub
    			System.out.println("Connected : "+socket);
    			try {
    ```
    
3. Message를 전달할 때는 HTTP Protocol을 이용한다. 
    
    ex) Get/ServerHW1/1.1 - Client 
    
    ex) HTTP/1.1 200 OK -Server  → 연결 성공
    
    ex) HTTP/1.1 400 -Bad request
    
    ex) HTTP/1.1 404 - Not Found
    
    ex) Close -서버와 클라이언트 연결 끊기.
    
    먼저 Client가 Syntax Get을 buffer로 전달을 하면 Server는 앞에 GET/ServerHW1을 보고 OK라는 문장을 보낸다. 다음으로 Client가 header부분에 domain대신 계산 수식을 전달한다.(5*2) …등등
    
    서버는 올바른 계산을 하여 Client에게 보내준다
    
    만약 Client가 close라는 말을 하면 connection을 종료시켜 준다.
    
4.  Client에서 산술 message를 보내면 Server에서는 protocol을 해석하고 계산을 해서 답을 Client에게 보내준다. 만약 Client에서 보낸 message가 오류가 있으면 Incorrect : 어떤 이유인지 라는 message를 보낸다.
    
    ```java
    //methods for determining numbers
    	private static boolean isNumeric(String str) {
    	    try {
    	        Double.parseDouble(str);
    	        return true;
    	    } catch (NumberFormatException e) {
    	        return false;
    	    }
    	}
    	 // Method to perform the calculation based on the input expression
    	public static String calc(String exp) {
    		StringTokenizer st = new StringTokenizer(exp," ");
    		String[] st2 = exp.split(" ");
    		if(st.countTokens()!=3) { //if integer num is over 3.
    			return "Incorrect : Too many arguments.";
    		}
    		
    		if(st2[1].equals("/")&&st2[2].equals("0")) { //if divide 0.
    			return "Incorrect : divided by zero";
    		}
    		else if(!st2[1].equals("+") && !st2[1].equals("-") && !st2[1].equals("*") && !st2[1].equals("/")) { // if use not calculation(+,-,*,/)
    			return "Incorrect : Type Calculation";
    		}
    		else if(!isNumeric(st2[0]) || !isNumeric(st2[2])){ //if num is not existed. 
    			return "Incorrect : Please type Numbers";
    		}
    		String res="";
    		int op1 = Integer.parseInt(st.nextToken());
    		String opcode = st.nextToken();
    		int op2 =Integer.parseInt(st.nextToken());
    		
    		switch(opcode) {
    			case "+" :
    				res=Integer.toString(op1+op2);
    				break;
    			case "-":
    				res=Integer.toString(op1-op2);
    				break;
    			case "*":
    				res=Integer.toString(op1*op2);
    				break;
    			case "/":
    				res=Integer.toString(op1/op2);
    				break;
    		}
    		return res;
    	}
    ```
    
    1. 수가 3개가 넘어갈 때
    2. 0으로 나누기를 할 때
    3. 연산자를 쓰지 않았을 때
    4. 숫자를 쓰지 않았을 때 
5. Client에서 CLOSE라는 http syntax를 보내면 서버와의 연결은 끊긴다. 하지만 서버는 연결이 끊기지 않는다.
    
    ```java
    //Server
    if(inputMessage2.startsWith("CLOSE")) { //if Client wants to close program.
    				        		out.println("CLOSE");
    				        		break; // Terminate Server and Client. 
    				        	}
    //Client
    String result = in.readLine();
    						if(result.equals("CLOSE")) {
    							break;
    						}
    ```
    

## 구조도

![HW1_diagram.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/247af101-2d9d-4262-901b-593fd7a568b5/264ce651-7cc7-4156-ad9c-999edad80da8/HW1_diagram.png)

## 코드

Server

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.StringTokenizer;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ServerHW1 {
	static Integer PORT; //Port number
	
	//methods for determining numbers
	private static boolean isNumeric(String str) {
	    try {
	        Double.parseDouble(str);
	        return true;
	    } catch (NumberFormatException e) {
	        return false;
	    }
	}
	 // Method to perform the calculation based on the input expression
	public static String calc(String exp) {
		StringTokenizer st = new StringTokenizer(exp," ");
		String[] st2 = exp.split(" ");
		if(st.countTokens()!=3) { //if integer num is over 3.
			return "Incorrect : Too many arguments.";
		}
		
		if(st2[0].equals("DIV")&&st2[2].equals("0")) { //if divide 0.
			return "Incorrect : divided by zero";
		}
		else if(!st2[0].equals("ADD") && !st2[0].equals("SUB") && !st2[0].equals("MUL") && !st2[0].equals("DIV")) { // if use not calculation(+,-,*,/)
			return "Incorrect : Type Calculation again Please UpperBound";
		}
		else if(!isNumeric(st2[1]) || !isNumeric(st2[2])){ //if num is not existed. 
			return "Incorrect : Please type Numbers";
		}
		String res="";
		String opcode = st.nextToken();
		int op1 = Integer.parseInt(st.nextToken());
		int op2 =Integer.parseInt(st.nextToken());
		
		switch(opcode) {
			case "ADD" :
				res=Integer.toString(op1+op2);
				break;
			case "SUB":
				res=Integer.toString(op1-op2);
				break;
			case "MUL":
				res=Integer.toString(op1*op2);
				break;
			case "DIV":
				res=Integer.toString(op1/op2);
				break;
		}
		return res;
	}
	
	public static void main(String[] args) throws Exception{
		try {
			FileReader fr1=new FileReader("C:\\server\\Server_info.txt"); //get IP,PORT from Server_info.txt. 
			BufferedReader br1 = new BufferedReader(fr1);
			String str;
			 if ((str = br1.readLine()) == null) {
		            PORT = 1234;
		            System.out.println("Default Port: " + PORT);
		        }
			 else {
		            String[] temp = str.split(" "); //Split space " ", get IP, Port from text file.
		            PORT = Integer.parseInt(temp[1]);
		            System.out.println("Port: " + PORT);
		        }//Recall the Port number from the File.
			
			ServerSocket server = new ServerSocket(PORT); //Server Open and wait for clients
			System.out.println("Server is connected Client...");
			ExecutorService pool= Executors.newFixedThreadPool(20); //Enable Thread for Multiple Connections
			while(true) {
				Socket socket = server.accept(); //Server connect Clients.
				pool.execute(new Capitalizer(socket)); 
			}
		}catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}
	}
	
	private static class Capitalizer implements Runnable{
		private Socket socket;
		Capitalizer(Socket socket){
			this.socket=socket;
		}
		@Override 
		//Use Thread 
		public void run() {
			// TODO Auto-generated method stub
			System.out.println("Connected : "+socket);
			try {
				BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
				PrintWriter out = new PrintWriter(socket.getOutputStream(),true);
				while(true) {
					   String inputMessage = in.readLine(); 
					   String[] temp = inputMessage.split("/"); // ex)GET/SEdsada/21.12 '/'
					   if (temp[0].equals("GET")&&temp[1].equals("ServerHW1")) { //Protocol get.
				        out.println("HTTP/1.1 200 OK Content-type : text/html");
				        while(true) {				        	
				        	String inputMessage2 = in.readLine();
				        	
				        	if(inputMessage2.startsWith("CLOSE")) { //if Client wants to close program.
				        		out.println("CLOSE");
				        		break; // Terminate Server and Client. 
				        	}
				        	else {				        		
				        		String result = calc(inputMessage2);
				        		if(result.startsWith("Incorrect")) { //Incorrect syntax, and result.
				        			out.println(result.substring(10));
				        			out.flush();
				        		}
				        		else {				        		
				        			out.println(result);
				        			out.flush();
				        		}
				        	}
				        }
					  }
					  else if (temp[0].equals("CLOSE")) { //Protocol close
					        out.println("HTTP/1.1 404 Not Found");
					        break;
					    }
					  else { //Exception get,close..
					        out.println("HTTP/1.1 400 Bad Request ");
					        continue;
					    }
					  break; //in if(inputMessage2.startsWith("CLOSE") -> Terminate Server and Client.
				}
				
			} catch (Exception e) {
				// TODO: handle exception
				e.printStackTrace();
			}finally {
				try {
					if(socket != null) {						
						socket.close();
						}
				} catch (Exception e2) {
					// TODO: handle exception
					e2.printStackTrace();
				}
				
			}
		}
		
	}
}
```

클라이언트
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import java.util.Scanner;

public class ClientHW1 {
	static String IP  = "localhost";// Default IP address //IP adress
	static Integer PORT = 1234; //Port number
	
	Socket client = null;
	public static void main(String[] args) {
		 ClientHW1 clientHW = new ClientHW1(); //  Create an instance of the class
	        clientHW.connectToServer();
	}
	public void connectToServer() {
		try {
			FileReader fr1=new FileReader("C:\\server\\Server_info.txt"); //get IP,PORT from Server_info.txt. 
			BufferedReader br1 = new BufferedReader(fr1);
			String str;
			try {				
				while((str=br1.readLine())!=null) {
					System.out.println(str);
					String[] temp = str.split(" "); //Split space " ", get IP, Port from text file.
					IP=temp[0];
					PORT=Integer.parseInt(temp[1]);
					System.out.println("IP : "+IP+" Port : "+ PORT);
				}//Read File and retrieve the IP address port number required for Socket connection from the file.
				
			} catch (Exception e) {
				// TODO: handle exception
				System.out.println("Default values are used.");
			}finally {
				try {
					 if (br1 != null) {
	                        br1.close();
	                    }
	                    if (fr1 != null) {
	                        fr1.close();
	                    }
				} catch (Exception e2) {
					// TODO: handle exception
					e2.printStackTrace();
				}
			}
			//Ready for Socket in Client.
			client = new Socket(IP,PORT);
			
			Scanner stin = new Scanner(System.in);
			BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
			PrintWriter out = new PrintWriter(client.getOutputStream(),true);
			
			//Using HTTP Message syntax Get
			//ex) GET/ServerHW1/1.1
			System.out.println("Try Connection ServerHW1 Using HTTP Syntax");
			String useInput;
			while(!(useInput = stin.nextLine()).equals("CLOSE")) { //CLOSE syntax is 'Termination' 
				out.println(useInput);
				out.flush();
				
				String InputMessage=in.readLine();
				System.out.println("Server : "+ InputMessage);
				if(InputMessage.equals("Error Message...")) {//Exception GET,CLOSE
					System.out.println("Try Connection ServerHW1 Using HTTP Syntax");
					continue;
				}
				else {										
					while(true) {//Successful use Connection. 
						System.out.println("Please type numbers and Calculations");
						String outMessage = stin.nextLine();
						out.println(outMessage);
						
						String result = in.readLine();
						if(result.equals("CLOSE")) {
							break;
						}
						else {							
							System.out.println("Result is : " + result);	
						}
					}
				}
				break;
			}
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}finally {
			try {
				if(client !=null) {
					client.close();
				}
			} catch (Exception e2) {
				// TODO: handle exception
				e2.printStackTrace();
			}
		}
	}
}
