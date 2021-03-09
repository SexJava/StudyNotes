# Mysql高级

1. ### 采用Linux+Docker，部署MySQL（省略Docker安装）

   1. 查询MySQL镜像

      ```shell
      docker search mysql
      ```

   2. 拉取（获取）MySQL镜像（latest为最新版本，这里演示版本为mysql5.7）

      ```shell
      docker pull mysql:latest
      ```

   3. 查看镜像是否安装成功

      ```shell
      docker images
      ```

      ![image-20200623223136612](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623223136612.png)

   4. 运行MySQL容器

      ```shell
      docker run -itd --name mysql-test-p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
      ```

      参数说明：

      - 

      - -p 3306:3306：映射容器服务的3306端口到宿主机的3306端口，外部访问可以直接通过**宿主机ip(Linux服务器Ip地址):3306**访问到MySQL服务
      - MYSQL_ROOT_PASSWORD=123456 ：设置MySQL服务root用户的密码

   5. 启动成功，运行**docker ps**命令查看，运行中的容器

      ```shell
      docker ps
      ```

      ![image-20200623224123870](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623224123870.png)

   6. 进入MySQL容器

      ```shell
      docker exec -it mysql-test bash
      ```

      ![image-20200623224258381](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623224258381.png)

   7. 测试MySQL服务，并退出

      ```shell
      mysql -u root -p
      ```

      ![](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623224522336.png)

      

   8. 退出MySQL容器

      ```shell
      exit
      ```

      ![image-20200623224608303](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623224608303.png)

   9. 停止运行MySQL容器

      ```
      docker stop mysql-test
      ```

      ![image-20200623224726161](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623224726161.png)

   10. 再次启动之前创建好的MySQL容器

       ```shell
       docker start mysql-test
       ```

       ![image-20200623224835485](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623224835485.png)

   11. 检查所有存在的容器，包括启动和停止状态的

       ```shell
       docker ps -a
       ```

       ![image-20200623225039602](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623225039602.png)

   12. 删除存在容器（必须先使用**docker stop**停止容器）

       ```shell
       docker rm mysql-test
       ```

       ![image-20200623225818993](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200623225818993.png)

   13. 删除所有停止的容器

       ```shell
       docker rm $(docker ps -a -q)
       ```

       