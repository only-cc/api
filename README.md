
    /**
     * Using API only requires three steps
     * 1. session/create
     * 2. idkey/create
     * 3. session/enter
     */

    public static final Logger log = LoggerFactory.getLogger(ApiTest.class);

    // Users create schools or institutions, return to an organization or school ID (replace their own)
    final static int orgId = 30019;

    // APIKEY returned by a user creating a school or organization
    final static String secretKey = "fc1e7e0e98ec11e7b092f100df051847";

    // Interface address (replacing the requested request)
    final static String API_SELECT_SESSION = "http://cc.knoocrc.cn/cc-api/v1/session/list";

    final static String API_CREATE_SESSION = "http://cc.knoocrc.cn/cc-api/v1/session/create";

    final static String API_END_SESSION = "http://cc.knoocrc.cn/cc-api/v1/session/end";

    final static String API_CREATE_SESSION_IDKEY = "http://cc.knoocrc.cn/cc-api/v1/idkey/create";

    final static char md5Chars[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

    String doSessionKey = "";

    /**
     * Priority creation
     */
    @Before
    public void beforeClass() throws Exception{
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("orgId", orgId);
        long timestamp = new Date().getTime();
        paramMap.put("timestamp", timestamp);
        paramMap.put("sessionName", "测试数据");

        paramMap.put("startTime", "2017-09-14 11:00:00");
        paramMap.put("endTime", "2017-09-14 17:00:00");
        paramMap.put("aheadMinutes", 10);
        paramMap.put("delayMinutes", 10);
        paramMap.put("sessionType", 2);

        String sign = generateSign(paramMap);
        if (sign == null) {
            Assert.fail();
        }
        paramMap.put("sign", sign);
        JSONObject value = httpPost(API_CREATE_SESSION, paramMap);
        if(value != null && (boolean)value.get("isSuccess")) {
            JSONObject dataValue = value.getJSONObject("data");
            doSessionKey = dataValue.getString("sessionKey");
        }
        log.info(String.valueOf(value));
    }

    @Test
    public void apiSelectSession() throws Exception {
        Map<String, Object> paramMap = new HashMap<String, Object>();

        //Gets the parameters that initiate the request
        paramMap.put("orgId", orgId);
        long timestamp = new Date().getTime();
        paramMap.put("timestamp", timestamp);

        // To generate the signature
        String sign = generateSign(paramMap);
        if (sign == null) {
            Assert.fail();
        }
        paramMap.put("sign", sign);
        JSONObject value = httpPost(API_SELECT_SESSION, paramMap);

        log.info(String.valueOf(value));
    }

    @Test
    public void createSessionIdkey() throws Exception {
        Map<String, Object> paramMap = new HashMap<String, Object>();

        //Gets the parameters that initiate the request
        paramMap.put("orgId", orgId);
        long timestamp = new Date().getTime();
        paramMap.put("timestamp", timestamp);
        paramMap.put("sessionKey", doSessionKey);
        paramMap.put("role", 1);

        // To generate the signature
        String sign = generateSign(paramMap);
        if (sign == null) {
            Assert.fail();
        }
        paramMap.put("sign", sign);
        JSONObject value = httpPost(API_CREATE_SESSION_IDKEY, paramMap);

        log.info(String.valueOf(value));
    }

    @Test
    public void endSession() throws Exception {
        Map<String, Object> paramMap = new HashMap<String, Object>();

        //Gets the parameters that initiate the request
        paramMap.put("orgId", orgId);
        long timestamp = new Date().getTime();
        paramMap.put("timestamp", timestamp);
        paramMap.put("sessionKey", doSessionKey);

        // To generate the signature
        String sign = generateSign(paramMap);
        if (sign == null) {
            Assert.fail();
        }
        paramMap.put("sign", sign);
        JSONObject value = httpPost(API_END_SESSION, paramMap);

        log.info(String.valueOf(value));
    }

### This is a fetching request parameter, then the key value is spelled into a string, and the above string is 16 bits of MD5HEX, for example

    private static String generateSign(Map<String, Object> params) throws Exception {
        Object[] array = params.keySet().toArray();

        // Rank all other parameters in key ascending order
        java.util.Arrays.sort(array);
        String keyStr = "";
        for (int i = 0; i < array.length; i++) {
            // Splicing key and corresponding value into a string
            String key = array[i].toString();
            keyStr += key + params.get(key);
        }

        // Splicing the allocated secretkey in the back of the string obtained by the request parameter
        keyStr += secretKey;

        // The hexadecimal string of md5 values is the value of sign
        return md5(keyStr);
    }

    public static String md5(String str) throws Exception {
        MessageDigest md5 = getMD5Instance();
        md5.update(str.getBytes("UTF-8"));
        byte[] digest = md5.digest();
        char[] chars = toHexChars(digest);
        return new String(chars);
    }

    private static MessageDigest getMD5Instance() {
        try {
            return MessageDigest.getInstance("MD5");
        } catch (NoSuchAlgorithmException ignored) {
            throw new RuntimeException(ignored);
        }
    }

    private static char[] toHexChars(byte[] digest) {
        char[] chars = new char[digest.length * 2];
        int i = 0;
        for (byte b : digest) {
            char c0 = md5Chars[(b & 0xf0) >> 4];
            chars[i++] = c0;
            char c1 = md5Chars[b & 0xf];
            chars[i++] = c1;
        }
        return chars;
    }

    /**
     * httpPost
     */
    public static JSONObject httpPost(String url, Map<String, Object> mapParam) {
        // post请求返回结果
        DefaultHttpClient httpClient = new DefaultHttpClient();
        JSONObject jsonResult = null;
        HttpPost httpPost = new HttpPost(url);
        try {
            List<NameValuePair> list = new ArrayList<NameValuePair>();
            Iterator iterator = mapParam.entrySet().iterator();
            while (iterator.hasNext()) {
                Map.Entry<String, Object> elem = (Map.Entry<String, Object>) iterator.next();
                list.add(new BasicNameValuePair(elem.getKey(), String.valueOf(elem.getValue())));
            }
            if (list.size() > 0) {
                UrlEncodedFormEntity entity = new UrlEncodedFormEntity(list, "utf-8");
                httpPost.setEntity(entity);
            }
            HttpResponse result = httpClient.execute(httpPost);
            url = URLDecoder.decode(url, "UTF-8");
            if (result.getStatusLine().getStatusCode() == 200) {
                String str = "";
                try {
                    str = EntityUtils.toString(result.getEntity());
                    jsonResult = JSONObject.parseObject(str);
                } catch (Exception e) {
                    log.error("post send error:" + url, e);
                }
            }
        } catch (IOException e) {
            log.error("post send fail:" + url, e);
        }
        return jsonResult;
    }