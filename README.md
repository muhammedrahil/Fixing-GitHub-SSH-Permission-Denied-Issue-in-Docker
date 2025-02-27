# Fixing GitHub SSH Permission Denied Issue in Docker

## **Issue**

When trying to run `git pull` inside a Docker container, you encounter the following error:

```
The authenticity of host 'github.com (140.82.113.4)' can't be established.
ECDSA key fingerprint is SHA256:p2QAMXNIC1TJYWeIOttrVc98/R1BUFWu3/LiyKgUfQM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,140.82.113.4' (ECDSA) to the list of known hosts.
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

## **Cause**

- The SSH key required to authenticate with GitHub is missing or not recognized inside the Docker container.
- Incorrect file permissions on SSH keys.
- The SSH agent is not forwarding the key inside the container.

## **Solution**

Exit from docker.

### **Step 1: Check if the SSH Key Exists in the Container**

Run the following command inside the Docker container:

```bash
ls -la /root/.ssh
```

If `id_rsa` and `id_rsa.pub` are missing, follow the next steps.



```
ubuntu@ip-172-26-12-54:~$ ls -la ~/.ssh
total 44
drwx------  2 ubuntu ubuntu 4096 Sep 20 10:35 .
drwxr-x--- 14 ubuntu ubuntu 4096 Feb 27 06:49 ..
-rw-------  1 ubuntu ubuntu 1626 Dec  7  2023 authorized_keys
-r--------  1 ubuntu ubuntu   52 Nov 16  2023 config
-r--------  1 ubuntu ubuntu 1679 Nov 16  2023 default
-r--------  1 ubuntu ubuntu 2610 Oct 21  2023 git
-rw-r--r--  1 ubuntu ubuntu  576 Oct 21  2023 git.pub
-rw-------  1 ubuntu ubuntu 5254 Sep 20 10:35 known_hosts
-rw-------  1 ubuntu ubuntu 4418 Sep 20 10:34 known_hosts.old
```

### **Step 2: Copy SSH Key from Host to Container**

Run these commands on your host machine:

```bash
docker cp ~/.ssh/id_rsa <container_id>:/root/.ssh/id_rsa
docker cp ~/.ssh/id_rsa.pub <container_id>:/root/.ssh/id_rsa.pub
docker cp ~/.ssh/known_hosts <container_id>:/root/.ssh/known_hosts
```

> Replace `<container_id>` with your actual running container ID. You can find it using `docker ps`.

in case coming error\


```
Error response from daemon: Could not find the file /root/.ssh in container devpatriosoft
```



```
ubuntu@ip-172-26-12-54:~$ docker cp ~/.ssh/git devpatriosoft:/root/.ssh/id_rsa
Successfully copied 4.61kB to devpatriosoft:/root/.ssh/id_rsa
Error response from daemon: Could not find the file /root/.ssh in container devpatriosoft
```

Go to inside a docker

```
docker exec -it <container_id> bash
```

create a directory 

```
mkdir -p /root/.ssh
chmod 700 /root/.ssh
exit  # Exit the container
```

### **Step 3: Set Correct Permissions Inside the Container**

Access your container:

```bash
docker exec -it <container_id> bash
```

check these files are exists and are not\


```
cd /root/.ssh/
```



```
root@9073f8ae699c:~/.ssh# ls
id_rsa  id_rsa.pub  known_hosts
```

Then run:

```bash
chmod 600 /root/.ssh/id_rsa
chmod 644 /root/.ssh/id_rsa.pub
chmod 644 /root/.ssh/known_hosts
```

### **Step 4: Test SSH Connection**

Inside the container, run:

```bash
ssh -T git@github.com
```

If you see:

```
Hi <your-github-username>! You've successfully authenticated, but GitHub does not provide shell access.
```

Then SSH is working correctly.

### **Step 5: Pull Code from GitHub**

Now, try pulling the repository again:

```bash
git pull origin dev
```

## **Conclusion**

Following these steps should resolve the "Permission denied (publickey)" issue when using Git inside a Docker container.

