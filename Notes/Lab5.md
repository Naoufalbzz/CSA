# Lab 5
- Review the concept of canary tokens and understand how they are different/similar to honeypots
  - Canary Tokens: 
    -  Deployment: Canary tokens are strategically placed in locations where they are not expected to be accessed under normal circumstances, such as within a sensitive file, a dummy database entry, or a network share.
    -  Triggering: When an attacker interacts with the token (e.g., opens a file, clicks a link, or queries a database), it triggers an alert.
    -  Alerting: The triggering of a canary token sends an immediate alert to the system administrator or security team, usually including details about the source of the interaction (e.g., IP address, timestamp).
  - Honeypots: 
    -  Deployment: Honeypots are set up within the network environment to look like legitimate systems, services, or data, but are isolated from the production environment.
    -  Attraction: These decoy systems are made attractive to attackers by appearing vulnerable or containing valuable data.
    -  Interaction: When an attacker engages with a honeypot, all their actions are monitored and logged, providing insights into the attack methods and possibly the identity of the attacker.
    -  Analysis: The data collected from honeypot interactions can be used to improve security measures and understand emerging threats.
## Cowrie
- Why is companyrouter, in this environment, an interesting device to configure with a SSH honeypot?
  - High Exposure: As a router, it likely has significant exposure on the network, making it an attractive target for attackers.
  - Misconception of Value: Attackers might see a router as a valuable target, believing it could give them access to more critical internal systems or networks.
  - Network Segmentation: A honeypot on a router can help monitor traffic that might be aimed at internal network segments, providing insight into potential threats.
- What could be a good argument to NOT configure the router with a honeypot service?
  -  Potential Disruption: If misconfigured, the honeypot could disrupt legitimate network operations or compromise the router's functionality.
  -  Security Risks: If the honeypot is not properly isolated, it could potentially be exploited by attackers to launch attacks on the rest of the network.
  -  Complexity: Adding a honeypot to a router might complicate the router's configuration and management.
-  Change your current SSH configuration in such a way that the SSH server (daemon) is not listening on port 22 anymore but on port 2222.
  -  `sudo nano /etc/ssh/sshd_config` port line naar 2222 (selinux uit) -> `sudo systemctl restart sshd`
  -  `ssh -p 2222 companyrouter`
  -  Docker installeren:
```
sudo dnf update -y
sudo dnf install -y dnf-utils
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
sudo docker --version
```
- `sudo docker run -p 22:2222 cowrie/cowrie:latest` 
- nu is honeypot op port 22
- `ssh companyrouter` -> je gaat niet kunnen inloggen door honeypot
- Attack your router and try to SSH normally. What do you notice?
  -  I just can try many passwords and i can see every attempt in the log file but i cant get in (na docker run commando te doen krijg je log file live)
  -  ssh with port 2222 is possible and lets me ssh normally
-  What credentials work? Do you find credentials that don't work?
  - when doing ssh with root@192.168.100.253 and password vagrant i can get in, vagrant:vagrant doesnt work  
  -   Cowrie is designed to log and trap all login attempts. It may accept commonly used or guessed credentials to simulate a real-world attack scenario
-  Do you get a shell?
   -  Cowrie is designed to provide a fake shell to attackers
-  Are your commands logged? Is the IP address of the SSH client logged? If this is the case, where?
   -   Yes, Cowrie logs all commands executed by the attacker. Logs are typically found in /cowrie/var/log/cowrie/ directory inside the Docker container
-   Can an attacker perform malicious things?
   - Cowrie is designed to simulate an environment where attackers can attempt to perform malicious actions. However, these actions are contained within the honeypot environment and do not affect the host system or network. Cowrie is set up to monitor and log such activities to help in understanding attack techniques.   
- Are the actions, in other words the commands, logged to a file? Which file?
   -  `ssh root@192.168.100.253` om op honeypot te geraken
   - `sudo docker ps -a` -> ` sudo docker logs [id]`
   -  `cowrie.log` for general log entries.
   -  `commands.log` for executed commands.
   -  `login.log` for login attempts.
-  If you are an experienced hacker, how would/can you realize this is not a normal environment?
   - Unusual Response: Commands and responses might feel slow or inconsistent compared to a real system.
   - Limited Functionality: Certain commands or tools might not work as expected or might be restricted.
   - Environment Clues: Files and directories that shouldn’t be accessible might be visible or logs might be in non-standard locations.   
   - Repetitive Patterns: If the environment seems too scripted or predictable, it might be a honeypot.
   - Network Responses: Responses from the server might be abnormal or have specific patterns that indicate a honeypot.

## Critical thinking (security) when using "Docker as a service"
- What are some (at least 2) advantages of running services (for example cowrie but it could be sql server as well) using docker?
   -  Isolation and Consistency: Containers provide a consistent environment across different systems, preventing conflicts between applications.
   -  Portability and Easy Deployment: Docker images are self-contained and can be easily moved between different systems, simplifying deployment.
- What could be a disadvantage? Give at least 1.
   -  Security Concerns: Containers share the host OS kernel, which can expose them to kernel-level exploits affecting other containers or the host.

- Explain what is meant with "Docker uses a client-server architecture."
   -  Docker uses a client-server model where the Docker client communicates with the Docker daemon (server). The Docker client is the command-line interface or GUI through which users interact with Docker to run commands, build images, and manage containers. The Docker daemon is the background service that manages Docker containers, images, networks, and volumes
-  As which user is the docker daemon running by default?
   -   The Docker daemon runs as the `root` user by default.
-   What could be an advantage of running a honeypot inside a virtual machine compared to running it inside a container?
    -  VMs provide better isolation from the host and other systems compared to containers, reducing the risk of attacks spreading.



## Other honeypots
- What type of honeypot is "honeyup"?
  -  Honeyup is a low-interaction honeypot designed to emulate specific services and capture basic attack attempts.
- What is the idea behind "opencanary"?
  - OpenCanary is a canary honeypot that simulates various network services and alerts when they are accessed, helping to detect unauthorized access or attacks.

- Is a HTTP(S) honeypot a good idea? Why or why not?
  -  High Visibility: HTTP(S) services are commonly targeted, so a honeypot can capture a lot of attack traffic.
  -  Insight into Attacks: It provides valuable data on web-based attacks and vulnerabilities.