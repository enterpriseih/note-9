### 将本地项目上传到git

* **在本地初始化一个版本库**

  在项目文件夹右键打开git bash，输入命令git init，将这个文件夹便成git可管理的仓库

<img src="D:%5study%5note%5git%5将本地项目上传到git.assets%5Cimage-20210202234646867.png" alt="image-20210202234646867"  />

* **将文件提交到本地仓库**

  执行git add . 命令将项目文件全部添加到暂存区，执行git commit -m "提交信息" 命令将项目提交到本地仓库

  ![image-20210202235441722](%E5%B0%86%E6%9C%AC%E5%9C%B0%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0git.assets/image-20210202235441722-1612282578396.png)

* **git SSH key配置**

* **创建远程仓库**

  ![image-20210203000018163](%E5%B0%86%E6%9C%AC%E5%9C%B0%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0git.assets/image-20210203000018163-1612282712870.png)

![image-20210203000153882](%E5%B0%86%E6%9C%AC%E5%9C%B0%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0git.assets/image-20210203000153882-1612282722982.png)

* **关联远程仓库**

  执行命令git remote add origin git@github.com:y-time/note.git

  ![image-20210203000755234](%E5%B0%86%E6%9C%AC%E5%9C%B0%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0git.assets/image-20210203000755234-1612282742884.png)

  git@github.com:y-time/note.git 为远程仓库地址

  ![image-20210203000648124](%E5%B0%86%E6%9C%AC%E5%9C%B0%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0git.assets/image-20210203000648124-1612282751079.png)

* **将本地项目推送到远程仓库**

  执行命令 git push -u origin master

  由于新建的远程仓库是空的，所以要加上-u这个参数，等远程仓库里面有了内容之后可以直接执行git push origin master

  ![image-20210203000854850](%E5%B0%86%E6%9C%AC%E5%9C%B0%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0git.assets/image-20210203000854850-1612282775101.png)