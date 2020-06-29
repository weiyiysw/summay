# 实用代码片段

下载模板文件

~~~java
public void downloadTemplate(HttpServeletResponse response) throws Exception {
    // todo: set prefix
    String path = prefix + File.separator + "deviceid_upload_template.xlsx";
            File file = new File(path);
            // 以流的形式下载文件。
            InputStream fis = new FileInputStream(file);
            byte[] buffer = new byte[fis.available()];
            fis.read(buffer);
            fis.close();
            // 清空response
            response.reset();
            // 设置response的Header
            response.addHeader("Content-Disposition", "attachment;filename=" + filename);
            response.addHeader("Content-Length", "" + file.length());
            OutputStream toClient = new BufferedOutputStream(response.getOutputStream());
            response.setContentType("application/octet-stream");
            toClient.write(buffer);
            toClient.flush();
            toClient.close();
}
~~~



HttpClient工具类

~~~java
public class HttpClientUtil {

    private final static Logger log = LoggerFactory.getLogger(HttpClientUtil.class);

    private final static OkHttpClient httpClient = new OkHttpClient();

    public final static String HTTP_GET = "GET";
    public final static String HTTP_POST = "POST";
    public final static String HTTP_PUT = "PUT";

    private static final MediaType JSON = MediaType.parse("application/json; charset=utf-8");

    public static Response doGet(String url, Map<String, String> params) {
        return doGet(url, null, params);
    }

    public static Response doGet(String url, Map<String, String> headers, Map<String, String> params) {
        return execute(httpClient, url, HTTP_GET,null, headers, params);
    }

    public static Response doGet(OkHttpClient client, String url, Map<String, String> params) {
        return execute(client, url, HTTP_GET,null, null, params);
    }

    public static Response doPostWithFormData(String url, Map<String, String> formData) {
        return doPostWithFormData(url, null, formData);
    }

    public static Response doPostWithFormData(String url, Map<String, String> headers,
                                  Map<String, String> formData) {
        return doPost(httpClient, url, headers, formData, null);
    }

    public static Response doPostWithJson(String url, String json) {
        return doPostWithJson(url, null, json);
    }

    public static Response doPostWithJson(String url, Map<String, String> headers, String json) {
        return doPost(httpClient, url, headers, null, json);
    }

    private static Response doPost(OkHttpClient client, String url, Map<String, String> headers,
                                  Map<String, String> formData, String json) {
        RequestBody body = generateBody(formData, json);

        return execute(client, url, HTTP_POST, body, headers, null);
    }

    public static Response doPutWithFormData(String url, Map<String, String> formData) {
        return doPutWithFormData(url, null, formData);
    }

    public static Response doPutWithFormData(String url, Map<String, String> headers,
                                              Map<String, String> formData) {
        return doPut(httpClient, url, headers, formData, null);
    }

    public static Response doPutWithJson(String url, String json) {
        return doPutWithJson(url, null, json);
    }

    public static Response doPutWithJson(String url, Map<String, String> headers, String json) {
        return doPut(httpClient, url, headers, null, json);
    }

    private static Response doPut(OkHttpClient client, String url, Map<String, String> headers,
                                 Map<String, String> formData, String json) {
        RequestBody body = generateBody(formData, json);

        return execute(client, url, HTTP_PUT, body, headers, null);
    }

    /**
     * formData and body can not be null at the same time!!!
     * if so, throw RuntimeException
     *
     * @param formData
     * @param body
     * @return
     */
    private static RequestBody generateBody(Map<String, String> formData, String body) {
        if (formData != null && !formData.isEmpty()) {
            FormBody.Builder builder = new FormBody.Builder();
            for (Map.Entry<String, String> entry : formData.entrySet()) {
                builder.add(entry.getKey(), entry.getValue());
            }
            return builder.build();
        } else {
            if (StringUtils.isEmpty(body)) {
                log.error("post call use error, only support formData or json!");
                throw new RuntimeException("request body generate err!");
            }
            return RequestBody.create(JSON, body);
        }
    }

    private static Response execute(OkHttpClient client, String url, String method, RequestBody body,
                                   Map<String, String> headers, Map<String, String> getParams) {
        Request request = generateRequest(url, method, body, headers,  getParams);
        Call call = client.newCall(request);
        try {
            return call.execute();
        } catch (IOException e) {
            log.error("execute {} {} form error!", url, method, e);
        }
        return null;
    }

    /**
     *
     * @param url
     * @param method only support GET or POST or PUT
     * @param headers
     * @param params
     * @return
     */
    private static Request generateRequest(String url, String method, RequestBody body,
                                           Map<String, String> headers, Map<String, String> params) {
        HttpUrl.Builder urlBuilder = HttpUrl.parse(url).newBuilder();
        if (params != null && !params.isEmpty()) {
            for (Map.Entry<String, String> entry : params.entrySet()) {
                urlBuilder.addQueryParameter(entry.getKey(), entry.getValue());
            }
        }

        String resultUrl = urlBuilder.build().toString();
        Request.Builder builder = new Request.Builder().url(resultUrl);
        if (headers != null && !headers.isEmpty()) {
            for(Map.Entry<String, String> header : headers.entrySet()) {
                builder.addHeader(header.getKey(), header.getValue());
            }
        }

        return builder.method(method, body).build();
    }
}
~~~

