##Load Balance

              clint
                |
                |
            nginx+lua
            /        \
           /          \
        nginx+lua   nginx+lua
          |             |
          |             |
        nginx+dns   nginx+dns

##global backup

            backup server
                |
                |
            redis cluster
                |
                |
            backup client

##httpdns

    |----return--------->client<---return-------|
    |                       |                   |
    |                       |                   |
    |                       |                   |
    |                   Get(http://httpdns/dn?domain)
    |                       |                   |
    |                       |                   |
    |                       |yes                |
    from redis<-------redis cluster<---cache----|
                            |no                 |
                            |                   |
                            |                   |
                            |                   |resolve
                        DNS server -------------|   

##structure

                svn/git------watch_and_checkout------online
                  |
                  |
                  |
                producter


##init_sys and confiure

               confiure_web
                    |                  |---repo----svn/git
                    |                  |
              configure server---------|---template----svn/git
                    |
            -----------------
            |       |       |
            |       |       |
        client    client    client        

##cron job

/v1/worker/auid

    {
        "auid":"1001001",
        "tags":"mobile_pro",
        "ipaddr":"10.10.10.10",
        "hostname":"webserver"
    }
/v1/worker/auid/backup

    {
        "auid":"1001001",
        "redis":{
            "local":"ok",
            "push":"ok"
        },
        "mysql":{
            "local":"ok",
            "push":"ok"
        }
    }
/v1/worker/auid/service

    {
        "auid":"1001001",
        "servie":{
            "nginx":"ok",
            "php-fpm":"ok",
            "redis":"ok",
            "mysql":"ok"
        }
    }
/v1/worker/auid/sysinfo

    {
        "auid":"1001001",
        "disk"{
            "used":"50",
            "free":"50"
        },
        "mem":{
            "used":"50",
            "free":"50"
        },
        "cpu":{
            "sys":"5"
        }
    }
/v1/worker/auid/