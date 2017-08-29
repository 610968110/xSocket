# xSocket
xSocket    


 xSocket使用说明：所有方法回调都运行在主线程,支持一个服务端连接多个客户端,客户端支持断掉重连（包括断网重连），服务端断网不会通知客户端重新连接，但会重启服务，客户端需要手动重新连接。
 
 
 1.需要权限（必须）    
 
      uses-permission android:name="android.permission.INTERNET"  
      
      uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" 
      
      uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" 
      
      uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" 
      
      uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"  
      
      
      
      
      
 2.若使用服务端，需要注册服务（可选） 
 
      service android:name="com.lbx.socket.service.SocketService" 
      
      
      
      
      
      
 3.初始化（必须）  
 
      SocketManager manager = SocketManager.init(this)    
      
      //当客户端成功连接到服务端，服务端向客户端发送的连接成功的消息    
      
      //null或""为不发送，默认不发送                
      
      .setServerConnectMsg("connect success")    
      
      //是否打印xSocket原生log，第二个参数为log的tag，默认不打印                
      
      .setPrintLog(true, "xyss"); 
      
      
   3.1初始化服务端(不用服务端时可省略)（可选）
   
   
      ServiceManager.init(manager); 
      
      
   3.2初始化客户端(不用客户端时可省略)（可选） 
   
      ClientManager.init(manager); 
      
      
      
      
      
 4.释放，这里其实是监听网络变化广播的注销，也会把一些对象置空，所以确保不再用xSocket时再调用（必须） 
 
      SocketManager.release(this);  
      
      
      
      
      
 5.服务端用法（可选）  
 
 
   5.1打开服务  
   
      //上下文，监听端口，回调 
      
      ServiceManager.getInstance().startService(this, 8081, new ServiceConnectCallback() {     
      
      //绑定服务时调用   
      
      @Override 
      
      public void onServiceConnected(SocketService socketService) {}      
            
            //服务时调用           
            
            @Override            
            
            public void onServiceDisconnected(ComponentName name) {
            
            }      
            
            //客户端连接上时调用   
            
            @Override           
            
            public void onClientConnected(Socket client) {                
            
            super.onClientConnected(client);            
            
            }      
            
            //客户端断开时调用（与客户端没断开时就关闭服务也会回调此方法，这时client参数为null，所有客户端全部掉线）            
            
            @Override     
            
            public void onClientDisconnected(Socket client) {                
            
            super.onClientDisconnected(client);            
            
            }      
            
            //收到客户端消息时调用，进行再主线程            
            
            @Override            
            
            public void onServiceReceiver(String msg) {                
            
            Log.e(TAG, "onServiceReceiver = " + msg);           
            
            }        
            
            });    
            
            
            5.2获取客户端列表，默认按连接时间排序    
            
            /**   *class Client   
            
            *   
            
            *public Socket socket;   
            
            *public PrintWriter printWriter;   
            
            *public BufferedReader bufferedReader;   
            
            *public long connectTime  连接时间,以手机时间为准   
            
            */  
            
            List<Client> list = ServiceManager.getInstance().getClientsList();    
            
            
            5.3向客户端发送消息    
            
            //连接一个客户端时  
            
            ServiceManager.getInstance().sendToClient("123");    
            
            //连接多个客户端时  
            
            ServiceManager.getInstance().sendToClient("123", socket);
            
            //发给某个客户端          
            
            ServiceManager.getInstance().sendToClient("123", socket1, socket2);
            
            //发给多个客户端          
            
            Socket[] clients = new Socket[]{socket1, socket2};        
            
            ServiceManager.getInstance().sendToClient("123", clients);//发给多个客户端    
            
            ServiceManager.getInstance().sendToAllClient("123");//发给全部客户端    
            
            
            5.4关闭（解绑）服务  ServiceManager.getInstance().closeService(this);  
            
        6.客户端用法（可选）   
            
            6.1连接socket    
            
            //上下文，服务端ip，端口，回调   默认socket断掉自动连接 
            
            ClientManager.getInstance().connectToService(this, "172.16.0.109", 8081,new ClientConnectCallback() {         
            
            //连接到服务端时调用                   
            
            @Override                    
            
            public void connectSuccess(Client client) {                        
            
            Log.e(TAG, "connectSuccess = " + client);                    
            
            }          
            
            //连接服务端失败时调用                   
            
            @Override                   
            
            public void connectErr(String err) {                        
            
            Log.e(TAG, "connectErr = " + err);                    
            
            }          
            
            //收到服务端发送的信息时调用，运行在主线程                    
            
            @Override                    
            
            public void onClientReceiver(String msg) {                        
            
            Log.e(TAG, "onClientReceiver = " + msg);                    
            
            }          
            
            //与服务端断开连接时调用     
            
            //autoDisconnected自动断开（断网、连接失败、服务端停止运行/崩溃，等因素）                    
            
            //activeDisconnected 手动断开                    
            
            @Override                    
            
            public void onDisconnected(String msg) {                        
            
            super.onDisconnected(msg);                        
            
            Log.e(TAG, "onDisconnected  " + msg);                    
            
            }                
            
            });       
            
            6.2连接服务端的其他方式     
            
            //第三个参数选择客户端断掉后是否重连，默认为重连   
            
            ClientConfig clientConfig = new ClientConfig("ip", 8080, true);   
            
            ClientManager.getInstance().connectToService(this,clientConfig,new ServiceConnectCallback() {            
            
            ...            
            
            }         
