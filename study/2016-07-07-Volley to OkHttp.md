�� volley �л��� OkHttp �����Ŀ�

[ԭ������](http://gdky005.com/2016/06/20/%E4%BB%8E-volley-%E5%88%87%E6%8D%A2%E5%88%B0-OkHttp-%E9%81%87%E5%88%B0%E7%9A%84%E5%9D%91/)


�⼸��������Ŀ�� volley �л��� okhttp,������һЩС���⣬������������

֮ǰ����ֱ�ӽ� volley �л��� okhttp, �ײ�϶�ʹ�� okhttp, �������Ҳʹ�� okhttp�����ǿ��ǵ����ۿ��ܱȽϴ��������ǻ������ϸ��Ľ�������� �ϲ������Ȼʹ�� volley,���Ƕ��ڵײ㷢������ĵط�������ֱ���л��� okhttp.

�����쳣��

�л��ɹ��������ĵ�һ��������ǣ�������û��ʹ�ã����ǿͻ��� ������ͨ�������Ĺ��ܵģ���˱���Ҫ�� �����ܡ�

���� okhttp ���� issue �Ļش�Ū�ö�ζ����У�������һ�������ҡ� ���Ҳ���ù��ˣ��ȷŷţ����Ƚ���������⡣

֮����˼��죬�ٻ���Ū����ʱ�򣬾�ͻȻ���ˣ��˷ܻ��ˡ��Ͻ�����֮ǰ����û��ʲô���죿

�����ԱȺ��֣�ԭ���� ֮ǰд volley ��ʱ���������ģ�

```java
HttpURLConnection connection;
    if ("https".equals(url.getProtocol())) {
        Proxy proxy = new Proxy(Proxy.Type.HTTP,
                InetSocketAddress
                        .createUnresolved(FLOWPACKAGEHOST, FLOWPACKAGETCPPORT));
        connection = (HttpURLConnection) url
                .openConnection(proxy);
        connection.addRequestProperty("Proxy-Authorization",
                "Basic MzAwMDAwNDU4NDo0Mjk5NjREMEI1Q0QyODc6");
    } else {
        Proxy proxy = new Proxy(Proxy.Type.HTTP,
                InetSocketAddress
                        .createUnresolved(FLOWPACKAGEHOST, FLOWPACKAGEPORT));
        connection = (HttpURLConnection) url
                .openConnection(proxy);
        connection.addRequestProperty("Authorization",
                "Basic MzAwMDAwNDU4NDo0Mjk5NjREMEI1Q0QyODc6");
    }
    connection.addRequestProperty("Proxy-Connection", "Keep-Alive");
    
```

��Ҫ������ https �� http, Ȼ�����洫��� key �� �˿ںŶ���һ����

������ okhttp ����ò���ǲ���Ҫ���ֵġ�ֻ��Ҫ����д��


```java
/**
     * ������ͨ���� ������
     * @param builder
     */
    private void setUnicomProxy(OkHttpClient.Builder builder) {
        //�����ͨ������
        if (TrafficUtil.getUnicomProxyAvailable()) {
            Authenticator proxyAuthenticator = new Authenticator() {
                @Override
                public okhttp3.Request authenticate(Route route, Response response) throws IOException {
                    return response.request().newBuilder().header("Proxy-Authorization", "Basic " +
                            "MzAwMDAwNDU4NDo0Mjk5NjREMEI1Q0QyODc6").header("Proxy-Connection",
                            "Keep-Alive").build();
                }
            };

            builder.proxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress(FLOWPACKAGEHOST,
                    FLOWPACKAGETCPPORT)));
            builder.proxyAuthenticator(proxyAuthenticator);
        }
    }
```
�Ϳ����ˡ�
FLOWPACKAGEHOST -> test.proxy.1111.com (��������)
FLOWPACKAGETCPPORT -> 8143
�⻹������һ��żȻ�Ļ��ᣬ������ţ�������Ƶ��ų��þá�

��ע�� ���� key �������޸��˼����ַ����������У���Ҫֱ��ʹ�ÿ϶����еģ� ����

SSL/STL֤�� ����

���ǵڶ������������⣬֤��һֱû���ã�һʹ�� https �Ľӿھ�ʧ�ܡ�������취�ǣ�

```java
    @NonNull
    private SSLContext getSslContext(InputStream... certificates) {
        SSLContext sslContext = null;

        try {
            CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            keyStore.load(null);
            int index = 0;
            for (InputStream certificate : certificates) {
                String certificateAlias = Integer.toString(index++);
                keyStore.setCertificateEntry(certificateAlias, certificateFactory.generateCertificate
                        (certificate));
                try {
                    if (certificate != null)
                        certificate.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            sslContext = SSLContext.getInstance("TLS");

            TrustManagerFactory trustManagerFactory =
                    TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());

            trustManagerFactory.init(keyStore);
            sslContext.init(null, trustManagerFactory.getTrustManagers(),
                    new SecureRandom());

        } catch (KeyStoreException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (KeyManagementException e) {
            e.printStackTrace();
        } catch (Exception e){
            e.printStackTrace();
        }   finally {
        }

        return sslContext;
    }


/**
     * ���� OkHttps ����У�鹦��
     * @param builder
     */
    private void setOkhttpSSLContext(OkHttpClient.Builder builder) {
        SSLContext sslContext = getSslContext(KaolaApplication.mContext.getResources().openRawResource(R
                .raw.kl_magic));

        if (sslContext != null) {
            builder.hostnameVerifier(new HostnameVerifier() {
                @Override
                public boolean verify(String hostname, SSLSession session) {
                    HostnameVerifier hostnameVerifier = HttpsURLConnection.getDefaultHostnameVerifier();
                    return hostnameVerifier.verify("xxx.com", session); //����xxx ����У�� (������һ���ǳ���Ҫ�ĵط���ȱ������һ���϶�����)
                }
            });
            builder.sslSocketFactory(sslContext.getSocketFactory());
        }
    }
```

����Ƿǳ���Ҫ�ģ�
hostnameVerifier.verify("xxx.com", session);��Ҫ�ǽ��� https ������У�飬��֤��ƥ��������������ƥ��ɹ�����ô�Ϳ���ֱ��ʹ�ã����� https ����ʧ�ܣ��޷���ȷ��������

֮ǰ���Թ�ʹ�� ʹ��
builder.hostnameVerifier(new AllowAllHostnameVerifier());

�������Ժ���֤�飬Ĭ�϶�����Ҳ������ʹ�ã����� ���ڰ�ȫ������

post ��������Ϊ�գ�

������������ıȽ����⣬ԭ���ǣ����ǵ� https �Ľӿ�ʹ���� post ���󣬵��� post ����û�в�����ͨ�ò��������� url ����׷���ˣ������� ��� request û�� body( body ���Ƕ� post ����Ĳ��� ������).

���� okhttp ���ڲ���Ϊ�յ�����ֱ�ӷ��� null, ���Զ������ֲ��淶�� �ӿڶ���ͱ����ˡ��� okhttp issue ����Ҳ�й�����������ۣ�˵����������� http �ı�׼�����Բ��ܷ������󡣽���취�����һ���յ� �����Ϳ��ԣ����Ǿ����� ����:����, ���������ֵ���������ߺͷ�����Լ����һ�£��� temp ���棬������Ҳ�϶�����������ֶ�ȡ���ݡ�

����ο������
```java
public void addRequest(int method, final Map<String, String> params, final String baseUrl,
                           final TypeReference<? extends BaseResponse> type, final JsonResultCallback callback) {
            ......
          if (params.size() == 0 && method == Request.Method.POST)
            params.put("temp", "temp"); //��� method POST must have a request body.;
            ......
}
```
�������������󣬻����Ϳ��Է���ʹ���ˡ