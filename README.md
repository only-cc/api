
    //用户创建学校或机构,返回机构或学校ID
    final static int orgId = 30001;

    // 用户创建学校或机构返回的APIKEY
    final static String secretKey = "6a3747a06dc211e7a8c4816a023b4ce6";

    final static String requestApiUrl = "http://cc.knoocrc.cn/cc-api/v1/session/list";

    private static StringBuffer getParamStr(Map<String, Object> params) {
        if (params == null) {
            return null;
        }
        StringBuffer paramStr = new StringBuffer();
        Iterator<String> iterator = params.keySet().iterator();
        while (iterator.hasNext()) {
            String name = iterator.next();
            paramStr.append(name);
            Object value = params.get(name);
            if (value.getClass().isArray()) {
                Object[] objects = (Object[]) value;
                for (Object object : objects) {
                    paramStr.append(object.toString());
                }
            } else {
                paramStr.append(value);
            }
        }
        return paramStr;
    }

    /**
     * 签名例子
     * secretKey是736bd0cee32f4291a7ec7ddf368c1b85
     * 待签名字符串如下
     * video=0&duration=2&startTime=2013-12-17 17:11&partner=20131213190138&title=test course
     * 拼接后为
     * duration=2&partner=20131213190138&startTime=2013-12-17 17:11&title=test course&video=0736bd0cee32f4291a7ec7ddf368c1b85
     * 再对上述字符串进行16位MD5HEX
     * 身份校验完成后还要判断时间戳是否超时，有效期是5分钟，如果超时返回错误：时间戳超时
     *
     * @param paramMap
     * @param secretKey
     * @return
     */
    public static String md5Signature(Map<String, Object> paramMap, String secretKey) {
        String result = null;
        StringBuffer originalStr = new StringBuffer();
        originalStr.append(getParamStr(paramMap)).append(secretKey);
        if (originalStr == null) {
            return result;
        }
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] bArr = originalStr.toString().getBytes("UTF-8");
            byte[] md5Value = md.digest(bArr);
            BigInteger bigInt = new BigInteger(1, md5Value);
            result = bigInt.toString(16);
            while (result.length() < 32)
                result = "0" + result;
        } catch (Exception e) {
            throw new RuntimeException("sign error !");
        }
        return result;
    }

    public static void main(String[] args) throws Exception {
        long timestamp = new Date().getTime();
        TreeMap<String, Object> apiMap = new TreeMap<String, Object>();
        apiMap.put("orgId", orgId);
        apiMap.put("timestamp", timestamp);
        String sign = Test.md5Signature(apiMap, secretKey);
        apiMap.put("sign", sign);
        Connection connection = getConnection(requestApiUrl, Connection.Method.POST, apiMap);
        String data = connection.execute().body();
        System.out.print(data);
    }

    public static Connection getConnection(String url, Connection.Method method, Map<String, Object> paramMap) {
        Connection connection = Jsoup.connect(url).timeout(10 * 60 * 1000).ignoreContentType(true);
        connection.method(method);
        if (null != paramMap) {
            for (String key : paramMap.keySet()) {
                connection.data(key, paramMap.get(key).toString());
            }
        }
        return connection;
    }