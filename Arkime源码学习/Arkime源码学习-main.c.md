## 源码
```c

/* main.c  -- Initialization of components */

#include "moloch.h"
#include <pwd.h>
#include <grp.h>
#include <errno.h>
#include <sys/resource.h>
#ifdef _POSIX_MEMLOCK
#include <sys/mman.h>
#endif
#ifdef _POSIX_PRIORITY_SCHEDULING
#include <sched.h>
#endif
#include "pcap.h"
#include "arkimeconfig.h"

#ifndef BUILD_VERSION
#define BUILD_VERSION "unkn"
#endif

/******************************************************************************/
MolochConfig_t         config;
extern void           *esServer;
GMainLoop             *mainLoop;
char                  *moloch_char_to_hex = "0123456789abcdef"; /* don't change case */
unsigned char          moloch_char_to_hexstr[256][3];
unsigned char          moloch_hex_to_char[256][256];
uint32_t               hashSalt;

extern MolochWriterQueueLength moloch_writer_queue_length;
extern MolochPcapFileHdr_t     pcapFileHeader;

MOLOCH_LOCK_DEFINE(LOG);

/******************************************************************************/
LOCAL  gboolean showVersion    = FALSE;

#define FREE_LATER_SIZE 32768
LOCAL int freeLaterFront;
LOCAL int freeLaterBack;
typedef struct {
    void              *ptr;
    GDestroyNotify     cb;
    uint32_t           sec;
} MolochFreeLater_t;
MolochFreeLater_t  freeLaterList[FREE_LATER_SIZE];
MOLOCH_LOCK_DEFINE(freeLaterList);

/******************************************************************************/
gboolean moloch_debug_flag()
{
    config.debug++;
    config.quiet = 0;
    return TRUE;
}

/******************************************************************************/
gboolean moloch_cmdline_option(const gchar *option_name, const gchar *input, gpointer UNUSED(data), GError **UNUSED(error))
{
    char *equal = strchr(input, '=');
    if (!equal)
        CONFIGEXIT("The option %s requires a '=' in value '%s'", option_name, input);

    char *key = g_strndup(input, equal - input);
    char *value = g_strdup(equal + 1);

    if (!config.override) {
        config.override = g_hash_table_new_full(g_str_hash, g_str_equal, g_free, g_free);
    }
    g_hash_table_insert(config.override, key, value);

    return TRUE;
}
/******************************************************************************/

LOCAL  GOptionEntry entries[] =
{
    { "config",    'c',                    0, G_OPTION_ARG_FILENAME,       &config.configFile,    "Config file name, default '" CONFIG_PREFIX "/etc/config.ini'", NULL },
    { "pcapfile",  'r',                    0, G_OPTION_ARG_FILENAME_ARRAY, &config.pcapReadFiles, "Offline pcap file", NULL },
    { "pcapdir",   'R',                    0, G_OPTION_ARG_FILENAME_ARRAY, &config.pcapReadDirs,  "Offline pcap directory, all *.pcap files will be processed", NULL },
    { "monitor",   'm',                    0, G_OPTION_ARG_NONE,           &config.pcapMonitor,   "Used with -R option monitors the directory for closed files", NULL },
    { "packetcnt",   0,                    0, G_OPTION_ARG_INT,            &config.pktsToRead,    "Number of packets to read from each offline file", NULL},
    { "delete",      0,                    0, G_OPTION_ARG_NONE,           &config.pcapDelete,    "In offline mode delete files once processed, requires --copy", NULL },
    { "skip",      's',                    0, G_OPTION_ARG_NONE,           &config.pcapSkip,      "Used with -R option and without --copy, skip files already processed", NULL },
    { "reprocess",   0,                    0, G_OPTION_ARG_NONE,           &config.pcapReprocess, "In offline mode reprocess files, use the same files table entry", NULL },
    { "recursive",   0,                    0, G_OPTION_ARG_NONE,           &config.pcapRecursive, "When in offline pcap directory mode, recurse sub directories", NULL },
    { "node",      'n',                    0, G_OPTION_ARG_STRING,         &config.nodeName,      "Our node name, defaults to hostname.  Multiple nodes can run on same host", NULL },
    { "host",        0,                    0, G_OPTION_ARG_STRING,         &config.hostName,      "Override hostname, this is what remote viewers will use to connect", NULL },
    { "tag",       't',                    0, G_OPTION_ARG_STRING_ARRAY,   &config.extraTags,     "Extra tag to add to all packets, can be used multiple times", NULL },
    { "filelist",  'F',                    0, G_OPTION_ARG_STRING_ARRAY,   &config.pcapFileLists, "File that has a list of pcap file names, 1 per line", NULL },
    { "op",          0,                    0, G_OPTION_ARG_STRING_ARRAY,   &config.extraOps,      "FieldExpr=Value to set on all session, can be used multiple times", NULL},
    { "option",    'o',                    0, G_OPTION_ARG_CALLBACK,       moloch_cmdline_option, "Key=Value to override config.ini", NULL},
    { "version",   'v',                    0, G_OPTION_ARG_NONE,           &showVersion,          "Show version number", NULL },
    { "debug",     'd', G_OPTION_FLAG_NO_ARG, G_OPTION_ARG_CALLBACK,       moloch_debug_flag,     "Turn on all debugging", NULL },
    { "quiet",     'q',                    0, G_OPTION_ARG_NONE,           &config.quiet,         "Turn off regular logging", NULL },
    { "copy",        0,                    0, G_OPTION_ARG_NONE,           &config.copyPcap,      "When in offline mode copy the pcap files into the pcapDir from the config file", NULL },
    { "dryrun",      0,                    0, G_OPTION_ARG_NONE,           &config.dryRun,        "dry run, nothing written to databases or filesystem", NULL },
    { "flush",       0,                    0, G_OPTION_ARG_NONE,           &config.flushBetween,  "In offline mode flush streams between files", NULL },
    { "nospi",       0, G_OPTION_FLAG_HIDDEN, G_OPTION_ARG_NONE,           &config.noSPI,         "no SPI data written to ES", NULL },
    { "tests",       0, G_OPTION_FLAG_HIDDEN, G_OPTION_ARG_NONE,           &config.tests,         "Output test suite information", NULL },
    { "nostats",     0, G_OPTION_FLAG_HIDDEN, G_OPTION_ARG_NONE,           &config.noStats,       "Don't send node stats", NULL },
    { "insecure",    0,                    0, G_OPTION_ARG_NONE,           &config.insecure,      "Disable certificate verification for https calls", NULL },
    { "nolockpcap",  0,                    0, G_OPTION_ARG_NONE,           &config.noLockPcap,    "Don't lock offline pcap files (ie., allow deletion)", NULL },
    { "ignoreerrors",0, G_OPTION_FLAG_HIDDEN, G_OPTION_ARG_NONE,           &config.ignoreErrors,  "Ignore most errors and continue", NULL },
    { NULL,          0, 0,                                    0,           NULL, NULL, NULL }
};


/******************************************************************************/
void free_args()
{
    g_free(config.nodeName);
    g_free(config.hostName);
    g_free(config.configFile);
    if (config.pcapReadFiles)
        g_strfreev(config.pcapReadFiles);
    if (config.pcapFileLists)
        g_strfreev(config.pcapFileLists);
    if (config.pcapReadDirs)
        g_strfreev(config.pcapReadDirs);
    if (config.extraTags)
        g_strfreev(config.extraTags);
    if (config.extraOps)
        g_strfreev(config.extraOps);
}
/******************************************************************************/
void parse_args(int argc, char **argv)
{
    GError *error = NULL;
    GOptionContext *context;

    extern char *curl_version(void);
    extern char *pcre_version(void);
    extern const char *MMDB_lib_version(void);

    context = g_option_context_new ("- capture");
    g_option_context_add_main_entries (context, entries, NULL);
    if (!g_option_context_parse (context, &argc, &argv, &error))
    {
        g_print ("option parsing failed: %s\n", error->message);
        exit (1);
    }

    g_option_context_free(context);

    config.pcapReadOffline = (config.pcapReadFiles || config.pcapReadDirs || config.pcapFileLists);

    if (!config.configFile)
        config.configFile = g_strdup(CONFIG_PREFIX "/etc/config.ini");

    if (showVersion || config.debug) {
        printf("arkime-capture %s/%s session size=%d packet size=%d api=%d\n", PACKAGE_VERSION, BUILD_VERSION, (int)sizeof(MolochSession_t), (int)sizeof(MolochPacket_t), MOLOCH_API_VERSION);
    }

    if (showVersion) {
        printf("glib2: %u.%u.%u\n", glib_major_version, glib_minor_version, glib_micro_version);
        printf("libpcap: %s\n", pcap_lib_version());
        printf("curl: %s\n", curl_version());
        printf("pcre: %s\n", pcre_version());
        //printf("magic: %d\n", magic_version());
        printf("yara: %s\n", moloch_yara_version());
        printf("maxminddb: %s\n", MMDB_lib_version());

        exit(0);
    }

    if (glib_major_version !=  GLIB_MAJOR_VERSION ||
        glib_minor_version !=  GLIB_MINOR_VERSION ||
        glib_micro_version !=  GLIB_MICRO_VERSION) {

        LOG("WARNING - glib compiled %d.%d.%d vs linked %u.%u.%u",
                GLIB_MAJOR_VERSION, GLIB_MINOR_VERSION, GLIB_MICRO_VERSION,
                glib_major_version, glib_minor_version, glib_micro_version);
    }


    if (!config.hostName) {
        config.hostName = malloc(256);
        gethostname(config.hostName, 256);
        char *dot = strchr(config.hostName, '.');
        if (!dot) {
            char domainname[256];
            if (getdomainname(domainname, 255) == 0 && strlen(domainname) > 0 && strcmp(domainname, "(none)") != 0) {
                g_strlcat(config.hostName, ".", 255);
                g_strlcat(config.hostName, domainname, 255);
            } else {
                LOG("WARNING: gethostname doesn't return a fully qualified name and getdomainname failed, this may cause issues when viewing pcaps, use the --host option - %s", config.hostName);
            }
        }
        config.hostName[255] = 0;
    }

    if (!config.nodeName) {
        config.nodeName = g_strdup(config.hostName);
        char *dot = strchr(config.nodeName, '.');
        if (dot) {
            *dot = 0;
        }
    }

    if (config.debug) {
        LOG("debug = %d", config.debug);
        LOG("nodeName = %s", config.nodeName);
        LOG("hostName = %s", config.hostName);
    }

    if (config.tests) {
        config.dryRun = 1;
    }

    if (config.pcapSkip && config.copyPcap) {
        printf("Can't skip and copy pcap files\n");
        exit(1);
    }

    if (config.pcapDelete && !config.copyPcap) {
        printf("--delete requires --copy\n");
        exit(1);
    }

    if (config.copyPcap && !config.pcapReadOffline) {
        printf("--copy requires -r or -R\n");
        exit(1);
    }

    if (config.pcapMonitor && !config.pcapReadDirs) {
        printf("Must specify directories to monitor with -R\n");
        exit(1);
    }

    if (config.pcapReadFiles) {
        for (int i = 0; config.pcapReadFiles[i]; i++) {
            if (strcmp("-", config.pcapReadFiles[i]) == 0 && !config.copyPcap) {
                printf("-r - requires --copy be used\n");
                exit(1);
            }
        }
    }
}
/******************************************************************************/
void moloch_free_later(void *ptr, GDestroyNotify cb)
{
    struct timespec currentTime;
    clock_gettime(CLOCK_PROCESS_CPUTIME_ID, &currentTime);

    MOLOCH_LOCK(freeLaterList);
    if ((freeLaterBack + 1) % FREE_LATER_SIZE == freeLaterFront) {
        freeLaterList[freeLaterFront].cb(freeLaterList[freeLaterFront].ptr);
        freeLaterFront = (freeLaterFront + 1) % FREE_LATER_SIZE;
    }

    freeLaterList[freeLaterBack].sec = currentTime.tv_sec + 7;
    freeLaterList[freeLaterBack].ptr = ptr;
    freeLaterList[freeLaterBack].cb  = cb;
    freeLaterBack = (freeLaterBack + 1) % FREE_LATER_SIZE;
    MOLOCH_UNLOCK(freeLaterList);
}
/******************************************************************************/
LOCAL gboolean moloch_free_later_check (gpointer UNUSED(user_data))
{
    if (freeLaterFront == freeLaterBack)
        return TRUE;

    struct timespec currentTime;
    clock_gettime(CLOCK_PROCESS_CPUTIME_ID, &currentTime);
    MOLOCH_LOCK(freeLaterList);
    while (freeLaterFront != freeLaterBack &&
           freeLaterList[freeLaterFront].sec < currentTime.tv_sec) {
        freeLaterList[freeLaterFront].cb(freeLaterList[freeLaterFront].ptr);
        freeLaterFront = (freeLaterFront + 1) % FREE_LATER_SIZE;
    }
    MOLOCH_UNLOCK(freeLaterList);
    return TRUE;
}
/******************************************************************************/
LOCAL void moloch_free_later_init()
{
    g_timeout_add_seconds(1, moloch_free_later_check, 0);
}

/******************************************************************************/
void *moloch_size_alloc(int size, int zero)
{
    size += 8;
    void *mem = (zero?g_slice_alloc0(size):g_slice_alloc(size));
    memcpy(mem, &size, 4);
    return (char *)mem + 8;
}
/******************************************************************************/
int moloch_size_free(void *mem)
{
    int size;
    mem = (char *)mem - 8;

    memcpy(&size, mem, 4);
    g_slice_free1(size, mem);
    return size - 8;
}
/******************************************************************************/
void controlc(int UNUSED(sig))
{
    LOG("Control-C");
    signal(SIGINT, exit); // Double Control-C quits right away
    moloch_quit();
}
/******************************************************************************/
void terminate(int UNUSED(sig))
{
    LOG("Terminate");
    moloch_quit();
}
/******************************************************************************/
void reload(int UNUSED(sig))
{
    moloch_plugins_reload();
}
/******************************************************************************/
uint32_t moloch_get_next_prime(uint32_t v)
{
    static uint32_t primes[] = {1009, 10007, 49999, 99991, 199799, 400009, 500009, 732209,
                                1092757, 1299827, 1500007, 1987411, 2999999, 4000037,
                                5000011, 6000011, 7000003, 8000009, 9000011, 10000019,
                                11000027, 12000017, 13000027, 14000029, 15000017, 16000057,
                                17000023, 18000041, 19000013, 20000003, 21000037, 22000001,
                                0};

    int p;
    for (p = 0; primes[p]; p++) {
        if (primes[p] > v)
            return primes[p];
    }
    return primes[p-1];
}
/******************************************************************************/
//https://graphics.stanford.edu/~seander/bithacks.html#RoundUpPowerOf2
uint32_t moloch_get_next_powerof2(uint32_t v)
{
    v--;
    v |= v >> 1;
    v |= v >> 2;
    v |= v >> 4;
    v |= v >> 8;
    v |= v >> 16;
    v++;
    return v;
}
/******************************************************************************/
unsigned char *moloch_js0n_get(unsigned char *data, uint32_t len, char *key, uint32_t *olen)
{
    uint32_t key_len = strlen(key);
    int      i;
    uint32_t out[4*100]; // Can have up to 100 elements at any level

    *olen = 0;
    int rc;
    if ((rc = js0n(data, len, out, sizeof(out))) != 0) {
        LOG("ERROR - Parse error %d for >%s< in >%.*s<\n", rc, key, len, data);
        fflush(stdout);
        return 0;
    }

    for (i = 0; out[i]; i+= 4) {
        if (out[i+1] == key_len && memcmp(key, data + out[i], key_len) == 0) {
            *olen = out[i+3];
            return data + out[i+2];
        }
    }
    return 0;
}
/******************************************************************************/
char *moloch_js0n_get_str(unsigned char *data, uint32_t len, char *key)
{
    uint32_t           value_len;
    unsigned char     *value = 0;

    value = moloch_js0n_get(data, len, key, &value_len);
    if (!value)
        return NULL;
    return g_strndup((gchar*)value, value_len);
}
/******************************************************************************/
const char *moloch_memstr(const char *haystack, int haysize, const char *needle, int needlesize)
{
    const char *p;
    while (haysize >= needlesize && (p = memchr(haystack, *needle, haysize - needlesize + 1))) {
        if (memcmp(p, needle, needlesize) == 0)
            return p;
        haysize -= (p - haystack + 1);
        haystack = p+1;
    }
    return NULL;
}
/******************************************************************************/
const char *moloch_memcasestr(const char *haystack, int haysize, const char *needle, int needlesize)
{
    const char *p;
    const char *end = haystack + haysize - needlesize;
    int i;

    for (p = haystack; p <= end; p++)
    {
        for (i = 0; i < needlesize; i++) {
            if (tolower(p[i]) != needle[i]) {
                goto memcasestr_outer;
            }
        }
        return p;

        memcasestr_outer: ;
    }
    return NULL;
}
/******************************************************************************/
gboolean moloch_string_add(void *hashv, char *string, gpointer uw, gboolean copy)
{
    MolochStringHash_t *hash = hashv;
    MolochString_t *hstring;

    HASH_FIND(s_, *hash, string, hstring);
    if (hstring) {
        hstring->uw = uw;
        return FALSE;
    }

    hstring = MOLOCH_TYPE_ALLOC0(MolochString_t);
    if (copy) {
        hstring->str = g_strdup(string);
    } else {
        hstring->str = string;
    }
    hstring->len = strlen(string);
    hstring->uw = uw;
    HASH_ADD(s_, *hash, hstring->str, hstring);
    return TRUE;
}
/******************************************************************************/
SUPPRESS_UNSIGNED_INTEGER_OVERFLOW
uint32_t moloch_string_hash(const void *key)
{
    unsigned char *p = (unsigned char *)key;
    uint32_t n = 0;
    while (*p) {
        n = (n << 5) - n + *p;
        p++;
    }

    n ^= hashSalt;

    return n;
}
/******************************************************************************/
SUPPRESS_UNSIGNED_INTEGER_OVERFLOW
uint32_t moloch_string_hash_len(const void *key, int len)
{
    unsigned char *p = (unsigned char *)key;
    uint32_t n = 0;
    while (len) {
        n = (n << 5) - n + *p;
        p++;
        len--;
    }

    n ^= hashSalt;

    return n;
}

/******************************************************************************/
int moloch_string_cmp(const void *keyv, const void *elementv)
{
    char *key = (char*)keyv;
    MolochString_t *element = (MolochString_t *)elementv;

    return strcmp(key, element->str) == 0;
}
/******************************************************************************/
int moloch_string_ncmp(const void *keyv, const void *elementv)
{
    char *key = (char*)keyv;
    MolochString_t *element = (MolochString_t *)elementv;

    return strncmp(key, element->str, element->len) == 0;
}
/******************************************************************************/
uint32_t moloch_int_hash(const void *key)
{
    return (uint32_t)((long)key);
}

/******************************************************************************/
int moloch_int_cmp(const void *keyv, const void *elementv)
{
    uint32_t key = (uint32_t)((long)keyv);
    MolochInt_t *element = (MolochInt_t *)elementv;

    return key == element->i_hash;
}
/******************************************************************************/
typedef struct {
    MolochWatchFd_func  func;
    gpointer            data;
} MolochWatchFd_t;

/******************************************************************************/
LOCAL void moloch_gio_destroy(gpointer data)
{
    g_free(data);
}
/******************************************************************************/
LOCAL gboolean moloch_gio_invoke(GIOChannel *source, GIOCondition condition, gpointer data)
{
    MolochWatchFd_t *watch = data;

    return watch->func(g_io_channel_unix_get_fd(source), condition, watch->data);
}

/******************************************************************************/
gint moloch_watch_fd(gint fd, GIOCondition cond, MolochWatchFd_func func, gpointer data)
{

    MolochWatchFd_t *watch = g_new0(MolochWatchFd_t, 1);
    watch->func = func;
    watch->data = data;

    GIOChannel *channel = g_io_channel_unix_new(fd);

    gint id =  g_io_add_watch_full(channel, G_PRIORITY_DEFAULT, cond, moloch_gio_invoke, watch, moloch_gio_destroy);

    g_io_channel_unref(channel);
    return id;
}

/******************************************************************************/
void moloch_drop_privileges()
{
    if (getuid() != 0)
        return;

    if (config.dropGroup) {
        struct group   *grp;
        grp = getgrnam(config.dropGroup);
        if (!grp) {
            CONFIGEXIT("Group '%s' not found", config.dropGroup);
        }

        if (setgid(grp->gr_gid) != 0) {
            CONFIGEXIT("Couldn't change group - %s", strerror(errno));
        }
    }

    if (config.dropUser) {
        struct passwd   *usr;
        usr = getpwnam(config.dropUser);
        if (!usr) {
            CONFIGEXIT("User '%s' not found", config.dropUser);
        }

        if (setuid(usr->pw_uid) != 0) {
            CONFIGEXIT("Couldn't change user - %s", strerror(errno));
        }
    }


}
/******************************************************************************/
LOCAL  MolochCanQuitFunc  canQuitFuncs[20];
LOCAL  const char        *canQuitNames[20];
LOCAL  int                canQuitFuncsNum;

void moloch_add_can_quit (MolochCanQuitFunc func, const char *name)
{
    if (canQuitFuncsNum >= 20) {
        LOGEXIT("ERROR - Can't add canQuitFunc");
    }
    canQuitFuncs[canQuitFuncsNum] = func;
    canQuitNames[canQuitFuncsNum] = name;
    canQuitFuncsNum++;
}
/******************************************************************************/
/*
 * Don't actually end main loop until all the various pieces are done
 */
gboolean moloch_quit_gfunc (gpointer UNUSED(user_data))
{
LOCAL gboolean readerExit   = TRUE;
LOCAL gboolean writerExit   = TRUE;

// On the first run shutdown reader and sessions
    if (readerExit) {
        readerExit = FALSE;
        if (moloch_reader_stop)
            moloch_reader_stop();
        moloch_packet_exit();
        moloch_session_exit();
        if (config.debug)
            LOG("Read exit finished");
        return G_SOURCE_CONTINUE;
    }

// Wait for all the can quits to signal all clear
    int i;
    for (i = 0; i < canQuitFuncsNum; i++) {
        int val = canQuitFuncs[i]();
        if (val != 0) {
            if (config.debug && canQuitNames[i]) {
                LOG ("Can't quit, %s is %d", canQuitNames[i], val);
            }
            return G_SOURCE_CONTINUE;
        }
    }

// Once all clear stop the writer and wait for all clears again
    if (writerExit) {
        writerExit = FALSE;
        if (!config.dryRun && config.copyPcap) {
            moloch_writer_exit();
            if (config.debug)
                LOG("Write exit finished");
            return G_SOURCE_CONTINUE;
        }
    }

// Can quit the main loop now
    g_main_loop_quit(mainLoop);
    return G_SOURCE_REMOVE;
}
/******************************************************************************/
void moloch_quit()
{
    if (config.quitting)
        return;

    if (config.debug)
        LOG("Quitting");

    config.quitting = TRUE;
    g_timeout_add(100, moloch_quit_gfunc, 0);
}
/******************************************************************************/
/*
 * Don't actually init nids/pcap until all the pre tags are loaded.
 * TRUE - call again in 1ms
 * FALSE - don't call again
 */
gboolean moloch_ready_gfunc (gpointer UNUSED(user_data))
{
    if (moloch_http_queue_length(esServer))
        return TRUE;

    if (config.debug)
        LOG("maxField = %d", config.maxField);

    if (config.pcapReadOffline) {
        if (config.dryRun || !config.copyPcap) {
            moloch_writers_start("inplace");
        } else {
            moloch_writers_start(NULL);
        }

    } else {
        if (config.dryRun) {
            moloch_writers_start("null");
        } else {
            moloch_writers_start(NULL);
        }
    }
    moloch_readers_start();
    if (!config.pcapReadOffline && (pcapFileHeader.dlt == DLT_NULL || pcapFileHeader.snaplen == 0))
        LOGEXIT("ERROR - Reader didn't call moloch_packet_set_dltsnap");
    return FALSE;
}
/******************************************************************************/
void moloch_hex_init()
{
    int i, j;
    for (i = 0; i < 16; i++) {
        for (j = 0; j < 16; j++) {
            moloch_hex_to_char[(unsigned char)moloch_char_to_hex[i]][(unsigned char)moloch_char_to_hex[j]] = i << 4 | j;
            moloch_hex_to_char[toupper(moloch_char_to_hex[i])][(unsigned char)moloch_char_to_hex[j]] = i << 4 | j;
            moloch_hex_to_char[(unsigned char)moloch_char_to_hex[i]][toupper(moloch_char_to_hex[j])] = i << 4 | j;
            moloch_hex_to_char[toupper(moloch_char_to_hex[i])][toupper(moloch_char_to_hex[j])] = i << 4 | j;
        }
    }

    for (i = 0; i < 256; i++) {
        moloch_char_to_hexstr[i][0] = moloch_char_to_hex[(i >> 4) & 0xf];
        moloch_char_to_hexstr[i][1] = moloch_char_to_hex[i & 0xf];
    }
}

/*
void moloch_sched_init()
{
#ifdef _POSIX_PRIORITY_SCHEDULING
    struct sched_param sp;
    sp.sched_priority = sched_get_priority_max(SCHED_FIFO);
    int result = sched_setscheduler(0, SCHED_FIFO, &sp);
    if (result != 0) {
        LOG("WARNING: Failed to change to FIFO scheduler - %s", strerror(errno));
    } else if (config.debug) {
        LOG("Changed to FIFO scheduler with priority %d", sp.sched_priority);
    }
#endif
}
*/
/******************************************************************************/
void moloch_mlockall_init()
{
#ifdef _POSIX_MEMLOCK
    struct rlimit l;
    getrlimit(RLIMIT_MEMLOCK, &l);
    if (l.rlim_max != RLIM_INFINITY && l.rlim_max < 4000000000LL) {
        LOG("WARNING: memlock in limits.conf must be unlimited or at least 4000000, currently %lu", (unsigned long)l.rlim_max/1024);
        return;
    }

    if (l.rlim_cur != l.rlim_max) {
        if (config.debug)
            LOG("Setting memlock soft to %lu", (unsigned long)l.rlim_max);
        l.rlim_cur = l.rlim_max;
        setrlimit(RLIMIT_MEMLOCK, &l);
    }

    int result = mlockall(MCL_FUTURE | MCL_CURRENT);
    if (result != 0) {
        LOG("WARNING: Failed to mlockall - %s", strerror(errno));
    } else if (config.debug) {
        LOG("mlockall with max of %lu", (unsigned long)l.rlim_max);
    }
#endif
}
/******************************************************************************/
#ifdef FUZZLOCH

/* This replaces main for libFuzzer.  Basically initialized everything like main
 * would for starting up and set some important settings.  Must be run from tests
 * directory, and config.test.ini will be loaded for fuzz node.
 */

MolochPacketBatch_t   batch;
uint64_t              fuzzloch_sessionid = 0;

int LLVMFuzzerInitialize(int *UNUSED(argc), char ***UNUSED(argv))
{
    config.configFile = g_strdup("config.test.ini");
    config.dryRun = 1;
    config.pcapReadOffline = 1;
    config.hostName = strdup("fuzz.example.com");
    config.nodeName = strdup("fuzz");

    hashSalt = 0;
    pcapFileHeader.dlt = DLT_EN10MB;

    moloch_free_later_init();
    moloch_hex_init();
    moloch_config_init();
    moloch_writers_init();
    moloch_writers_start("null");
    moloch_readers_init();
    moloch_readers_set("null");
    moloch_plugins_init();
    moloch_field_init();
    moloch_http_init();
    moloch_db_init();
    moloch_packet_init();
    moloch_config_load_local_ips();
    moloch_config_load_packet_ips();
    moloch_yara_init();
    moloch_parsers_init();
    moloch_session_init();
    moloch_plugins_load(config.plugins);
    moloch_rules_init();
    moloch_packet_batch_init(&batch);
    return 0;
}

/******************************************************************************/
/* In libFuzzer mode this is called for each packet.
 * There are no packet threads in fuzz mode, and the batch call will actually
 * process the packet.  The current time just increases for each packet.
 */
int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    static uint64_t       ts = 10000;
    BSB                   bsb;

    BSB_INIT(bsb, data, size);

    fuzzloch_sessionid++;

    while (BSB_REMAINING(bsb) > 3 && !BSB_IS_ERROR(bsb)) {
        uint16_t len = 0;
        BSB_IMPORT_u16(bsb, len);

        if (len == 0 || len > BSB_REMAINING(bsb))
            break;

        u_char *ptr = 0;
        BSB_IMPORT_ptr(bsb, ptr, len);

        if (!ptr || BSB_IS_ERROR(bsb))
            break;

        // LOG("Packet %llu %d", fuzzloch_sessionid, len);

        MolochPacket_t *packet = MOLOCH_TYPE_ALLOC0(MolochPacket_t);
        packet->pktlen         = len;
        packet->pkt            = ptr;
        packet->ts.tv_sec      = ts >> 4;
        packet->ts.tv_usec     = ts & 0x8;
        ts++;
        packet->readerFilePos  = 0;
        packet->readerPos      = 0;

        // In FUZZ mode batch will actually process it
        moloch_packet_batch(&batch, packet);
    }

    return 0;
}

#else
int main(int argc, char **argv)
{
    signal(SIGHUP, reload);
    signal(SIGINT, controlc);
    signal(SIGTERM, terminate);
    signal(SIGUSR1, exit);
    signal(SIGCHLD, SIG_IGN);

    mainLoop = g_main_loop_new(NULL, FALSE);

    hashSalt = (uint32_t)time(NULL);

    parse_args(argc, argv);
    if (config.debug)
        LOG("THREAD %p", (gpointer)pthread_self());

    if (config.insecure)
        LOG("\n\nDON'T DO IT!!!! `--insecure` is a bad idea\n\n");

    moloch_free_later_init();
    moloch_hex_init();
    moloch_config_init();
    arkime_dedup_init();
    moloch_writers_init();
    moloch_readers_init();
    moloch_plugins_init();
    moloch_plugins_load(config.rootPlugins);
    if (config.pcapReadOffline)
        moloch_readers_set("libpcap-file");
    else
        moloch_readers_set(NULL);
    if (!config.pcapReadOffline) {
        moloch_drop_privileges();
        config.copyPcap = 1;
        moloch_mlockall_init();
    }
    moloch_field_init();
    moloch_http_init();
    moloch_db_init();
    moloch_packet_init();
    moloch_config_load_local_ips();
    moloch_config_load_packet_ips();
    moloch_yara_init();
    moloch_parsers_init();
    moloch_session_init();
    moloch_plugins_load(config.plugins);
    moloch_rules_init();
    g_timeout_add(1, moloch_ready_gfunc, 0);

    g_main_loop_run(mainLoop);

    if (!config.pcapReadOffline || config.debug)
        LOG("Final cleanup");
    moloch_plugins_exit();
    moloch_parsers_exit();
    moloch_db_exit();
    moloch_http_exit();
    moloch_field_exit();
    moloch_readers_exit();
    arkime_dedup_exit();
    moloch_config_exit();
    moloch_rules_exit();
    moloch_yara_exit();

    g_main_loop_unref(mainLoop);

    free_args();
    exit(0);
}
#endif


```

