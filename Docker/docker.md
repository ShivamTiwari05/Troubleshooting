****Issue-1****
Problem Statement:  we were trying to run the application on docker, my one application running on port 8081 is running fine but other  
applications running on 8080 and 9090 is having health check issues due to which my application fails after running for few seconds leading to restart of my container.....  

**Solution**:  
1) Checking the ec2 instance container restarting phase:
    a) Check Restarting Containers:
   ```
   docker ps -a
   ```

   b) Check Logs for the Container

   ```
   docker logs <container_id>
  ```
  Look for:

    i) Stack trace
    ii) Crash errors
    iii) Port binding failure
    iv) Health check failures
    v) OOM (out of memory)

  c) Inspect Container for Exit Code
  ```
  docker inspect <container_id> | grep -i exitcode
  ```

  d) check docker events
  ```
  docker events
  ```
4) Checking the logs:  no as such errors
5) Between 2 ports 8080 and 8081 the application 8081 was running so NO network issue:  
  checklists:  
  | Check                          | Command                            |  
  | ------------------------------ | ---------------------------------- |  
  | a. Container port exposed?        | `docker ps`                        |  
  | b. Container DNS works?           | `ping google.com` inside container |  
  | c. Internet access works?         | `curl 8.8.8.8` or `ping 8.8.8.8`   |  
  | d. Listening ports?               | `netstat -tuln` inside container   |  
  | e. Docker network?                | docker network ls and `docker network inspect <network>` |  
  | f. Host port reachable?           | `curl localhost:<port>` on EC2     |  
  | g. Security group allows traffic? | Check in AWS Console  if ports are allowed or not             |  
    h. Firewall or IPTables Rules:
      ```
      sudo iptables -L -n -v
      sudo systemctl status firewalld
      ```

    i. Test DNS Resolution from Container  
       From Inside container:  
        ```
        cat /etc/resolv.conf
        ping google.com
        ```  

      If DNS doesn’t resolve, check /etc/docker/daemon.json on host:  

          {
            "dns": ["8.8.8.8", "8.8.4.4"]
          }

        Then restart Docker:
          ```
          sudo systemctl restart docker
          ```

      j. Check Docker Bridge Interface
          ```
          ifconfig docker0
          ```
          If missing, Docker's default networking is broken. Restart Docker:
          ```
          sudo systemctl restart docker
          ```


      k. 


6) Checking the ports are open or not in inbound rules of security groups of instance: already mentioned above
7) checking the live logs of container using---> watch docker ps
8) checked the resource usage and the resource of EC2 : here we found ythe issue thus: Changing the resource from t2.large to m5.2xlarge



