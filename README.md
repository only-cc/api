
    public static final Logger log = LoggerFactory.getLogger(ApiTest.class);

    // Users create schools or institutions, return to an organization or school ID (replace their own)
    final static int orgId = 30001;

    // APIKEY returned by a user creating a school or organization
    final static String secretKey = "6a3747a06dc211e7a8c4816a023b4ce6";

    // Interface address (replacing the requested request)
    final static String requestApiUrl = "http://cc.knoocrc.cn/cc-api/v1/session/list";

    final static char md5Chars[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

    @Test
    public void testApiRequest() throws Exception {
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
        Connection connection = getConnection(requestApiUrl, Connection.Method.POST, paramMap);
        String data = connection.execute().body();
        if (data == null) {
            Assert.fail();
        }
        JSONObject value = (JSONObject) JSONObject.parse(data);
        String responseOk = value.get("isSuccess").toString();
        if (responseOk == null || "false".equals(value.get("isSuccess").toString())) {
            Assert.fail();
        }
        log.info(data);
    }

### This is a fetching request parameter, then the key value is spelled into a string, and the above string is 16 bits of MD5HEX, for example

    private String generateSign(Map<String, Object> params) throws Exception {
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