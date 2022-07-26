---
title: HttpClient的使用
categories:
  - Java
  - JavaWeb
tags:
  - HttpClient
copyright: false
top_img: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp8.jpg'
cover: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp8.jpg'
abbrlink: 983b
date: 2021-06-06 22:30:52
updated: 2021-6-24 23:44:48
---

[HttpClient官网](http://hc.apache.org/)

本文参考地址：[【CSDN】HttpClient详细使用示例  justry_deng](https://blog.csdn.net/justry_deng/article/details/81042379)

HttpClient主要用于访问其他接口

# 准备

**引入相关依赖：**

```xml
        <!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.13</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.76</version>
        </dependency>
```

>fastjson有将对象转化为json字符串的功能等。
>
>这里使用的是4.5版本的httpclient，各个版本在使用上会有差异。

---

**用于以下案例的测试：**

```java
@RestController
public class MyController {

    @GetMapping("/doGet1")
    public String doGet1() {
        return "这个Get请求没有任何参数";
    }

    @GetMapping("/doGet2")
    public String doGet2(String name, Integer age) {
        return name + "今年" + age + "岁了";
    }

    @PostMapping("/doPost1")
    public String doPost1() {
        return "这个Post请求没有任何参数";
    }

    @PostMapping("/doPost2")
    public String doPost2(String name, Integer age) {
        return name + "今年" + age + "岁了";
    }

    @PostMapping("/doPost3")
    public String doPost3(@RequestBody User user) {
        return user.toString();
    }

    @PostMapping("/doPost4")
    public String doPost4(@RequestBody User user, Integer flag, String meaning) {
        return user.toString() + "\n" + flag + ">>>" + meaning;
    }
}
```



# GET方式

## 无参

```java
public static void doGet1() {
        // 获得Http客户端
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();
        // 创建Get请求
        HttpGet httpGet = new HttpGet("http://localhost:8080/doGet1");
        // 响应模型
        CloseableHttpResponse response = null;
        try {
            // 由客户端执行(发送)Get请求
            response = httpClient.execute(httpGet);
            System.out.println("响应状态为:" + response.getStatusLine());

            // 从响应模型中获取响应实体
            HttpEntity responseEntity = response.getEntity();
            if (responseEntity != null) {
                System.out.println("响应内容长度为:" + responseEntity.getContentLength());
                System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
            }
            //释放实体资源
            EntityUtils.consume(responseEntity);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                // 释放资源
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```



## 有参方式一

* **手动在url后面加上参数**

```java
public static void doGet2() {
        // 获得Http客户端
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();
        // 参数
        StringBuilder params = new StringBuilder();
        try {
            // 字符数据最好encoding以下;这样一来，某些特殊字符才能传过去(如:某人的名字就是“&”,不encoding的话,传不过去)
            params.append("name=")
                    .append(URLEncoder.encode("阿楠", "utf-8"))
                    .append("&")
                    .append("age=24");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        // 创建Get请求
        HttpGet httpGet = new HttpGet("http://localhost:8080/doGet2" + "?" + params);
        // 响应模型
        CloseableHttpResponse response = null;
        try {
            // 配置信息
            RequestConfig requestConfig = RequestConfig.custom()
                    // 设置连接超时时间(单位毫秒)
                    .setConnectTimeout(5000)
                    // 设置请求超时时间(单位毫秒)
                    .setConnectionRequestTimeout(5000)
                    // socket读写超时时间(单位毫秒)
                    .setSocketTimeout(5000)
                    // 设置是否允许重定向(默认为true)
                    .setRedirectsEnabled(true).build();

            // 将上面的配置信息 运用到这个Get请求里
            httpGet.setConfig(requestConfig);

            // 由客户端执行(发送)Get请求
            response = httpClient.execute(httpGet);
            System.out.println("响应状态为:" + response.getStatusLine());

            // 从响应模型中获取响应实体
            HttpEntity responseEntity = response.getEntity();
            if (responseEntity != null) {
                System.out.println("响应内容长度为:" + responseEntity.getContentLength());
                System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
            }
            //释放实体资源
            EntityUtils.consume(responseEntity);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                // 释放资源
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



## 有参方式二

- **将参数放入键值对类中，再放入URI中，从而通过URI得到HttpGet实例**

```java
public static void doGet3() {
        // 获得Http客户端
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();
        // 参数
        URI uri = null;
        try {
            // 将参数放入键值对类NameValuePair中,再放入集合中
            List<NameValuePair> params = new ArrayList<>();
            params.add(new BasicNameValuePair("name", "阿楠"));
            params.add(new BasicNameValuePair("age", "18"));
            // 设置uri信息,并将参数集合放入uri;
            // 注:这里也支持一个键值对一个键值对地往里面放setParameter(String key, String value)
            uri = new URIBuilder().setScheme("http").setHost("localhost")
                    .setPort(8080).setPath("/doGet2")
                    .setParameters(params).build();
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
        // 创建Get请求
        HttpGet httpGet = new HttpGet(uri);

        // 响应模型
        CloseableHttpResponse response = null;
        try {
            // 配置信息
            RequestConfig requestConfig = RequestConfig.custom()
                    // 设置连接超时时间(单位毫秒)
                    .setConnectTimeout(5000)
                    // 设置请求超时时间(单位毫秒)
                    .setConnectionRequestTimeout(5000)
                    // socket读写超时时间(单位毫秒)
                    .setSocketTimeout(5000)
                    // 设置是否允许重定向(默认为true)
                    .setRedirectsEnabled(true).build();

            // 将上面的配置信息 运用到这个Get请求里
            httpGet.setConfig(requestConfig);

            // 由客户端执行(发送)Get请求
            response = httpClient.execute(httpGet);
            System.out.println("响应状态为:" + response.getStatusLine());

            // 从响应模型中获取响应实体
            HttpEntity responseEntity = response.getEntity();
            if (responseEntity != null) {
                System.out.println("响应内容长度为:" + responseEntity.getContentLength());
                System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
            }
            //释放实体资源
            EntityUtils.consume(responseEntity);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                // 释放资源
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



# POST方式

## 无参

```java
 public static void doPost1() {
        // 获得Http客户端
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();
        // 创建Post请求
        HttpPost httpPost = new HttpPost("http://localhost:8080/doPost1");
        // 响应模型
        CloseableHttpResponse response = null;
        try {
            // 由客户端执行(发送)Post请求
            response = httpClient.execute(httpPost);
            System.out.println("响应状态为:" + response.getStatusLine());

            // 从响应模型中获取响应实体
            HttpEntity responseEntity = response.getEntity();
            if (responseEntity != null) {
                System.out.println("响应内容长度为:" + responseEntity.getContentLength());
                System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
            }
            //释放实体资源
            EntityUtils.consume(responseEntity);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                // 释放资源
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

```



## 有参（普通参数）

```java
public static void doPost2() {
        // 获得Http客户端
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();

        // 参数
        StringBuilder params = new StringBuilder();
        try {
            // 字符数据最好encoding以下;这样一来，某些特殊字符才能传过去(如:某人的名字就是“&”,不encoding的话,传不过去)
            params.append("name=").append(URLEncoder.encode("nan", "utf-8"))
                    .append("&")
                    .append("age=24");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        // 创建Post请求
        HttpPost httpPost = new HttpPost("http://localhost:8080/doPost2" + "?" + params);

        // 设置ContentType(注:如果只是传普通参数的话,ContentType不一定非要用application/json)
        httpPost.setHeader("Content-Type", "application/json;charset=utf8");

        // 响应模型
        CloseableHttpResponse response = null;
        try {
            // 由客户端执行(发送)Post请求
            response = httpClient.execute(httpPost);
            System.out.println("响应状态为:" + response.getStatusLine());

            // 从响应模型中获取响应实体
            HttpEntity responseEntity = response.getEntity();
            if (responseEntity != null) {
                System.out.println("响应内容长度为:" + responseEntity.getContentLength());
                System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                // 释放资源
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



## 有参（对象参数）

```java
 public static void doPost3() {
        // 获得Http客户端
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();

        // 创建Post请求
        HttpPost httpPost = new HttpPost("http://localhost:8080/doPost3");
        User user = new User();
        user.setName("霍华德");
        user.setAge(35);
        user.setGender("男");
        user.setMotto("扣篮要暴力~");

        // 利用阿里的fastjson，将Object转换为json字符串;
        // (需要导入com.alibaba.fastjson.JSON包)
        String jsonString = JSON.toJSONString(user);

        StringEntity entity = new StringEntity(jsonString, "UTF-8");

        // post请求是将参数放在请求体里面传过去的;这里将entity放入post请求体中
        httpPost.setEntity(entity);

        httpPost.setHeader("Content-Type", "application/json;charset=utf8");

        // 响应模型
        CloseableHttpResponse response = null;
        try {
            // 由客户端执行(发送)Post请求
            response = httpClient.execute(httpPost);
            System.out.println("响应状态为:" + response.getStatusLine());

            // 从响应模型中获取响应实体
            HttpEntity responseEntity = response.getEntity();
            if (responseEntity != null) {
                System.out.println("响应内容长度为:" + responseEntity.getContentLength());
                System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
            }
            //释放实体资源
            EntityUtils.consume(responseEntity);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                // 释放资源
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



## 有参（普参+对参）

```java
 public static void doPost4(){
        // 获得Http客户端
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();

        // 创建Post请求
        // 参数
        URI uri = null;
        try {
            // 将参数放入键值对类NameValuePair中,再放入集合中
            List<NameValuePair> params = new ArrayList<>();
            params.add(new BasicNameValuePair("flag", "4"));
            params.add(new BasicNameValuePair("meaning", "这是普通参数"));
            // 设置uri信息,并将参数集合放入uri;
            // 注:这里也支持一个键值对一个键值对地往里面放setParameter(String key, String value)
            uri = new URIBuilder().setScheme("http").setHost("localhost").setPort(8080)
                    .setPath("/doPost4").setParameters(params).build();
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }

        HttpPost httpPost = new HttpPost(uri);

        // 创建user参数
        User user = new User();
        user.setName("霍华德");
        user.setAge(35);
        user.setGender("男");
        user.setMotto("扣篮要暴力~");

        // 将user对象转换为json字符串，并放入entity中
        StringEntity entity = new StringEntity(JSON.toJSONString(user), "UTF-8");

        // post请求是将参数放在请求体里面传过去的;这里将entity放入post请求体中
        httpPost.setEntity(entity);

        httpPost.setHeader("Content-Type", "application/json;charset=utf8");

        // 响应模型
        CloseableHttpResponse response = null;
        try {
            // 由客户端执行(发送)Post请求3
            response = httpClient.execute(httpPost);
            System.out.println("响应状态为:" + response.getStatusLine());

            // 从响应模型中获取响应实体
            HttpEntity responseEntity = response.getEntity();

            if (responseEntity != null) {
                System.out.println("响应内容长度为:" + responseEntity.getContentLength());
                System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
            }
            //释放实体资源
            EntityUtils.consume(responseEntity);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                // 释放资源
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



# GET和POST请求封装

**DTO对象的封装：**

```java
/**
 * @Date: 2021/06/07/9:15 PM
 * @Description: ApiUtil所需要的DTO对象
 */
public class ApiRequest {

    //使用的api接口名
    private String apiUrl;

    //接口响应时间
    private Long callTimeStamp;

    //状态码200（正常）/400（异常）
    private Integer returnCode;

    //请求参数
    private Object request;

    //返回结果
    private Object result;

    public String getApiUrl() {
        return apiUrl;
    }

    public void setApiUrl(String apiUrl) {
        this.apiUrl = apiUrl;
    }

    public Long getCallTimeStamp() {
        return callTimeStamp;
    }

    public void setCallTimeStamp(Long callTimeStamp) {
        this.callTimeStamp = callTimeStamp;
    }

    public Integer getReturnCode() {
        return returnCode;
    }

    public void setReturnCode(Integer returnCode) {
        this.returnCode = returnCode;
    }

    public Object getRequest() {
        return request;
    }

    public void setRequest(Object request) {
        this.request = request;
    }

    public Object getResult() {
        return result;
    }

    public void setResult(Object result) {
        this.result = result;
    }

    @Override
    public String toString() {
        return "ApiRequest{" +
                "apiUrl='" + apiUrl + '\'' +
                ", callTimeStamp=" + callTimeStamp +
                ", returnCode=" + returnCode +
                ", request=" + request +
                ", result=" + result +
                '}';
    }
}
```

---

**调用接口的封装：**

```java

public class ApiUtil {

    public static final String CHARSET = "UTF-8";
    public static final Integer SUCCESS_CODE = 200;
    public static final Integer ERROR_CODE = 400;

    /**
     * @Description:根据method调用API接口返回封装后的dto
     * @Param: [method, url, params]
     * @Return: java.lang.String
     * @date: 6/7/2021
     * @Time: 3:37 PM
     */
    public static ApiRequest request(String method, String url, Map<String, String> params) throws Exception {
        if (method == null) {
            throw new RuntimeException("指定请求方式POST/GET");
        } else if (method.equalsIgnoreCase("GET")) {
            return doGet(url, params);
        } else if (method.equalsIgnoreCase("POST")) {
            return doPost(url, params);
        } else {
            throw new RuntimeException("指定请求方式POST/GET");
        }
    }

    /**
     * @Description:GET请求方式封装
     * @Param: [url, params]
     * @Return: aia.cn.oeServiceCommon.dto.ApiRequest
     * @author: Nan-ZN.Xu@aia.com
     * @date: 6/8/2021
     * @Time: 10:35 AM
     */
    public static ApiRequest doGet(String url, Map<String, String> params) {
        RequestConfig config = RequestConfig.custom()
                .setSocketTimeout(5000)//数据传输过程中数据包之间间隔的最大时间
                .setConnectTimeout(5000)//连接建立时间
                .setConnectionRequestTimeout(5000)//从连接池获取连接的超时时间
                .build();
        //创建httpclient客户端并设置请求配置
        CloseableHttpClient httpClient = HttpClientBuilder.create().setDefaultRequestConfig(config).build();

        //初始化DTO对象
        ApiRequest apiRequest = new ApiRequest();
        apiRequest.setApiUrl(url);
        apiRequest.setReturnCode(ERROR_CODE);
        apiRequest.setCallTimeStamp(null);
        apiRequest.setRequest(params);
        apiRequest.setResult(null);

        if (StringUtils.isBlank(url)) {
            return apiRequest;
        }

        if (params != null && !params.isEmpty()) {
            try {
                List<NameValuePair> pairs = new ArrayList<>(params.size());
                for (Map.Entry<String, String> entry : params.entrySet()) {
                    String value = entry.getValue();
                    if (value != null) {
                        pairs.add(new BasicNameValuePair(entry.getKey(), value));
                    }
                }
                //将参数拼接在url后面同时设置编码格式
                url += "?" + EntityUtils.toString(new UrlEncodedFormEntity(pairs), CHARSET);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        //创建Get请求
        HttpGet httpGet = new HttpGet(url);
        CloseableHttpResponse response = null;

        try {
            long startTimeStamp = System.currentTimeMillis();
            //由客户端执行(发送)Get请求
            response = httpClient.execute(httpGet);
            long endTimeStamp = System.currentTimeMillis();

            //获取状态码
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode != SUCCESS_CODE) {
                apiRequest.setCallTimeStamp(endTimeStamp - startTimeStamp);
                //中止Get请求
                httpGet.abort();
            }

            //从响应模型中获取响应实体
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                apiRequest.setReturnCode(SUCCESS_CODE);
                //设置响应时间
                apiRequest.setCallTimeStamp(endTimeStamp - startTimeStamp);
                //获取响应内容
                String result = EntityUtils.toString(entity, CHARSET);
                //将JSON文本串转换为对象
                JSONObject resultOj = JSONObject.parseObject(result);
                apiRequest.setResult(resultOj);
            }
            //释放实体资源
            EntityUtils.consume(entity);
        } catch (ParseException | IOException e) {
            e.printStackTrace();
        } finally {
            //释放资源
            try {
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return apiRequest;
    }

    /**
     * @Description:POST请求方式封装
     * @Param: [url, params]
     * @Return: aia.cn.oeServiceCommon.dto.ApiRequest
     * @date: 6/8/2021
     * @Time: 10:44 AM
     */
    public static ApiRequest doPost(String url, Map<String, String> params) {
        RequestConfig config = RequestConfig.custom()
                .setSocketTimeout(5000)//数据传输过程中数据包之间间隔的最大时间
                .setConnectTimeout(5000)//连接建立时间
                .setConnectionRequestTimeout(5000)//从连接池获取连接的超时时间
                .build();
        //创建httpclient客户端并设置请求配置
        CloseableHttpClient httpClient = HttpClientBuilder.create().setDefaultRequestConfig(config).build();

        //初始化DTO对象
        ApiRequest apiRequest = new ApiRequest();
        apiRequest.setApiUrl(url);
        apiRequest.setReturnCode(ERROR_CODE);
        apiRequest.setCallTimeStamp(null);
        apiRequest.setRequest(params);
        apiRequest.setResult(null);

        if (StringUtils.isBlank(url)) {
            return apiRequest;
        }

        //创建POST请求
        HttpPost httpPost = new HttpPost(url);
        if (params != null && !params.isEmpty()) {
            try {
                List<NameValuePair> pairs = new ArrayList<>(params.size());
                for (Map.Entry<String, String> entry : params.entrySet()) {
                    String value = entry.getValue();
                    if (value != null) {
                        pairs.add(new BasicNameValuePair(entry.getKey(), value));
                    }
                }
                //设置参数
                httpPost.setEntity(new UrlEncodedFormEntity(pairs, CHARSET));
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }

        CloseableHttpResponse response = null;
        try {
            long startTimeStamp = System.currentTimeMillis();
            //发送请求
            response = httpClient.execute(httpPost);
            long endTimeStamp = System.currentTimeMillis();

            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode != SUCCESS_CODE) {
                apiRequest.setCallTimeStamp(endTimeStamp - startTimeStamp);
                //中止请求
                httpPost.abort();
            }

            //从响应模型中获取响应实体
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                apiRequest.setReturnCode(SUCCESS_CODE);
                apiRequest.setCallTimeStamp(endTimeStamp - startTimeStamp);
                //获取响应内容并转换为对象
                String result = EntityUtils.toString(entity, CHARSET);
                JSONObject resultOj = JSONObject.parseObject(result);
                apiRequest.setResult(resultOj);
            }
            //释放实体资源
            EntityUtils.consume(entity);
        } catch (ParseException | IOException e) {
            e.printStackTrace();
        } finally {
            //释放资源
            try {
                if (httpClient != null) {
                    httpClient.close();
                }
                if (response != null) {
                    response.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return apiRequest;
    }
}
```

