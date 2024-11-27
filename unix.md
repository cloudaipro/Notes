###  [To Clear PageCache only](https://www.geeksforgeeks.org/how-to-clear-ram-memory-cache-buffer-and-swap-space-on-linux/)
```shell
sudo sh -c 'echo 1 >  /proc/sys/vm/drop_caches'
```
---

```shell
find . -name '*.ipynb' -print -exec grep 'DataReader' {} \;   
```
```shell
find . -name '*.ipynb' -exec grep -l "mapper" {} \;
```
