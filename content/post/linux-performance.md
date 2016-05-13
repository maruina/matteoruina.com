+++
date = "2016-05-12T09:20:18+01:00"
draft = false
title = "Linux performance and troubleshooting handbook"
tags = ["linux", "performance", "troubleshooting"]

+++

# Tools
## atop
View the contents of a file interactively
```bash
atop -r "${ATOP_FILE}"
```
Show the next sample from the file `t`
Show the previous sample from the file `T`

## Command line
Top 10 process by memory usage in Mb
```bash
ps -eo size,pid,user,command --sort -size | awk '{ hr=$1/1024 ; printf("%13.2f Mb ",hr) } { for ( x=4 ; x<=NF ; x++ ) { printf("%s ",$x) } print "" }' | head -n 10
```
