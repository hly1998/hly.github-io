---
layout: post
title: Java爬虫入门（二）
description: "Java爬虫入门"
modified: 2018-9-17
tags: [sample post]
category: java爬虫
---

环境：JDK 10.0.2

IDE： Eclipse

### 简介
在上一篇文章中介绍了爬虫的一些准备工作，那么现在开始编写爬虫代码

### Page类
Page类用来保存网页的一些信息，比如分别以字节形式和字符串形式存储的网页内容、网页的Document对象、字符编码、url等信息，代码如下：
```java
    public class Page {
        private byte[] content ;  //以byte形式存储的内容      
        private Document doc  ;//网页的Document对象
        private String charset ;//字符集编码
        private String url ;//url
```
建立*Page*类的构造函数：
```java
    public Page(byte[] content , String url){
        this.content = content ;
        this.url = url ;
    }
```
因为在*Page*类中成员变量都是私有变量，所以为了获得这些变量，需要写一些get形式的成员函数：
```java
    public String getCharset(){return charset;}
    public String getUrl(){return url ;}
    public byte[] getContent(){ return content ;}
```
除了这几个外，比较关键的是获取*Document*对象，要想获取正确的内容，就要知道正确的编码，在上一节已经说明了可以借助*UniversalDetector*类中的方法来获取，所以先定一个获取编码的函数getEncoding()：
```java
    public String getEncoding(byte[] bytes){
        String DEFAULT_ENCODING = "UTF-8";
        UniversalDetector detector = new UniversalDetector(null);
        detector.handleData(bytes, 0, bytes.length);
        detector.dataEnd();
        String encoding = detector.getDetectedCharset();
        detector.reset();
        if (encoding == null) {
            encoding = DEFAULT_ENCODING;
        }
        return encoding;
    }
```
然后就可以编写getDoc()函数了
```java
public Document getDoc(){
    if (doc != null) {
        return doc;
    }else{
        if(charset==null){
            charset = getEncoding(content);
        }
        try{
            doc = Jsoup.parse(new String(content, charset, url);
            return doc;
        }catch(Exception e){
            e.printStackTrace();
            return null;
        }
    }
}
```
*Page*类最后是这样的:
```java
    import java.io.UnsupportedEncodingException;
    import org.jsoup.Jsoup;
    import org.jsoup.nodes.Document;

    public class Page {
        private byte[] content ;
        private Document doc  ;
        private String charset ;
        private String url ;

        public Page(byte[] content , String url){
            this.content = content ;
            this.url = url ;
        }

        public String getCharset(){return charset;}
        public String getUrl(){return url ;}
        public byte[] getContent(){ return content ;}
        public Document getDoc(){
            if (doc != null) {
                return doc;
            }else{
                if(charset==null){
                    charset = getEncoding(content);
                }   
                try{
                    doc = Jsoup.parse(new String(content, charset, url);
                    return doc;
                    }catch(Exception e){
                    e.printStackTrace();
                    return null;
                }
            }
        }
    }
```

### Links类
*Links*类用来保存各种链接，比如已经访问过的网页链接、未访问过的网址链接、图片链接、各种文件链接等，当然不介意的话不必写这个类，直接把存放各种链接的Set集合放到主类中也未尝不可，但是这样不够优雅QAQ，如果以后不断增加新的集合，程序就会显得十分臃肿，也不好维护。这个类很好写，首先也是定义一下私有变量：
```java
    private static Set visitedUrlSet = new HashSet();   //访问过的链接地址
    private static LinkedList unVisitedUrl = new LinkedList();  //未访问过的链接地址
    private static LinkedList unVisitedImgUrl = new LinkedList();   //未访问过的图片链接
```
然后是一些很简单的函数定义：
```java
    //获取以访问链接的地址
    public static int getVisitedUrlNum() {
        return visitedUrlSet.size();
    }
    //把地址添加到已访问地址集合
    public static void addVisitedUrlSet(String url) {
        visitedUrlSet.add(url);
    }
    //把地址加入到未访问的地址队列
    public static void addUnvisitedUrlQueue(String url) {
        if (url != null && !url.trim().equals("")  && !visitedUrlSet.contains(url)  && !        unVisitedUrlQueue.contains(url)){
            unVisitedUrlQueue.add(url);
        }
    }
    //获取未访问地址队列的首个链接
    public static Object removeHeadOfUnVisitedUrlQueue() {
        return unVisitedUrlQueue.removeFirst();
    }
    //查看未访问地址是否为空
    public static boolean unVisitedUrlQueueIsEmpty() {
        return unVisitedUrlQueue.isEmpty();
    }
```

