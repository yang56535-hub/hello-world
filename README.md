# hello-world
此存储库用与练习GitHub流

针对你遇到的 `git@gitlab.com: Permission denied (publickey)` 错误，以下是系统化的排查与解决方案。结合你的 Windows 环境（Git Bash/MINGW64），按优先级从高到低逐步操作：

------

### 🔑 **1. 验证公钥是否正确添加到 GitLab**

- **检查公钥内容一致性**
  运行 `cat ~/.ssh/id_rsa.pub` 或 `cat ~/.ssh/id_ed25519.pub`，确保输出的公钥内容 ​**​完整无换行符​**​，且与 GitLab 账户中 `Settings → SSH Keys` 添加的内容完全一致

  ⚠️ 常见问题：

  - 复制时遗漏首尾字符（如 `ssh-rsa AAA...` 或邮箱注释）。

  - 公钥格式错误（如误复制私钥内容

     

    ```
    id_rsa
    ```

     

    而非

     

    ```
    id_rsa.pub
    ```

    ）

    

- **确认密钥类型支持**
  GitLab 仅支持 RSA（≥2048位）、ECDSA、ED25519 等密钥类型。若使用旧版 RSA-1024 或不安全算法（如 DSA），需重新生成：

  bash

  复制

  ```
  ssh-keygen -t ed25519 -C "your_email@example.com"  # 推荐
  或
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  ```

  生成后重新复制公钥到 GitLab

  

  

------

### 🔒 **2. 启动 SSH 代理并加载私钥**

- **确保代理运行 & 私钥已添加**
  依次执行：

  bash

  复制

  ```
  eval "$(ssh-agent -s)"  # 启动代理（输出应为 "Agent pid XXXX"）
  ssh-add ~/.ssh/id_ed25519  # 加载私钥（若使用 RSA 则替换为 id_rsa）
  ```

  ✅ 验证是否加载成功：`ssh-add -l` 应显示私钥指纹

  

- **代理未生效的解决**

  - Windows 服务问题：重启 Git Bash 或执行 `exec ssh-agent bash` 重新挂载代理。

  - 私钥路径错误：若私钥不在默认路径，需用

     

    ```
    ssh-add /path/to/your_private_key
    ```

     

    指定

    

------

### 📂 **3. 检查文件与目录权限**

- 

  关键权限设置

  

  bash

  复制

  ```
  chmod 700 ~/.ssh      # .ssh 目录权限
  chmod 600 ~/.ssh/id_* # 所有私钥文件权限
  ```

  ⚠️ 权限不当（如

   

  ```
  755
  ```

  ）会导致 SSH 拒绝使用私钥

  

------

### ⚙️ **4. 配置 SSH 配置文件（多密钥必备）**

若存在多个密钥（如同时用 GitHub/GitLab），需创建 `~/.ssh/config` 文件明确指定：

bash

复制

```
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519_gitlab  # 替换为你的私钥路径
  IdentitiesOnly yes                     # 强制仅用指定密钥
```

💡 保存后测试：`ssh -T git@gitlab.com`





------

### 🌐 **5. 验证远程仓库地址 & 网络**

- 

  确认 Git 远程地址为 SSH 格式

  

  bash

  复制

  ```
  git remote -v  # 查看地址，应为 git@gitlab.com:username/repo.git
  ```

  若显示 HTTPS 地址（

  ```
  https://gitlab.com/...
  ```

  ），需修改：

  bash

  复制

  ```
  git remote set-url origin git@gitlab.com:username/repo.git
  ```[1,4](@ref)  
  ```

- 

  检查网络与防火墙

  

  - 测试连接：`ping gitlab.com` 是否通。

  - 企业环境：可能需配置代理或放行端口

     

    ```
    22
    ```

    
