---
title: "unity游戏开发-MYSQL"
date: 2024-03-19 09:58:44
categories: "unity游戏开发"
tags: ["unity", "MYSQL"]
summary: "unity游戏开发之与MYSQL建立远程数据库"
---

### MySQL配置

1. 先安装一个数据库，我使用的是 `8.0.35` 版本的，版本没什么大的要求。也可以安装一个可视的 `UI` 界面，例如 `workbench`，方便做一些操作
2. 在数据库中添加表格，然后将某一个 `user` 的 `localhost` 设置为 `%`，就可以实现其他设备的登录了

### unity端配置

1. 在 `Assets` 文件目录下新建一个文件夹 `Plugins` 用来存放数据库需要的一些文件 （必须是这个名称，不能更改，不然 unity 不能识别出来）
2. 可以不用下载 `MySQL Connnector/NET`，直接在 `vs`中安装 `mysql` 包 项目→管理 `NuGet` 程序包，直接在浏览中搜索 `MySQL`，安装第一个，之后它会自己把依赖装好
3. 然后，最最重要的一步，在vs的解决方案资源管理器的项目的引用之下，找到 `mysql.Data` 这个项目，查看它所在的路径，在文件资源管理器中找到它，把它拖到第一步所建好的文件夹中，然后进入unity它会报错，查看错误信息，是需要一些文件，然后再在项目引用之下找到 `Google.Protobuf` , `K4os.Compression.LZ4` , `K4os.Compression.LZ4.Streams` , `K4os.Hash.xxHash` , `BouncyCastle.Cryptography` , `System.IO.Pipelines` ,  `System.IO` , `System.Runtime.CompilerServices.Unsafe`，但是我的里面是没有 `ZstdSharp` 的，需要像第二步一样自己安装一下，然后拖到 `unity` 的 `Plugins`，需要注意版本问题，一般直接使用vs中对应程序集的路径就可以
    
  ![vsSet.png](./vsSet.png)
  
4. 也是极其重要的一步，在unity的安装目录之下的 `Editor/Data/MonoBleedingEdge/lib/mono/unityjit-win32` 中，找到以下四个文件
    - `I18N.dll`
    - `I18N.CJK.dll`
    - `I18N.MidEast.dll`
    - `I18N.West.dll`
    
    ![unitySet1.png](./unitySet1.png)
    
5. 把 `unity` 的 `File` → `Build Settings` → `Player Settings` → `Api Compatibility Level` 更改为 `.NET Framewok`
    
    ![unitySet.png](./unitySet.png)
    
6. 然后就可以实现在**局域网之中**连接到这个数据库了
7. 附上代码
    
    ```csharp
        public InputField username;
        public InputField password;
        public GameObject LOG;
        public Canvas canvas;
        // 登录
        string sqlSer = "server=49a17g0230.zicp.fun;port = 21514;database = dragondata;user = root;password = Beloved@25177";
        public void signin()
        {
            MySqlConnection conn = new MySqlConnection(sqlSer);
            try
            {
                conn.Open();
                UnityEngine.Debug.Log("------链接成功------");
    
                string sqlQuary = "select * from userlib where account = @paral1 and password = @paral2";
                MySqlCommand comd = new MySqlCommand(sqlQuary, conn);
                comd.Parameters.AddWithValue("paral1", username.text);
                comd.Parameters.AddWithValue("paral2", password.text);
    
                MySqlDataReader reader = comd.ExecuteReader();
                if (reader.Read())
                {
                    UnityEngine.Debug.Log("------用户存在，登录成功！------");
                    SceneManager.LoadScene(1);
                }
                else
                {
                    LOG.GetComponent<showlogcontent>().logmsg = "登录失败，用户不存在";
                    GameObject log = GameObject.Instantiate(LOG, transform.position, Quaternion.Euler(0, 0, 0));
                    log.transform.SetParent(canvas.transform, false);
                }
            }
            catch (System.Exception e)
            {
                UnityEngine.Debug.Log(e.Message);
            }
            finally
            {
                conn.Close();
            }
        }
        // 注册
        public void signup()
        {
            MySqlConnection conn = new MySqlConnection(sqlSer);
            try
            {
                conn.Open();
                // UnityEngine.Debug.Log("-----连接成功！------");
    
                string sqlQuary = "select * from userlib where account = @paral1 and password = @paral2";
                MySqlCommand comd = new MySqlCommand(sqlQuary, conn);
                comd.Parameters.AddWithValue("paral1", username.text);
                comd.Parameters.AddWithValue("paral2", password.text);
    
                MySqlDataReader reader = comd.ExecuteReader();
                if (reader.Read())
                    UnityEngine.Debug.Log("-----用户名已存在，请重新输！------");
                else
                {
                    Insert_User(username.text, password.text);
                    UnityEngine.Debug.Log("------注册成功，请进行登入------");
                }
            }
            catch (System.Exception e)
            {
                UnityEngine.Debug.Log(e.Message);
            }
            finally
            {
                conn.Close();
            }
        }
        private void Insert_User(string username, string password)
        {
            MySqlConnection conn = new MySqlConnection(sqlSer);
    
            try
            {
                conn.Open();
                string signupdata = System.DateTime.Now.ToString("G");
                string sqlInsert = "insert into userlib(account,password,signupdata) values('" + username + "','" + password + "','" + signupdata + "')";
                MySqlCommand comd2 = new MySqlCommand(sqlInsert, conn);
                int resule = comd2.ExecuteNonQuery();
            }
            catch (System.Exception e)
            {
                UnityEngine.Debug.Log(e.Message);
            }
            finally
            {
                conn.Close();
            }
        }
    ```
    

### 外网连接配置

可以使用局域网以外的网络连接上服务器，需要下载**花生壳**

1. 下载好之后，在花生壳中新增映射
    
    ![HSK.png](./HSK.png)
    
    - 映射类型选择 `TCP`
    - `TCP` 选择普通 `TCP` 就行
    - 不使用模板
    - 外网域名好像随便一个就行，我直接使用的软甲给的提示
    - 端口选择动态的，因为固定的要钱
    - 内网主机就是本机的IP地址，直接在中断输入 `ipconfig`，其中的 `IPv4` 地址就是
    - 内网端口就是 `mysql` 所使用的 `3306` 端口
2. 映射完成之后可以看到
    
    ![HSK1.png](./HSK1.png)
    
    其中最重要的就是访问地址了，可以在 `mysql` 中试着连接一下，其中的 `hostname`  就是外网域名，port就是映射之后的端口
    
    ![mysqltest.png](./mysqltest.png)
    
3. 好像需要开启本电脑的3306端口的远程连接
    - 进入应用 高级安全 `windowsDefender` 防火墙
    - 点击入站规则->新建规则
    - 配置端口， `mysql` 使用的是 `tcp` 协议，使用的是3306端口
4. 在 `unity` 中创建连接
`string sqlSer = "server=49a17g0230.zicp.fun;port = 21514;database = database;user = root;password = password";`
其中的server就是访问地址的域名，也就是外网域名，冒号后面的就是端口，也就是最终映射出的端口
5. 最后就可以实现数据库的远程连接了，并且不需要在同一个局域网