### RequestAndHandle类
这个类用来向网页发送请求和处理得到的结果，在上一节已经说明了使用apache.commons.httpclient包来发送请求和处理得到的结果。所以直接给出代码：
```java
    import org.apache.commons.httpclient.DefaultHttpMethodRetryHandler;
    import org.apache.commons.httpclient.HttpClient;
    import org.apache.commons.httpclient.HttpException;
    import org.apache.commons.httpclient.HttpStatus;
    import org.apache.commons.httpclient.methods.GetMethod;
    import org.apache.commons.httpclient.params.HttpMethodParams;

    import java.io.IOException;

    public class RequestAndResponse {
        public static Page sendRequstAndGetResponse(String url) {
            Page page = null;
            HttpClient httpClient = new HttpClient();        // 生成 HttpClinet 对象
            httpClient.getHttpConnectionManager().getParams().setConnectionTimeout(5000);  // 设置连接超时 5s
            GetMethod getMethod = new GetMethod(url);
            getMethod.getParams().setParameter(HttpMethodParams.SO_TIMEOUT, 5000);        // 设置请求超时 5s
            getMethod.getParams().setParameter(HttpMethodParams.RETRY_HANDLER, new DefaultHttpMethodRetryHandler());        // 设置请求重试处理
            try {
                int statusCode = httpClient.executeMethod(getMethod);
            if (statusCode != HttpStatus.SC_OK) {
                System.err.println("Method failed: " + getMethod.getStatusLine());
            }
            byte[] responseBody = getMethod.getResponseBody();// 读取为字节 数组
            page = new Page(responseBody,url); //封装成为页面
            } catch (HttpException e) {// 处理异常
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                getMethod.releaseConnection();        // 释放连接
            }
        return page;
    }
}

```
### Spider类
首先来写一下保存网页的函数savePage()：
```java
    public static void save(Page page,String path) {
        String fileName = page.getUrl(); //把链接名作为文件名
        String filePath = path+“/”+fileName; //存放的地址
        byte[] data = page.getContent(); //获得内容
        try {
            DataOutputStream out = new DataOutputStream(new FileOutputStream(new File(filePath)));
            for(int i = 0; i < data.length; i++) {
                out.write(data[i]);
            }
            out.flush();
            out.close();
            System.out.println("文件"+ fileName + "已经存放在"+ filePath  );
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
然后是地址初始化函数initSeeds()，开始先设置几个种子地址，然后从这几个地址开始去爬取网页
```java
    private void initCrawlerWithSeeds(String[] seeds) {
        for (int i = 0; i < seeds.length; i++){
        Links.addUnvisitedUrlQueue(seeds[i]);
    }
```
接着是getLinks()方法，这个方法获取该页面内所有图片链接:
```java
    public static Set<String> getLinks(Page page){
        Set<String> links  = new HashSet<String>() ; 
        Elements element=page.getDoc().select("img[src]");      
        Iterator iterator  = es.iterator();
        while(iterator.hasNext()) { 
            Element element = (Element) iterator.next();
            System.out.println("element.text() get:"+element.text()+":"+element.attr("abs:src"));
            links.add(element.attr("abs:src"));
        }
        return links;
    }
```
最后是爬取函数，这个函数需要设置保存路径path：
```java
    public void crawling(String[] seeds,String path) {
        initCrawlerWithSeeds(seeds);
        while (!Links.unVisitedUrlQueueIsEmpty()  && Links.getVisitedUrlNum() <= 1000) {  
            try{
                String visitUrl = (String) Links.removeHeadOfUnVisitedUrlQueue();    
                if (visitUrl == null){
                    continue;
                }
                Page page = RequestAndResponseTool.sendRequstAndGetResponse(visitUrl);
                Elements element = page.getDoc().getLinks(page);
                if(element.isEmpty()){
                    continue;
                }
                FileTool.save(page);
                Links.addVisitedUrlSet(visitUrl);
                Set<String> links = save(page,path);    //保存网页
                for (String link : links) {
                Links.addUnvisitedUrlQueue(link);
                }
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
```
终于写完啦，来看看最终*Spider*类的样子：
```java
    import org.jsoup.select.Elements;
    import java.util.Set;
    public class Spider {
        private void initCrawlerWithSeeds(String[] seeds) {
        for (int i = 0; i < seeds.length; i++){
        Links.addUnvisitedUrlQueue(seeds[i]);
        }
    }
    public static void save(Page page,String path) {
        String fileName = page.getUrl(); //把链接名作为文件名
        String filePath = path+“/”+fileName; //存放的地址
        byte[] data = page.getContent(); //获得内容
        try {
            DataOutputStream out = new DataOutputStream(new FileOutputStream(new File(filePath)));
            for(int i = 0; i < data.length; i++) {
            out.write(data[i]);
            }
            out.flush();
            out.close();
            System.out.println("文件"+ fileName + "已经存放在"+ filePath  );
            } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static Set<String> getLinks(Page page){
        Set<String> links  = new HashSet<String>() ; 
        Elements element=page.getDoc().select("img[src]");      
        Iterator iterator  = es.iterator();
        while(iterator.hasNext()) { 
            Element element = (Element) iterator.next();
            System.out.println("element.text() get:"+element.text()+":"+element.attr("abs:src"));
            links.add(element.attr("abs:src"));
        }
        return links;
    }
    public void crawling(String[] seeds,String path) {
        initCrawlerWithSeeds(seeds);
        while (!Links.unVisitedUrlQueueIsEmpty()  && Links.getVisitedUrlNum() <= 1000) {  
            try{
                String visitUrl = (String) Links.removeHeadOfUnVisitedUrlQueue();    
                if (visitUrl == null){
                    continue;
                }
                Page page = RequestAndResponseTool.sendRequstAndGetResponse(visitUrl);
                Elements element = page.getDoc().getLinks(page);
                if(element.isEmpty()){
                    continue;
                }
                FileTool.save(page);
                Links.addVisitedUrlSet(visitUrl);
                Set<String> links = save(page,path);    //保存网页
                for (String link : links) {
                    Links.addUnvisitedUrlQueue(link);
                }
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) {
        Spider crawler = new Spider();
        crawler.crawling(new String[]{"https://www.ustc.edu.cn/"},"clawer_file");
    }
}
```
爬取地址设置的是中国科学技术大学官网首页，存储地址在clawer_file文件夹中