## main()
```c
int main(int argc, char **argv)
{
    signal(SIGHUP, reload);//退出信号
    signal(SIGINT, controlc);//中断信号，如 ctrl-C，通常由用户生成。
    signal(SIGTERM, terminate); //发送给本程序的终止请求信号。
    signal(SIGUSR1, exit);      //用户自定义
    signal(SIGCHLD, SIG_IGN);    //进程Terminate或Stop时候，SIGCHLD会发送给它的父进程。缺省情况下该Signal会被忽略

    mainLoop = g_main_loop_new(NULL, FALSE);

    hashSalt = (uint32_t)time(NULL);

    parse_args(argc, argv);
    if (config.debug)
        LOG("THREAD %p", (gpointer)pthread_self());

    if (config.insecure)
        LOG("\n\nDON'T DO IT!!!! `--insecure` is a bad idea\n\n");

    moloch_free_later_init();
    moloch_hex_init();
    moloch_config_init();
    arkime_dedup_init();
    moloch_writers_init();
    moloch_readers_init();
    moloch_plugins_init();
    moloch_plugins_load(config.rootPlugins);
    if (config.pcapReadOffline)
        moloch_readers_set("libpcap-file");
    else
        moloch_readers_set(NULL);
    if (!config.pcapReadOffline) {
        moloch_drop_privileges();
        config.copyPcap = 1;
        moloch_mlockall_init();
    }
    moloch_field_init();
    moloch_http_init();
    moloch_db_init();
    moloch_packet_init();
    moloch_config_load_local_ips();
    moloch_config_load_packet_ips();
    moloch_yara_init();
    moloch_parsers_init();
    moloch_session_init();
    moloch_plugins_load(config.plugins);
    moloch_rules_init();
    g_timeout_add(1, moloch_ready_gfunc, 0);

    g_main_loop_run(mainLoop);

    if (!config.pcapReadOffline || config.debug)
        LOG("Final cleanup");
    moloch_plugins_exit();
    moloch_parsers_exit();
    moloch_db_exit();
    moloch_http_exit();
    moloch_field_exit();
    moloch_readers_exit();
    arkime_dedup_exit();
    moloch_config_exit();
    moloch_rules_exit();
    moloch_yara_exit();

    g_main_loop_unref(mainLoop);

    free_args();
    exit(0);
}
```
### SIGHUP
[[C语言函数学习#signal]]
原文链接：https://blog.csdn.net/z_ryan/article/details/80952498
在介绍SIGHUP信号之前，先来了解两个概念：进程组和会话。
进程组
　　进程组就是一系列相互关联的进程集合，系统中的每一个进程也必须从属于某一个进程组；每个进程组中都会有一个唯一的 ID(process group id)，简称 PGID；PGID 一般等同于进程组的创建进程的 Process ID，而这个进进程一般也会被称为进程组先导(process group leader)，同一进程组中除了进程组先导外的其他进程都是其子进程；
　　进程组的存在，方便了系统对多个相关进程执行某些统一的操作，例如，我们可以一次性发送一个信号量给同一进程组中的所有进程。
会话
　　会话（session）是一个若干进程组的集合，同样的，系统中每一个进程组也都必须从属于某一个会话；一个会话只拥有最多一个控制终端（也可以没有），该终端为会话中所有进程组中的进程所共用。一个会话中前台进程组只会有一个，只有其中的进程才可以和控制终端进行交互；除了前台进程组外的进程组，都是后台进程组；和进程组先导类似，会话中也有会话先导(session leader)的概念，用来表示建立起到控制终端连接的进程。在拥有控制终端的会话中，session leader 也被称为控制进程(controlling process)，一般来说控制进程也就是登入系统的 shell 进程(login shell)；
SIGHUP信号的触发及默认处理
　　在对会话的概念有所了解之后，我们现在开始==正式介绍一下SIGHUP信号==，SIGHUP 信号在用户终端连接(正常或非正常)结束时发出, 通常是在终端的控制进程结束时, 通知同一session内的各个作业, 这时它们与控制终端不再关联. 系统对SIGHUP信号的默认处理是终止收到该信号的进程。所以若程序中没有捕捉该信号，当收到该信号时，进程就会退出。
SIGHUP会在以下3种情况下被发送给相应的进程：
- 1、终端关闭时，该信号被发送到session首进程以及作为job提交的进程（即用 & 符号提交的进程）；
- 2、session首进程退出时，该信号被发送到该session中的前台进程组中的每一个进程；
- 3、若父进程退出导致进程组成为孤儿进程组，且该进程组中有进程处于停止状态（收到SIGSTOP或SIGTSTP信号），该信号会被发送到该进程组中的每一个进程。

## moloch_string_add
在moloch.h中812行进行声明，main.c中进行定义

### 描述
在hashv中查找string，如果找到，则将uw赋值给找到的hsting->uw，并返回FALSE。
如果没有找到，则新分配一个MolochString_t类型的hstring。
如果copy为TRUE，则将string复制一份，并赋值给hstring->str
否则，直接将string赋值给hstring->str
将string的长度赋值给hstring->len
将uw赋值给hstring->uw
向hashv哈希表中添加hstring这个元素

### 源码声明
在moloch.h中
```c
gboolean moloch_string_add(void *hashv, char *string, gpointer uw, gboolean copy);
```

### 参数
void *hashv,
char *string, 
gpointer uw, 
gboolean copy

### 返回
g

### 源码定义
429行
```c
gboolean moloch_string_add(void *hashv, char *string, gpointer uw, gboolean copy)
{
    MolochStringHash_t *hash = hashv;
    MolochString_t *hstring;

    HASH_FIND(s_, *hash, string, hstring);
    if (hstring) 
	{
        hstring->uw = uw;
        return FALSE;
    }

    hstring = MOLOCH_TYPE_ALLOC0(MolochString_t);
    if (copy) 
	{
        hstring->str = g_strdup(string);
    } 
	else 
	{
        hstring->str = string;
    }
    hstring->len = strlen(string);
    hstring->uw = uw;
    HASH_ADD(s_, *hash, hstring->str, hstring);
    return TRUE;
}
```
关于MolochStringHash_t：[[Arkime源码学习-moloch.h#moloch_string]]
### 实例
#### moloch_config_load()
config.c中 moloch_config_load()函数中443行：
[[Arkime源码学习-config.c#moloch_config_load]]
```c
moloch_string_add((MolochStringHash_t *)(char*)&config.dontSaveTags, tags[i], (gpointer)(long)num, TRUE);
```
#### moloch_readers_add()
readers.c中moloch_readers_add()59行
[[Arkime源码学习-readers.c#moloch_readers_add]]
```c
moloch_string_add(&readersHash, name, func, TRUE);
```
#### moloch_writers_add()
在writers.c中moloch_writers_add()函数中52行
[[Arkime源码学习-writers.c#moloch_writers_add]]
```c
moloch_string_add(&writersHash, name, func, TRUE);
```

## moloch_add_can_quit
```c
//main.c --585
void moloch_add_can_quit (MolochCanQuitFunc func, const char *name)
{
    if (canQuitFuncsNum >= 20) {
        LOGEXIT("ERROR - Can't add canQuitFunc");
    }
    canQuitFuncs[canQuitFuncsNum] = func;
    canQuitNames[canQuitFuncsNum] = name;
    canQuitFuncsNum++;
}
//实例：
moloch_add_can_quit((MolochCanQuitFunc)moloch_writer_queue_length, "writer queue length");
```

## moloch_string_hash
### 描述
对字符串中的每一位都参与运算`n = (n << 5) - n + *p;`，最后将得到的n与`hashSalt`进行异或操作，并将结果返回。
### 源码
```c
SUPPRESS_UNSIGNED_INTEGER_OVERFLOW
uint32_t moloch_string_hash(const void *key)
{
    unsigned char *p = (unsigned char *)key;
    uint32_t n = 0;
    while (*p) {
        n = (n << 5) - n + *p;
        p++;
    }

    n ^= hashSalt;

    return n;
}
```

## moloch_free_later_init
### 描述
每一秒钟执行 moloch_free_later_check函数
[[glib函数学习#g_timeout_add_seconds]]
### 源码
```c
LOCAL void moloch_free_later_init()
{
    g_timeout_add_seconds(1, moloch_free_later_check, 0);
}
```

### moloch_free_later_check
#### 描述
如果数组为空，则返回TRUE。
获取本进程运行时间，存储在currentTime中。
锁住freeLaterList，
如果freeLaterList不为空，并且freeLaterList 中 的freeLaterFront元素，即首部元素的秒数小于本进程运行的秒数，就一直执行下面的语句：
- 调用freeLaterList\[freeLaterFront]中的元素销毁函数cb，并且销毁函数的参数是freeLaterList\[freeLaterFront]中的ptr。
- freeLaterFront 加一，并对长度取余。
解锁freeLaterList。
返回TRUE

```c
LOCAL gboolean moloch_free_later_check (gpointer UNUSED(user_data))
{
    if (freeLaterFront == freeLaterBack)
        return TRUE;

    struct timespec currentTime;
    clock_gettime(CLOCK_PROCESS_CPUTIME_ID, &currentTime);
    MOLOCH_LOCK(freeLaterList);
    while (freeLaterFront != freeLaterBack &&
           freeLaterList[freeLaterFront].sec < currentTime.tv_sec) {
        freeLaterList[freeLaterFront].cb(freeLaterList[freeLaterFront].ptr);
        freeLaterFront = (freeLaterFront + 1) % FREE_LATER_SIZE;
    }
    MOLOCH_UNLOCK(freeLaterList);
    return TRUE;
}
```

### freeLaterList
```c
typedef struct {
    void              *ptr;
    GDestroyNotify     cb;
    uint32_t           sec;
} MolochFreeLater_t;
MolochFreeLater_t  freeLaterList[FREE_LATER_SIZE];
```

静态锁的定义
```c
#define MOLOCH_LOCK_DEFINE(var)  pthread_mutex_t var##_mutex = PTHREAD_MUTEX_INITIALIZER

MOLOCH_LOCK_DEFINE(freeLaterList);
//扩展到
pthread_mutex_t freeLaterList_mutex = PTHREAD_MUTEX_INITIALIZER;
//这是C语言静态锁的定义
```

```c
#define MOLOCH_LOCK(var)                pthread_mutex_lock(&var##_mutex)
MOLOCH_LOCK(freeLaterList);
//扩展为
pthread_mutex_lock(&freeLaterList_mutex);
```

GDestroyNotify
定义：
```c
void (* GDestroyNotify) (gpointer data)
```
Specifies the type of function which is called when a data element is destroyed. It is passed the pointer to the data element and should free any memory and resources allocated for it.
指定数据元素销毁时调用的函数类型。它被传递到数据元素的指针，并且应该释放为它分配的任何内存和资源。
参数：
- data	gpointer
 The data element.

## moloch_hex_init
### 描述
- 第一步
穷举所有的两位的十六进制数，在moloch_hex_to_char\[256]\[256]这个256\*256的数组中给每一个两位的十六进制数赋一个值，这个值是i右移四位，再与 j 或运算。
- 第二步
i 遍历0~256，将moloch_char_to_hexstr\[256]\[3]中的\[i]\[0]赋值为 i 的最后一个字节的前4个比特，将\[i]\[1]赋值为i的最后一个字节的后4个比特。如，i 为255（0000 0000 0000 0000 0000 0000 1111 1111）时，将最后一字节的前4比特1111赋值给\[i]\[0]，将最后一字节的后4比特1111赋值给\[i]\[1]。
### 源码
```c
void moloch_hex_init()
{
    int i, j;
    for (i = 0; i < 16; i++) {
        for (j = 0; j < 16; j++) {
            moloch_hex_to_char[(unsigned char)moloch_char_to_hex[i]][(unsigned char)moloch_char_to_hex[j]] = i << 4 | j;
            moloch_hex_to_char[toupper(moloch_char_to_hex[i])][(unsigned char)moloch_char_to_hex[j]] = i << 4 | j;
            moloch_hex_to_char[(unsigned char)moloch_char_to_hex[i]][toupper(moloch_char_to_hex[j])] = i << 4 | j;
            moloch_hex_to_char[toupper(moloch_char_to_hex[i])][toupper(moloch_char_to_hex[j])] = i << 4 | j;
        }
    }

    for (i = 0; i < 256; i++) {
        moloch_char_to_hexstr[i][0] = moloch_char_to_hex[(i >> 4) & 0xf];
        moloch_char_to_hexstr[i][1] = moloch_char_to_hex[i & 0xf];
    }
}
```

```c
char *moloch_char_to_hex = "0123456789abcdef"; /* don't change case */
unsigned char  moloch_char_to_hexstr[256][3];
unsigned char  moloch_hex_to_char[256][256];
```

## moloch_config_init
### 描述
[Arkime源码学习-config.c### moloch_config_init](Arkime源码学习-config.c.md)


## arkime_dedup_init
[Arkime源码学习-dedpu.c### arkime_dedup_init](Arkime源码学习-dedpu.c.md)

## moloch_writers_init
[Arkime源码学习-writers.c# moloch_writers_init](Arkime源码学习-writers.c.md)

## moloch_readers_init
[Arkime源码学习-readers.c#moloch_readers_init](Arkime源码学习-readers.c.md)

## moloch_plugins_init
[Arkime源码学习-plugins.c#moloch_plugins_init](Arkime源码学习-plugins.c.md)

## moloch_drop_privileges
```c
void moloch_drop_privileges()
{
    if (getuid() != 0)
        return;

    if (config.dropGroup) {
        struct group   *grp;
        grp = getgrnam(config.dropGroup);
        if (!grp) {
            CONFIGEXIT("Group '%s' not found", config.dropGroup);
        }

        if (setgid(grp->gr_gid) != 0) {
            CONFIGEXIT("Couldn't change group - %s", strerror(errno));
        }
    }

    if (config.dropUser) {
        struct passwd   *usr;
        usr = getpwnam(config.dropUser);
        if (!usr) {
            CONFIGEXIT("User '%s' not found", config.dropUser);
        }

        if (setuid(usr->pw_uid) != 0) {
            CONFIGEXIT("Couldn't change user - %s", strerror(errno));
        }
    }
}
```

## moloch_mlockall_init
```c
void moloch_mlockall_init()
{
#ifdef _POSIX_MEMLOCK
    struct rlimit l;
    getrlimit(RLIMIT_MEMLOCK, &l);
    if (l.rlim_max != RLIM_INFINITY && l.rlim_max < 4000000000LL) {
        LOG("WARNING: memlock in limits.conf must be unlimited or at least 4000000, currently %lu", (unsigned long)l.rlim_max/1024);
        return;
    }

    if (l.rlim_cur != l.rlim_max) {
        if (config.debug)
            LOG("Setting memlock soft to %lu", (unsigned long)l.rlim_max);
        l.rlim_cur = l.rlim_max;
        setrlimit(RLIMIT_MEMLOCK, &l);
    }

    int result = mlockall(MCL_FUTURE | MCL_CURRENT);
    if (result != 0) {
        LOG("WARNING: Failed to mlockall - %s", strerror(errno));
    } else if (config.debug) {
        LOG("mlockall with max of %lu", (unsigned long)l.rlim_max);
    }
#endif
}
```