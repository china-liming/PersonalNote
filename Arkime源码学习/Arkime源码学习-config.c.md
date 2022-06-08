## moloch_config_init
### 描述
首先初始化hash表config.dontSaveTags，其中config.dontSaveTags为大小为11的string类型的hash表。
然后加载配置文件
```c
//config.c --924行
void moloch_config_init()
{
    HASH_INIT(s_, config.dontSaveTags, moloch_string_hash, moloch_string_cmp);

    moloch_config_load();

    if (config.debug) {
        LOG("maxFileSizeB: %" PRIu64, config.maxFileSizeB);
    }

    if (config.interface && !config.interface[0]) {
        CONFIGEXIT("The interface= is set in the config file (%s), but it is empty. :( You need to fix this before Arkime can continue.\n", config.configFile);
    }

    if (!config.interface && !config.pcapReadOffline) {
        CONFIGEXIT("Please set interface= in the [default] or [%s] section of the config file (%s) OR on the capture command line use either a pcap file (-r) or pcap directory (-R) switch. You need to fix this before Arkime can continue.\n", config.nodeName, config.configFile);
    }

    if (!config.pcapDir || !config.pcapDir[0]) {
        CONFIGEXIT("You must set a non empty pcapDir= in the config file(%s) to save files to. You need to fix this before Arkime can continue.\n", config.configFile);
    }

    if (!config.dryRun) {
        g_timeout_add_seconds( 10, moloch_config_reload_files, 0);
    }
}
```
## moloch_config_load
关于GKeyFile：
[glib函数学习# GKeyFile读取配置文件](glib函数学习.md)
创建keyfile 和 molochKeyFile 这两个指针。
读取 config.configFile 这个配置文件，如果没有成功或者出错，则打印错误信息。
如果config.debug 为 0，重新在keyfile中找debug的值。
读取keyfile的"includes"项，如果读取的不为空，则加载includes中的每一项配置文件。
读取keyfile中的"rotateIndex"组中的"daily"项，赋值给rotateIndex，如果为空，则报错。
根据 rotateIndex 的值赋值给 config.rotate 相应的枚举值。
···

```c
void moloch_config_load()
{
    gboolean  status;
    GError   *error = 0;
    GKeyFile *keyfile;
    int       i;

    keyfile = molochKeyFile = g_key_file_new();

    status = g_key_file_load_from_file(keyfile, config.configFile, G_KEY_FILE_NONE, &error);
    if (!status || error) 
    {
        CONFIGEXIT("Couldn't load config file (%s) %s\n", config.configFile, (error?error->message:""));
    }

    if (config.debug == 0) 
    {
        config.debug = moloch_config_int(keyfile, "debug", 0, 0, 128);
    }

    char **includes = moloch_config_str_list(keyfile, "includes", NULL);
    if (includes) 
    {
        moloch_config_load_includes(includes);
        g_strfreev(includes);
        //LOG("KEYFILE:\n%s", g_key_file_to_data(molochKeyFile, NULL, NULL));
    }

    char *rotateIndex       = moloch_config_str(keyfile, "rotateIndex", "daily");

    if (!rotateIndex) 
	{
        CONFIGEXIT("The rotateIndex= can't be empty in config file (%s)\n", config.configFile);
    } 
	else if (strcmp(rotateIndex, "hourly") == 0)
        config.rotate = MOLOCH_ROTATE_HOURLY;
    else if (strcmp(rotateIndex, "hourly2") == 0)
        config.rotate = MOLOCH_ROTATE_HOURLY2;
    else if (strcmp(rotateIndex, "hourly3") == 0)
        config.rotate = MOLOCH_ROTATE_HOURLY3;
    else if (strcmp(rotateIndex, "hourly4") == 0)
        config.rotate = MOLOCH_ROTATE_HOURLY4;
    else if (strcmp(rotateIndex, "hourly6") == 0)
        config.rotate = MOLOCH_ROTATE_HOURLY6;
    else if (strcmp(rotateIndex, "hourly8") == 0)
        config.rotate = MOLOCH_ROTATE_HOURLY8;
    else if (strcmp(rotateIndex, "hourly12") == 0)
        config.rotate = MOLOCH_ROTATE_HOURLY12;
    else if (strcmp(rotateIndex, "daily") == 0)
        config.rotate = MOLOCH_ROTATE_DAILY;
    else if (strcmp(rotateIndex, "weekly") == 0)
        config.rotate = MOLOCH_ROTATE_WEEKLY;
    else if (strcmp(rotateIndex, "monthly") == 0)
        config.rotate = MOLOCH_ROTATE_MONTHLY;
    else 
	{
        CONFIGEXIT("Unknown rotateIndex '%s' in config file (%s), see https://arkime.com/settings#rotateindex\n", rotateIndex, config.configFile);
    }
    g_free(rotateIndex);

    config.nodeClass        = moloch_config_str(keyfile, "nodeClass", NULL);
    gchar **tags            = moloch_config_str_list(keyfile, "dontSaveTags", NULL);
    if (tags) 
	{
        for (i = 0; tags[i]; i++) 
		{
            if (!(*tags[i]))
                continue;
            int num = 1;
            char *colon = strchr(tags[i], ':');
            if (colon) 
			{
                *colon = 0;
                num = atoi(colon+1);
                if (num < 1)
                    num = 1;
                if (num > 0xffff)
                    num = 0xffff;
            }
            moloch_string_add((MolochStringHash_t *)(char*)&config.dontSaveTags, tags[i], (gpointer)(long)num, TRUE);
        }
        g_strfreev(tags);
    }

    config.plugins          = moloch_config_str_list(keyfile, "plugins", NULL);
    config.rootPlugins      = moloch_config_str_list(keyfile, "rootPlugins", NULL);
    config.smtpIpHeaders    = moloch_config_str_list(keyfile, "smtpIpHeaders", NULL);

    if (config.smtpIpHeaders) 
	{
        for (i = 0; config.smtpIpHeaders[i]; i++) 
		{
            int len = strlen(config.smtpIpHeaders[i]);
            char *lower = g_ascii_strdown(config.smtpIpHeaders[i], len);
            g_free(config.smtpIpHeaders[i]);
            config.smtpIpHeaders[i] = lower;
            if (lower[len-1] == ':')
                lower[len-1] = 0;
        }
    }

    config.prefix           = moloch_config_str(keyfile, "prefix", "arkime_");
    int len = strlen(config.prefix);
    if (len > 0 && config.prefix[len - 1] != '_') 
	{
        char *tmp  = malloc(len + 2);
        memcpy(tmp, config.prefix, len);
        tmp[len] = '_';
        tmp[len+1] = 0;
        g_free(config.prefix);
        config.prefix = tmp;
    }

    config.elasticsearch    = moloch_config_str(keyfile, "elasticsearch", "localhost:9200");
    config.interface        = moloch_config_str_list(keyfile, "interface", NULL);
    config.pcapDir          = moloch_config_str_list(keyfile, "pcapDir", NULL);
    config.bpf              = moloch_config_str(keyfile, "bpf", NULL);
    config.yara             = moloch_config_str(keyfile, "yara", NULL);
    config.emailYara        = moloch_config_str(keyfile, "emailYara", NULL);
    config.rirFile          = moloch_config_str(keyfile, "rirFile", NULL);
    config.ouiFile          = moloch_config_str(keyfile, "ouiFile", NULL);
    config.geoLite2ASN      = moloch_config_str_list(keyfile, "geoLite2ASN", "/var/lib/GeoIP/GeoLite2-ASN.mmdb;/usr/share/GeoIP/GeoLite2-ASN.mmdb;" CONFIG_PREFIX "/etc/GeoLite2-ASN.mmdb");
    config.geoLite2Country  = moloch_config_str_list(keyfile, "geoLite2Country", "/var/lib/GeoIP/GeoLite2-Country.mmdb;/usr/share/GeoIP/GeoLite2-Country.mmdb;" CONFIG_PREFIX "/etc/GeoLite2-Country.mmdb");
    config.dropUser         = moloch_config_str(keyfile, "dropUser", NULL);
    config.dropGroup        = moloch_config_str(keyfile, "dropGroup", NULL);
    config.pluginsDir       = moloch_config_str_list(keyfile, "pluginsDir", NULL);
    config.parsersDir       = moloch_config_str_list(keyfile, "parsersDir", CONFIG_PREFIX "/parsers ; ./parsers ");
    config.caTrustFile      = moloch_config_str(keyfile, "caTrustFile", NULL);
    char *offlineRegex      = moloch_config_str(keyfile, "offlineFilenameRegex", "(?i)\\.(pcap|cap)$");

    config.offlineRegex     = g_regex_new(offlineRegex, 0, 0, &error);
    if (!config.offlineRegex || error) {
        CONFIGEXIT("Couldn't parse offlineRegex (%s) %s\n", offlineRegex, (error?error->message:""));
    }
    g_free(offlineRegex);

    config.pcapDirTemplate  = moloch_config_str(keyfile, "pcapDirTemplate", NULL);
    if (config.pcapDirTemplate && config.pcapDirTemplate[0] != '/') {
        CONFIGEXIT("pcapDirTemplate MUST start with a / '%s'\n", config.pcapDirTemplate);
    }

    config.pcapDirAlgorithm = moloch_config_str(keyfile, "pcapDirAlgorithm", "round-robin");
    if (strcmp(config.pcapDirAlgorithm, "round-robin") != 0
            && strcmp(config.pcapDirAlgorithm, "max-free-percent") != 0
            && strcmp(config.pcapDirAlgorithm, "max-free-bytes") != 0) {
        CONFIGEXIT("'%s' is not a valid value for pcapDirAlgorithm.  Supported algorithms are round-robin, max-free-percent, and max-free-bytes.\n", config.pcapDirAlgorithm);
    }

    config.maxFileSizeG          = moloch_config_double(keyfile, "maxFileSizeG", 4, 0.01, 1024);
    config.maxFileSizeB          = config.maxFileSizeG*1024LL*1024LL*1024LL;
    config.maxFileTimeM          = moloch_config_int(keyfile, "maxFileTimeM", 0, 0, 0xffff);
    config.timeouts[SESSION_ICMP]= moloch_config_int(keyfile, "icmpTimeout", 10, 1, 0xffff);
    config.timeouts[SESSION_UDP] = moloch_config_int(keyfile, "udpTimeout", 60, 1, 0xffff);
    config.timeouts[SESSION_TCP] = moloch_config_int(keyfile, "tcpTimeout", 60*8, 10, 0xffff);
    config.timeouts[SESSION_SCTP]= moloch_config_int(keyfile, "sctpTimeout", 60, 10, 0xffff);
    config.timeouts[SESSION_ESP] = moloch_config_int(keyfile, "espTimeout", 60*10, 10, 0xffff);
    config.timeouts[SESSION_OTHER] = 60*10;
    config.tcpSaveTimeout        = moloch_config_int(keyfile, "tcpSaveTimeout", 60*8, 10, 60*120);
    int maxStreams               = moloch_config_int(keyfile, "maxStreams", 1500000, 1, 16777215);
    config.maxPackets            = moloch_config_int(keyfile, "maxPackets", 10000, 1, 0xffff);
    config.maxPacketsInQueue     = moloch_config_int(keyfile, "maxPacketsInQueue", 200000, 10000, 5000000);
    config.dbBulkSize            = moloch_config_int(keyfile, "dbBulkSize", 200000, MOLOCH_HTTP_BUFFER_SIZE*2, 10000000);
    config.dbFlushTimeout        = moloch_config_int(keyfile, "dbFlushTimeout", 5, 1, 60*30);
    config.maxESConns            = moloch_config_int(keyfile, "maxESConns", 20, 3, 500);
    config.maxESRequests         = moloch_config_int(keyfile, "maxESRequests", 500, 10, 2500);
    config.logEveryXPackets      = moloch_config_int(keyfile, "logEveryXPackets", 50000, 1000, 0xffffffff);
    config.pcapBufferSize        = moloch_config_int(keyfile, "pcapBufferSize", 300000000, 100000, 0xffffffff);
    config.pcapWriteSize         = moloch_config_int(keyfile, "pcapWriteSize", 0x40000, 0x10000, 0x8000000);
    config.maxFreeOutputBuffers  = moloch_config_int(keyfile, "maxFreeOutputBuffers", 50, 0, 0xffff);
    config.fragsTimeout          = moloch_config_int(keyfile, "fragsTimeout", 60*8, 60, 0xffff);
    config.maxFrags              = moloch_config_int(keyfile, "maxFrags", 10000, 100, 0xffffff);
    config.snapLen               = moloch_config_int(keyfile, "snapLen", 16384, 1, MOLOCH_PACKET_MAX_LEN);
    config.maxMemPercentage      = moloch_config_int(keyfile, "maxMemPercentage", 100, 5, 100);
    config.maxReqBody            = moloch_config_int(keyfile, "maxReqBody", 256, 0, 0x7fff);

    config.packetThreads         = moloch_config_int(keyfile, "packetThreads", 1, 1, MOLOCH_MAX_PACKET_THREADS);

    config.logUnknownProtocols   = moloch_config_boolean(keyfile, "logUnknownProtocols", config.debug);
    config.logESRequests         = moloch_config_boolean(keyfile, "logESRequests", config.debug);
    config.logFileCreation       = moloch_config_boolean(keyfile, "logFileCreation", config.debug);
    config.logHTTPConnections    = moloch_config_boolean(keyfile, "logHTTPConnections", config.debug || !config.pcapReadOffline);
    config.parseSMTP             = moloch_config_boolean(keyfile, "parseSMTP", TRUE);
    config.parseSMTPHeaderAll    = moloch_config_boolean(keyfile, "parseSMTPHeaderAll", FALSE);
    config.parseSMB              = moloch_config_boolean(keyfile, "parseSMB", TRUE);
    config.ja3Strings            = moloch_config_boolean(keyfile, "ja3Strings", FALSE);
    config.parseDNSRecordAll     = moloch_config_boolean(keyfile, "parseDNSRecordAll", FALSE);
    config.parseQSValue          = moloch_config_boolean(keyfile, "parseQSValue", FALSE);
    config.parseCookieValue      = moloch_config_boolean(keyfile, "parseCookieValue", FALSE);
    config.parseHTTPHeaderRequestAll  = moloch_config_boolean(keyfile, "parseHTTPHeaderRequestAll", FALSE);
    config.parseHTTPHeaderResponseAll = moloch_config_boolean(keyfile, "parseHTTPHeaderResponseAll", FALSE);
    config.supportSha256         = moloch_config_boolean(keyfile, "supportSha256", FALSE);
    config.reqBodyOnlyUtf8       = moloch_config_boolean(keyfile, "reqBodyOnlyUtf8", TRUE);
    config.compressES            = moloch_config_boolean(keyfile, "compressES", FALSE);
    config.antiSynDrop           = moloch_config_boolean(keyfile, "antiSynDrop", TRUE);
    config.readTruncatedPackets  = moloch_config_boolean(keyfile, "readTruncatedPackets", FALSE);
    config.trackESP              = moloch_config_boolean(keyfile, "trackESP", FALSE);
    config.yaraEveryPacket       = moloch_config_boolean(keyfile, "yaraEveryPacket", TRUE);
    config.autoGenerateId        = moloch_config_boolean(keyfile, "autoGenerateId", FALSE);
    config.enablePacketLen       = moloch_config_boolean(NULL, "enablePacketLen", FALSE);
    config.enablePacketDedup     = moloch_config_boolean(NULL, "enablePacketDedup", FALSE);

    config.maxStreams[SESSION_TCP] = MAX(100, maxStreams/config.packetThreads*1.25);
    config.maxStreams[SESSION_UDP] = MAX(100, maxStreams/config.packetThreads/20);
    config.maxStreams[SESSION_SCTP] = MAX(100, maxStreams/config.packetThreads/20);
    config.maxStreams[SESSION_ICMP] = MAX(100, maxStreams/config.packetThreads/200);
    config.maxStreams[SESSION_ESP] = MAX(100, maxStreams/config.packetThreads/200);
    config.maxStreams[SESSION_OTHER] = MAX(100, maxStreams/config.packetThreads/20);

    gchar **saveUnknownPackets     = moloch_config_str_list(keyfile, "saveUnknownPackets", NULL);
    if (saveUnknownPackets) {
        for (i = 0; saveUnknownPackets[i]; i++) {
            char *s = saveUnknownPackets[i];

            if (strcmp(s, "all") == 0) {
                memset(&config.etherSavePcap, 0xff, sizeof(config.etherSavePcap));
                memset(&config.ipSavePcap, 0xff, sizeof(config.ipSavePcap));
            } else if (strcmp(s, "ip:all") == 0) {
                memset(&config.ipSavePcap, 0xff, sizeof(config.ipSavePcap));
            } else if (strcmp(s, "ether:all") == 0) {
                memset(&config.etherSavePcap, 0xff, sizeof(config.etherSavePcap));
            } else if (strncmp(s, "ip:", 3) == 0) {
                int n = atoi(s+3);
                if (n < 0 || n > 0xff)
                    CONFIGEXIT("Bad saveUnknownPackets ip value: %s", s);
                BIT_SET(n, config.ipSavePcap);
            } else if (strncmp(s, "-ip:", 4) == 0) {
                int n = atoi(s+4);
                if (n < 0 || n > 0xff)
                    CONFIGEXIT("Bad saveUnknownPackets -ip value: %s", s);
                BIT_CLR(n, config.ipSavePcap);
            } else if (strncmp(s, "ether:", 6) == 0) {
                int n = atoi(s+6);
                if (n < 0 || n > 0xffff)
                    CONFIGEXIT("Bad saveUnknownPackets ether value: %s", s);
                BIT_SET(n, config.etherSavePcap);
            } else if (strncmp(s, "-ether:", 7) == 0) {
                int n = atoi(s+7);
                if (n < 0 || n > 0xffff)
                    CONFIGEXIT("Bad saveUnknownPackets -ether value: %s", s);
                BIT_CLR(n, config.etherSavePcap);
            } else if (strcmp(s, "corrupt") == 0) {
                config.corruptSavePcap = 1;
            } else if (strcmp(s, "-corrupt") == 0) {
                config.corruptSavePcap = 0;
            } else {
                CONFIGEXIT("Not sure what saveUnknownPackets %s is", s);
            }
        }
        g_strfreev(saveUnknownPackets);
    }
}
```
## moloch_config_str
```c

gchar *moloch_config_str(GKeyFile *keyfile, char *key, char *d)
{
    char *result;

    if (!keyfile)
        keyfile = molochKeyFile;

    if (config.override && keyfile == molochKeyFile && (result = g_hash_table_lookup(config.override, key))) {
        if (result[0] == 0)
            result = NULL;
        else
            result = g_strdup(result);
    } 
    else if (g_key_file_has_key(keyfile, config.nodeName, key, NULL)) 
    {
        result = g_key_file_get_string(keyfile, config.nodeName, key, NULL);
    } 
    else if (config.nodeClass && g_key_file_has_key(keyfile, config.nodeClass, key, NULL)) 
    {
        result = g_key_file_get_string(keyfile, config.nodeClass, key, NULL);
    } 
    else if (g_key_file_has_key(keyfile, "default", key, NULL)) 
    {
        result = g_key_file_get_string(keyfile, "default", key, NULL);
    } 
    else if (d) 
    {
        result = g_strdup(d);
    } 
    else 
    {
        result = NULL;
    }

    if (result)
        g_strstrip(result);

    if (config.debug) 
    {
        LOG("%s=%s", key, result?result:"(null)");
    }
    return result;
}
```
## moloch_config_int
### 描述
···
config.override 是一个GHashTable类型的指针，如果它不为空，并且keyfile == molochKeyFile，并且在config.override中可以读取到key，就把result 转化为整型赋值给value 。
···
config.debug = moloch_config_int(keyfile, "debug", 0, 0, 128);

```c
uint32_t moloch_config_int(GKeyFile *keyfile, char *key, uint32_t d, uint32_t min, uint32_t max)
{
    char     *result;
    uint32_t  value = d;

    if (!keyfile)
        keyfile = molochKeyFile;

    if (config.override && keyfile == molochKeyFile && (result = g_hash_table_lookup(config.override, key))) 
    {
        value = atol(result);
    } 
    else if (g_key_file_has_key(keyfile, config.nodeName, key, NULL)) 
    {
        value = g_key_file_get_integer(keyfile, config.nodeName, key, NULL);
    } 
    else if (config.nodeClass && g_key_file_has_key(keyfile, config.nodeClass, key, NULL)) 
    {
        value = g_key_file_get_integer(keyfile, config.nodeClass, key, NULL);
    } 
    else if (g_key_file_has_key(keyfile, "default", key, NULL)) 
    {
        value = g_key_file_get_integer(keyfile, "default", key, NULL);
    }

    if (value < min) 
    {
        LOG ("INFO: Reseting %s since %u is less then the min %u", key, value, min);
        value = min;
    }
    if (value > max) 
    {
        LOG ("INFO: Reseting %s since %u is greater then the max %u", key, value, max);
        value = max;
    }

    if (config.debug) 
    {
        LOG("%s=%u", key, value);
    }

    return value;
}

```
## moloch_config_str_list
通过 moloch_config_raw_str_list 找到字符串列表。
将列表中的每一串字符串头部尾部去掉空白字符。
将列表中的空字符串全部放到列表末尾。
如果处在debug状态，则打印出字符串列表以';'分隔的字符串。
返回字符串列表。
其中有一处：
/\*Moved front of string, need to realloc so g_strfreev doesn't blow\*/
意思是移动了其中一个字符串的头部，就要重新复制一份，否则g_strfreev会blow。不懂。。
```c
gchar **moloch_config_str_list(GKeyFile *keyfile, char *key, char *d)
{
    gchar **strs = moloch_config_raw_str_list(keyfile, key, d);
    if (!strs) {
        if (config.debug) 
        {
            LOG("%s=(null)", key);
        }
        return strs;
    }

    int i, j;
    for (i = j = 0; strs[i]; i++) //去掉头部尾部的空白字符，将空字符串移动到列表尾部
    {
        char *str = strs[i];

        /* Remove leading and trailing spaces */
        while (isspace(*str))
            str++;
        g_strchomp(str);

        /* Empty string */
        if (*str == 0) {
            g_free(strs[i]);
            continue;
        }

        /* Moved front of string, need to realloc so g_strfreev doesn't blow */
        if (str != strs[i]) 
        {
            str = g_strdup(str);
            g_free(strs[i]);
        }

        /* Save string back */
        strs[j] = str;
        j++;
    }

    /* NULL anything at the end that was moved forward */
    for (; j < i; j++)
        strs[j] = NULL;

    if (config.debug) {
        gchar *str = g_strjoinv(";", strs);
        LOG("%s=%s", key, str);
        g_free(str);
    }
    return strs;
}


```
## moloch_config_raw_str_list
### 描述
如果config.override哈希表存在，并且keyfile 文件是 molochKeyFile，并且可以找到在config.override中找到key对应的值，则将通过key在keyfile中找到的值以';'分割开，赋值给result。
或者在keyfile中的config.nodeName组中查找key
或者在keyfile中的config.nodeClass组中查找key
或者在keyfile中的"default"组中查找key
或者如果d这个参数不为0且不为空，就把d以';'分割开，赋值给result。
如果上述都不满足，result为NULL。
返回result。
```c
gchar **moloch_config_raw_str_list(GKeyFile *keyfile, char *key, char *d)
{
    char   *hvalue;
    gchar **result;

    if (!keyfile)
        keyfile = molochKeyFile;

    if (config.override && keyfile == molochKeyFile && (hvalue = g_hash_table_lookup(config.override, key))) 
    {
        result = g_strsplit(hvalue, ";", 0);
    } 
    else if (g_key_file_has_key(keyfile, config.nodeName, key, NULL)) 
    {
        result = g_key_file_get_string_list(keyfile, config.nodeName, key, NULL, NULL);
    } 
    else if (config.nodeClass && g_key_file_has_key(keyfile, config.nodeClass, key, NULL)) 
    {
        result = g_key_file_get_string_list(keyfile, config.nodeClass, key, NULL, NULL);
    } 
    else if (g_key_file_has_key(keyfile, "default", key, NULL)) 
    {
        result = g_key_file_get_string_list(keyfile, "default", key, NULL, NULL);
    } 
    else if (d) 
    {
        result = g_strsplit(d, ";", 0);
    } 
    else 
    {
        result = NULL;
    }

    return result;
}

```
## moloch_config_load_includes
### 描述
逐个读取includes中的文件，放到临时变量keyFile中，如果读取失败则报错。
从keyFile中读取所有组，即\[group1\]，
在每一个组中读取所有key值，遍历每一行key，读取其值，如果值不为0，且没有报错，则将\[group1\] key=value 写入molochKeyFile。

```c
void moloch_config_load_includes(char **includes)
{
    int       i, g, k;

    for (i = 0; includes[i]; i++) 
    {
        GKeyFile *keyFile = g_key_file_new();
        GError *error = 0;
        gboolean status = g_key_file_load_from_file(keyFile, includes[i], G_KEY_FILE_NONE, &error);
        if (!status || error) 
        {
            CONFIGEXIT("Couldn't load config includes file (%s) %s\n", includes[i], (error?error->message:""));
        }

        gchar **groups = g_key_file_get_groups (keyFile, NULL);
        for (g = 0; groups[g]; g++) 
        {
            gchar **keys = g_key_file_get_keys (keyFile, groups[g], NULL, NULL);
            for (k = 0; keys[k]; k++) 
            {
                char *value = g_key_file_get_value(keyFile, groups[g], keys[k], NULL);
                if (value && !error) 
                {
                    g_key_file_set_value(molochKeyFile, groups[g], keys[k], value);
                    g_free(value);
                }
            }
            g_strfreev(keys);
        }
        g_strfreev(groups);
        g_key_file_free(keyFile);
    }
}

```
## moloch_config_reload_files
```c
gboolean moloch_config_reload_files (gpointer UNUSED(user_data))
{
    int             i, f;
    struct stat     sb[MOLOCH_CONFIG_FILES];

    for (i = 0; i < numFiles; i++) {
        int changed = 0;
        for (f = 0; f < files[i].num; f++) {
            if (stat(files[i].name[f], &sb[f]) != 0) {
                LOG("Couldn't stat %s file %s error %s", files[i].desc, files[i].name[f], strerror(errno));
                changed = 0;
                break;
            }

            if (sb[f].st_size <= 1) { // Ignore tiny files for reloads
                changed = 0;
                break;
            }

            if (sb[f].st_mtime > files[i].modify[f]) {
                if (files[i].size[f] != sb[f].st_size) {
                    files[i].size[f] = sb[f].st_size;
                    changed = 0;
                    break;
                }
                if (config.debug)
                    LOG("Changed %s %s", files[i].desc, files[i].name[f]);
                changed = 1;
            }
        }

        // Something was changed
        if (changed) {
            if (files[i].cbs)
                files[i].cbs(files[i].name);
            else
                files[i].cb(files[i].name[0]);

            for (f = 0; f < files[i].num; f++) {
                files[i].size[f] = 0;
                files[i].modify[f] = sb[f].st_mtime;
            }
        }
    }

    return TRUE;
}

```

