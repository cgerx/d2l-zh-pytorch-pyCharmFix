# d2l-zh-pytorch-pyCharmFix
《动手学深度学习V2》中文课件（pytorch实现），针对pyCharm的修复版。解决课件在pyCharm中打开代码块无法高亮和无代码提示的问题。



## 源教程地址： https://zh.d2l.ai/

***


最近在学习沐神的《动手学深度学习》第二版，从官网下载的课件导入到pyCharm后，发现打开笔记后代码块无法高亮，并且编写代码无代码提示，而在同一个工程中使用pycharm创建的ipynb文件没有上述问题。针对于我这个强迫症患者来说简直无法忍受。  

使用vim对比了下两个文件的差距，发现主要体现在最外层的metadata这个标签：  
```yml 
# 官网jupyter生成的笔记
{
  ....
  
   "metadata": {
    "language_info": {
     "name": "python"
    }
   },
   "nbformat": 4,
   "nbformat_minor": 4
  }
}

```

```yml 
# 使用pycharm生成的笔记
{
  ....
  
  "metadata": {
    "kernelspec": {
     "display_name": "Python 3",
     "language": "python",
     "name": "python3"
    },
    "language_info": {
     "codemirror_mode": {
      "name": "ipython",
      "version": 2
     },
     "file_extension": ".py",
     "mimetype": "text/x-python",
     "name": "python",
     "nbconvert_exporter": "python",
     "pygments_lexer": "ipython2",
     "version": "2.7.6"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 0
}

```


拿了一个文件试了下，将原笔记本的最外层metadata标签，替换为pycharm笔记本的metadata标签，再使用pycharm打开，发现上述问题得到了完美解决。


***


剩下的就很简单了，写了一个main方法，读取工作目录中所有的ipynb格式的文件，去除掉最后一个metadata标签之后的内容，然后拼接上正确的内容，最后写回原来的文件。


因为我主业是java工程师，所以顺手使用的java做的实现。因为每个人环境可能不同，如果代码中的笔记你下载后无法使用，也可以仿照我的方法自行替换。



```java
package com.xconfig.d2l;

import org.apache.commons.io.IOUtils;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;

/**
 * Title:
 * Description:
 * Project: D2l4Pycharm
 * Author: cg
 * Create Time:2022/5/3 12:45
 */
public class D2l4Pycharm {

    
    // d2l的目录，替换为你电脑上的路径
    public static final String WORK_SPACE = "/Users/lixiao/PycharmProjects/d2l";

    
    // 替换标识
    public static final String SUBSTR_FLAG = "metadata";
    
    
    // 最后一个metadata标识，后面需要追加的内容。你可以替换为你自己pycharm生成的内容，注意是从metadata后面的双引号开始的
    public static final String APPEND_BLOCK = "\":{\n" +
            "  \"kernelspec\": {\n" +
            "   \"display_name\": \"Python 3\",\n" +
            "   \"language\": \"python\",\n" +
            "   \"name\": \"python3\"\n" +
            "  },\n" +
            "  \"language_info\": {\n" +
            "   \"codemirror_mode\": {\n" +
            "    \"name\": \"ipython\",\n" +
            "    \"version\": 2\n" +
            "   },\n" +
            "   \"file_extension\": \".py\",\n" +
            "   \"mimetype\": \"text/x-python\",\n" +
            "   \"name\": \"python\",\n" +
            "   \"nbconvert_exporter\": \"python\",\n" +
            "   \"pygments_lexer\": \"ipython2\",\n" +
            "   \"version\": \"2.7.6\"\n" +
            "  }\n" +
            " },\n" +
            " \"nbformat\": 4,\n" +
            " \"nbformat_minor\": 0\n" +
            "}";
    
    public static void main(String[] args) {
        File workspace = new File(WORK_SPACE);
        File[] subDirs = workspace.listFiles();
        Arrays.stream(subDirs).forEach(D2l4Pycharm::processDir);
    }
    
    
    
    private static void processDir(File dir){
        Arrays.stream(dir.listFiles()).filter(File::isFile)
                .filter(file -> file.getName().endsWith("ipynb"))
                .forEach(D2l4Pycharm::processFile);
    }
    
    private static void processFile(File file) {
        String content;
        try (FileInputStream fis = new FileInputStream(file)){
            content = IOUtils.toString(fis, "UTF-8");
            
            content = content.substring(0,content.lastIndexOf(SUBSTR_FLAG) + SUBSTR_FLAG.length());
            
            content += APPEND_BLOCK;
        } catch (IOException ex){
            throw new RuntimeException(ex);
        }
        
        
        try (FileOutputStream fos = new FileOutputStream(file)){
            IOUtils.write(content.getBytes(StandardCharsets.UTF_8), fos);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}



```



替换完成后效果：

<img width="1209" alt="image" src="https://user-images.githubusercontent.com/30364539/166409105-38d2e21e-f8eb-4e33-b695-ac67380b6230.png">


    
