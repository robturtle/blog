---
title: 如何遍历文件系统
date: 2016-12-07 21:45:16
tags:
- code snippet
- OS
- filesystem
- ruby
- python
- java
---

破个例，在这里列出一下遍历文件系统的代码片段。

从 [之前这篇文章](/2016/07/29/about-shell-2/) 我们知道，一个 Unix like 的文件系统就是一个树状结构（准确地说，是个有向无环图）。那么遍历一个文件系统的问题，本质上就是树的遍历而已。在一个  POSIX 系统里，表示树的节点的数据结构被称为 `i-node` ，在 BSD 里也被称为 `v-node` ，每个节点都有一个独一无二的标识，一般称为 `i-number` ，我们通过比较 `i-number` 就可以知道两个文件是否指向同一个位置。一个简化的节点的结构类似这样：

<!-- more -->

```c
struct Node {
  String name;
  Mode mode; // is it a directory? etc.
  Node[] children;
  Node parent;
};
```

### C

而实际的结构，通过 `man lstat` 就可以看到了，是这样的：

```c
struct stat { /* when _DARWIN_FEATURE_64_BIT_INODE is NOT defined */
         dev_t    st_dev;    /* device inode resides on */
         ino_t    st_ino;    /* inode's number */
         mode_t   st_mode;   /* inode protection mode */
         nlink_t  st_nlink;  /* number of hard links to the file */
         uid_t    st_uid;    /* user-id of owner */
         gid_t    st_gid;    /* group-id of owner */
         dev_t    st_rdev;   /* device type, for special file inode */
         struct timespec st_atimespec;  /* time of last access */
         struct timespec st_mtimespec;  /* time of last data modification */
         struct timespec st_ctimespec;  /* time of last file status change */
         off_t    st_size;   /* file size, in bytes */
         quad_t   st_blocks; /* blocks allocated for file */
         u_long   st_blksize;/* optimal file sys I/O ops blocksize */
         u_long   st_flags;  /* user defined flags for file */
         u_long   st_gen;    /* file generation number */
     };
```

那么在系统的最底层，就可以通过 POSIX 规定的函数/宏 (注意这些都不是标准库，在非 POSIX 系统下如 Windows 也不兼容)，来实现文件系统的遍历了：[【原链接】](http://stackoverflow.com/questions/9417957/file-system-tree-traversal)

```c
#include <unistd.h>
#include <stdio.h>
#include <dirent.h>
#include <string.h>
#include <sys/stat.h>

void printdir(char *dir, int depth)
{
    DIR *dp;
    struct dirent *entry;
    struct stat statbuf;
    int spaces = depth*4;

    if((dp = opendir(dir)) == NULL) {
        fprintf(stderr,"cannot open directory: %s\n", dir);
        return;
    }
    chdir(dir);
    while((entry = readdir(dp)) != NULL) {
        lstat(entry->d_name,&statbuf);
        if(S_ISDIR(statbuf.st_mode)) {
            /* Found a directory, but ignore . and .. */
            if(strcmp(".",entry->d_name) == 0 || 
                strcmp("..",entry->d_name) == 0)
                continue;
            printf("%*s%s/\n",spaces,"",entry->d_name);
            /* Recurse at a new indent level */
            printdir(entry->d_name,depth+1);
        }
        else printf("%*s%s\n",spaces,"",entry->d_name);
    }
    chdir("..");
    closedir(dp);
}

/*  Now we move onto the main function.  */

int main(int argc, char* argv[])
{
    char *topdir, pwd[2]=".";
    if (argc != 2)
        topdir=pwd;
    else
        topdir=argv[1];

    printf("Directory scan of %s\n",topdir);
    printdir(topdir,0);
    printf("done.\n");

    return 0;
}
```

### Python

在接下来的代码里，提供一个示例的文件树来对比执行结果：

![](https://www.dropbox.com/s/cv51u8v8zls1rfq/Screenshot%202016-12-07%2022.21.07.png?raw=1)



在比 C 更高级的层面，基本上都提供了抽象的文件系统来统一对 POSIX 和非 POSIX 的操作。在 Python 里就直接内置了 DFS 顺序的函数，返回一个 generator: [【原链接】](http://pythoncentral.io/how-to-traverse-a-directory-tree-in-python-guide-to-os-walk/)

```python
python << EOF
import os
from pprint import pprint
pprint(list(os.walk('.')))
EOF
```

执行结果：

![](https://www.dropbox.com/s/x7hjjdpki09428s/Screenshot%202016-12-07%2022.18.58.png?raw=1)

### Java

Java 里的遍历，基于两个基本方法：`File#isDirectory()` 以及 `File#listFiles()` ：[【原链接】](http://stackoverflow.com/questions/3154488/how-do-i-iterate-through-the-files-in-a-directory-in-java)

```java
public class traverse {
    public static void main(String... args) {
        File[] files = new File(".").listFiles();
        showFiles(files);
    }

    public static void showFiles(File[] files) {
        for (File file : files) {
            if (file.isDirectory()) {
                System.out.println("Directory: " + file.getName());
                showFiles(file.listFiles());
            } else {
                System.out.println("File: " + file.getName());
            }
        }
    }
}
```

执行结果：

![](https://www.dropbox.com/s/u3x5v5ut6i6tjsm/Screenshot%202016-12-07%2022.32.42.png?raw=1)

### Ruby

Ruby 里的基本方法和 Java 基本一致：[【原链接】](http://stackoverflow.com/questions/15697983/directory-traversal-in-ruby)

```ruby
def walk(start)
  Dir.foreach(start) do |x|
    path = File.join(start, x)
    if x == "." or x == ".."
      next
    elsif File.directory?(path)
      puts path + "/" # remove this line if you want; just prints directories
      walk(path)
    else
      puts x
    end
  end
end
```

执行结果：

![](https://www.dropbox.com/s/6njpfqk6lm6w663/Screenshot%202016-12-07%2022.47.33.png?raw=1)
