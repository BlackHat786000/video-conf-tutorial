# Video Conferencing Tutorial

## Simple Video Conferencing Using WebRTC

### Tutorial for Amazon EC2 Ubuntu

**This tutorial is tested with Apache Tomcat 9.0.95 and JDK 17.0.2.**

1. **Allow Inbound Traffic**
   - Allow inbound traffic on ports 8080 and 443.

2. **Build a WAR File**
   - Run the following command to build the WAR file:
     ```bash
     mvn clean package
     ```

3. **Download Apache Tomcat 9.0.95**
   - Use `wget` to download Apache Tomcat:
     ```bash
     wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.95/bin/apache-tomcat-9.0.95.tar.gz
     ```

4. **Extract the Downloaded Tomcat**
   - Extract the downloaded file:
     ```bash
     tar -xvzf apache-tomcat-9.0.95.tar.gz
     ```

5. **Move the Files to /opt/tomcat**
   - Move the extracted files:
     ```bash
     sudo mv apache-tomcat-9.0.95 /opt/tomcat
     ```

6. **Set Appropriate Permissions for Tomcat Files**
   - Change ownership to `ubuntu`:
     ```bash
     sudo chown -R ubuntu:ubuntu /opt/tomcat
     ```

7. **Configure Apache Tomcat**
   - Open and edit the `server.xml` configuration file:
     ```bash
     sudo nano /opt/tomcat/conf/server.xml
     ```

8. **Move the WAR File to Tomcat**
   - Rename the WAR file to `video-conf-tutorial.war` and move it to Tomcat's `webapps` directory:
     ```bash
     sudo mv /home/ubuntu/video-conf-tutorial.war /opt/tomcat/webapps/
     ```

9. **Start Tomcat**
   - **Start Tomcat as root** to bind Tomcat to an unprivileged port like 80 and deploy the WAR file:
     ```bash
     sudo /opt/tomcat/bin/startup.sh
     ```

10. **Check Tomcat Logs**
    - View the logs to ensure Tomcat is running correctly:
      ```bash
      tail -f /opt/tomcat/logs/catalina.out
      ```

11. **Access the Application**
    - Open a web browser and navigate to:
      ```
      http://your-ec2-public-ip:8080/video-conf-tutorial
      ```

### Steps to Obtain a Certificate Using Let's Encrypt

1. **Install Certbot**
   - Install Certbot to obtain the SSL certificate:
     ```bash
     sudo apt install certbot
     ```

2. **Run Tomcat on Port 80**
   - Modify `server.xml` to run Tomcat on port 80 (change from default port 8080):
     ```xml
     <Connector port="80" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443"
                maxParameterCount="1000" />
     ```

3. **Obtain the Certificate**
   - Use Certbot to obtain the certificate:
     ```bash
     sudo certbot certonly --webroot -w /opt/tomcat/webapps/ROOT -d absurvey.giize.com
     ```
   - The certificate will be saved at:
     - Certificate: `/etc/letsencrypt/live/absurvey.giize.com/fullchain.pem`
     - Key: `/etc/letsencrypt/live/absurvey.giize.com/privkey.pem`

### Steps to Redirect Port 8080 to 443 and Configure HTTPS Connector (Port 443)

1. **Edit `server.xml`**
   - Open and modify `server.xml` to configure port redirection and HTTPS:
     ```bash
     sudo nano /opt/tomcat/conf/server.xml
     ```
   - Add the following configuration:
     ```xml
     <Connector port="8080" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="443"
                maxParameterCount="1000" />

     <Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol" 
                maxThreads="150" SSLEnabled="true">
       <SSLHostConfig>
         <Certificate certificateFile="/etc/letsencrypt/live/absurvey.giize.com/cert.pem"
                      certificateKeyFile="/etc/letsencrypt/live/absurvey.giize.com/privkey.pem"
                      certificateChainFile="/etc/letsencrypt/live/absurvey.giize.com/chain.pem" />
       </SSLHostConfig>
     </Connector>
     ```

---

### Notes

- **Shutdown Tomcat:**
  ```bash
  sudo /opt/tomcat/bin/shutdown.sh
