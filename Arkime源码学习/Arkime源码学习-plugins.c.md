
## moloch_plugins_init

```c
void moloch_plugins_init()
{
    HASH_INIT(p_, plugins, moloch_string_hash, moloch_string_cmp);
}
```


## moloch_plugins_load
```c
void moloch_plugins_load(char **plugins) {

    if (!config.pluginsDir)
        return;

    if (!plugins)
        return;

    if (!g_module_supported ()) {
        LOG("ERROR - glib compiled without module support");
        return;
    }

    moloch_add_can_quit((MolochCanQuitFunc)moloch_plugins_outstanding, "plugin outstanding");

    int         i;

    for (i = 0; plugins[i]; i++) {
        const char *name = plugins[i];

        int d;
        GModule *plugin = 0;
        gchar   *path;
        for (d = 0; config.pluginsDir[d]; d++) {
            path = g_build_filename (config.pluginsDir[d], name, NULL);

            if (!g_file_test(path, G_FILE_TEST_EXISTS)) {
                g_free (path);
                continue;
            }

            plugin = g_module_open (path, 0); /*G_MODULE_BIND_LAZY | G_MODULE_BIND_LOCAL);*/

            if (!plugin) {
                LOG("ERROR - Couldn't load plugin %s from '%s'\n%s", name, path, g_module_error());
                g_free (path);
                continue;
            }
            break;
        }

        if (!plugin) {
            LOG("WARNING - plugin '%s' not found", name);
            continue;
        }

        MolochPluginInitFunc plugin_init;

        if (!g_module_symbol(plugin, "moloch_plugin_init", (gpointer *)(char*)&plugin_init) || plugin_init == NULL) {
            LOG("ERROR - Module %s doesn't have a moloch_plugin_init", name);
            continue;
        }

        plugin_init();

        if (config.debug)
            LOG("Loaded %s", path);

        g_free (path);
    }
}

```