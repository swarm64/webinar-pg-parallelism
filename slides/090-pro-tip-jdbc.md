-------------------------------------------------------------------------------
                    ____                _____ _
                   |  _ \ _ __ ___     |_   _(_)_ __
                   | |_) | '__/ _ \ _____| | | | '_ \ 
                   |  __/| | | (_) |_____| | | | |_) |
                   |_|   |_|  \___/      |_| |_| .__/
                                               |_|

* JDBC typically connects in "extended query mode", this blocks parallelism
* Use "simple query mode":

```bash
jdbc:postgresql://localhost/test?preferQueryMode=simple
```

-------------------------------------------------------------------------------
