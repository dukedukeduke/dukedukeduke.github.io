---
layout: post
title:  "golang 文件操作"
date:   2019-04-09 14:27:02 +0800
comments: true
tags:
- golang
- file
---

#### 1.判断文件（夹）是否存在

```
if _, err = os.Stat(dirPath); os.IsNotExist(err){
		fmt.Println("文件夹不存在：",dirPath)
		if err = os.MkdirAll(dirPath, 0777); err != nil{
			fmt.Printf(err.Error())
			return err
		}
		fmt.Println("创建文件夹成功：", dirPath)
	}
```

#### 2.创建文件夹（递归）

```
if _, err = os.Stat(dirPath); os.IsNotExist(err){
		fmt.Println("文件夹不存在：",dirPath)
		if err = os.MkdirAll(dirPath, 0777); err != nil{
			fmt.Printf(err.Error())
			return err
		}
		fmt.Println("创建文件夹成功：", dirPath)
	}
```

#### 3.写文件

```
if fileDownload, err = os.OpenFile(filePath,os.O_WRONLY | os.O_CREATE| os.O_APPEND,0777); err != nil{
    fmt.Println("打开文件错误：", filePath)
    }else{
        if _, err = fileDownload.Write(body); err != nil{
        fmt.Println("写入文件错误：", filePath)
        }else{
        fmt.Println("写入文件成功：", filePath)
        }
    }
```

#### 4.合并文件路径

```
dirPath = path.Join(currentDir, "data", love0001.platform, love0001.mode,version)
```

#### 5.读取文件

```
file, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

scanner := bufio.NewScanner(file)
for scanner.Scan() {
    fmt.Println(scanner.Text())
}

if err := scanner.Err(); err != nil {
    log.Fatal(err)
}
```
